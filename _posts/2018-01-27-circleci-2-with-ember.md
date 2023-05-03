---
layout: post
title: "Using CircleCI 2.0 with Ember"
description: "Using CircleCI 2.0 workflows with Ember.js"
category: Ember
tags: [ember, ci, circleci]
---

Ember projects and addons generated with ember-cli come with a `.travis.yml` configuration [out of the box](https://github.com/ember-cli/ember-new-output/blob/5b919fa1768cf1c421cf3367e256daefa666dfc0/.travis.yml), so it's clear why TravisCI has become the default for most Ember projects.
I, however, became quite fond of [CircleCI](https://circleci.com/) while using it at my last employer.
It has a great dashboard UI and a generous free tier, especially for open source projects.

After CircleCI 2.0 [became generally available](https://circleci.com/blog/launching-today-circleci-2-0-reaches-general-availability/) last summer, I transitioned our Ember app at $EMPLOYER over to it and found that the new version really had some interesting benefits for our project.
While I'm not at liberty to share the details of the $EMPLOYER project, I thought it would be interesting to revisit some of the things I learned then and make some general recommendations for using CircleCI with Ember applications and addons.

## Using CircleCI 2.0 with Ember Applications

I have a simple Ember application that I use for experimentation that is open sourced at [rebase-interactive/gitzoom-web](https://github.com/rebase-interactive/gitzoom-web). I'm going to use that project to iterate on a CircleCI 2.0 configuration.

First, I adapt [the default `.travis.yml`](https://github.com/ember-cli/ember-new-output/blob/5b919fa1768cf1c421cf3367e256daefa666dfc0/.travis.yml) that gets generated by ember-cli into an equivalent single-process configuration for CircleCI 2.0:

<script src="https://gist.github.com/kpfefferle/4aaf85e8e00839260c6a446e75f244c8.js?file=1-circleci.yml"></script>

- While CircleCI 1.0 used a `circle.yml` file at the project root, CircleCI 2.0 expects the configuration file to be named `config.yml` and be located inside a `.circleci` directory in the project root.
- By default, CircleCI 2.0 will run a job named `build`.
- CircleCI 2.0 runs all its processes inside of Docker containers. I'm using a [prebuilt image](https://circleci.com/docs/2.0/circleci-images/) for Node 6, and I've chosen the `-browsers` variant so that Chrome comes pre-installed in the container.
- While the the default `.travis.yml` sets the `JOBS` environment variable to `1`, I am setting it to `2` since I know that my CircleCI container can run two concurrent processes.
- I explicitly tell CircleCI to use `~/gitzoom-web` as the `working_directory` for this job. This makes it easy to refer to the location of my project in later steps.
- I define a series of `steps` for CircleCI 2.0 to run in sequence:
  - `checkout` is [a special step](https://circleci.com/docs/2.0/configuration-reference/#checkout) that checks out the source code (to the `working_directory` by default)
  - Most steps use `run` and have both a `name` (displayed in the CircleCI UI) and a `command` (run via shell).
  - Since I'm using [Yarn](https://yarnpkg.com) on this project, I use `yarn` instead of `npm` to install my dependencies and run my lint and test scripts.
  - In order to run `ember` commands (like the `ember test` command found in my `yarn test` script), I need to add `~/gitzoom-web/node_modules/.bin` to the container's `PATH`. I do this with a one-line unnamed `run` step.
  - `deploy` is [another special step](https://circleci.com/docs/2.0/configuration-reference/#deploy). It works just like a `run` step except that in jobs using parallelism, it will only run in one node and only if all nodes succeed (though that doesn't apply here). I use `|` to define a multi-line command that will only execute on the `master` branch.

While this gets my project building on CircleCI 2.0, it's not really taking advantage of any of the features that make CircleCI 2.0 so interesting.
I'll improve things by caching my installed dependencies next.

### Caching Dependencies

Caching in CircleCI 2.0 lets you reuse the data from expensive fetch operations from previous runs of a job to speed up future runs of that job.
To cache the contents of my `node_modules` directory between test runs, I add two new `steps` surrounding the installation of my NPM dependencies: `save_cache` and `restore_cache`.

<script src="https://gist.github.com/kpfefferle/4aaf85e8e00839260c6a446e75f244c8.js?file=2-circleci-caching.yml"></script>

- First, I define a `save_cache` step after my dependencies are installed.
  - CircleCI 2.0 caches are immutable, so once a cache has been written with a specific `key`, it cannot be overwritten. I use a `key` that increases in specificity left to right:
    - The `v1-` version prefix is a convenience that allows me to invalidate all preexisting immutable caches by bumping up to a new version prefix (such as `v2-`).
    - `deps-` defines this as my dependency cache, as opposed to some other cache I may add in the future such as a source cache.
    - `{% raw %}{{ .Branch }}-{% endraw %}` uses a [template](https://circleci.com/docs/2.0/caching/#using-keys-and-templates) to dynamically insert the name of the Git branch that is currently being built.
    - `{% raw %}{{ checksum "yarn.lock" }}{% endraw %}` generates a hash of my Yarn lockfile, ensuring that I get a new unique cache key any time the contents of my lockfile change.
  - I can include any number of paths in my cache, but in this case I only want to cache `./node_modules` (a path relative to my `working_directory`).
- Prior to installing my dependencies, I can now use `restore_cache` to look for an existing cache to pre-populate my `node_modules`. By defining multiple `keys` of decreasing specificity, I increase my chances of a cache hit as cache retrieval is prefix-matched:
  - First I look for a full match for the key defined in my `save_cache` step. If I'm on the same branch and my lockfile has not changed, I will retrieve a full restore of my `node_modules`.
  - Next I look for a match up to and including the current Git branch. Even if my lockfile has changed since the last build, I will still retrieve the pre-change `node_modules`.
  - Finally, I look for any existing dependency cache. Since it will return the latest prefix-matching cache, this still should be better than a full recreation of my `node_modules`.

It's possible to also cache the full source code of a project, which may be beneficial for larger projects. When project source has been recovered from cache, the `checkout` step will perform `git pull` instead of `git clone`.
For smaller projects like this one though, `git clone` is often faster than `restore_cache`.

Now the real fun begins: [Workflows](https://circleci.com/docs/2.0/workflows/)!

### Defining a CircleCI 2.0 Workflow

CircleCI 2.0's workflow feature lets me break up the build into smaller parts, each of which runs in its own isolated container. If one job in my workflow fails, I can rerun _just the failed job_ instead of having to run the entire build over in its entirety.
I can also define which jobs depend on which other jobs, which lets me parallelize parts of my build that are able to run independently from each other.

<script src="https://gist.github.com/kpfefferle/4aaf85e8e00839260c6a446e75f244c8.js?file=3-circleci-workflow.yml"></script>

There's a lot going on here, but I'll try to break it down a bit at a time.

- Instead of one `build` job, there are now five distinct jobs: `checkout_code`, `install_dependencies`, `lint_js`, `run_tests`, and `deploy_production`.
  - Since each job uses the same Docker image and working directory, I've defined these options at the top of the file using Yaml anchor syntax so that I don't have to repeat them. Instead I include them in each job with `<<: *defaults`.
  - The `checkout_code` job runs the special `checkout` step and then passes the checked out code to future jobs using `persist_to_workspace`. By defining the `root` as `.` (relative to `working_directory`) and the `paths` as `.` (relative to `root`), the entire contents of the working directory are passed along for future jobs to use.
      > Workspaces pass data along to other jobs in a workflow; caches pass data along to the same job in future workflow runs.
  - The `install_dependencies` job starts out by using `attach_workspace` to download the results of `checkout_code` into the working directory (`at: .`) of its container. It restores the dependency cache, installs dependencies, and caches the dependencies for future runs of the job. Finally, it uses `persist_to_workspace` to pass the installed dependencies forward to future steps in the workflow.
  - The `lint_js` job attaches the workspace and then runs the JavaScript linter command. It does not persist anything to the workspace as it does not produce anything used by upcoming jobs.
  - The `run_tests` job attaches the workspace and then runs the test script. It likewise does not persist anything to the workspace.
  - The `deploy_production` job runs the deployment command. Notice that the command no longer includes a branch conditional, as I define that filtering condition in the workflow itself.

- After defining the individual jobs, I chain them together by defining a workflow called `test_and_deploy` (this can be named anything that makes sense for the project). Here I list the `jobs` that will be performed as part of the workflow.
  - `checkout_code` runs immediately.
  - `install_dependencies` requires `checkout_code`, so it waits for that job to succeed before running.
  - `lint_js` and `run_tests` both require `install_dependencies`, so they wait for that job to succeed and then they *both* start running (in parallel containers)!
  - `deploy_production` requires both `lint_js` and `run_tests`, so it waits for both jobs to succeed before running. It additionally includes a [branch filter](https://circleci.com/docs/2.0/workflows/#branch-level-job-execution) so that it only runs on the `master` branch.

The result is that each job runs independently of the others and in the order defined by the workflow (with `deploy_production` not shown as this build was not run on `master`):

![CircleCI Workflow](/images/circleci-workflow.png)

#### Filtering Jobs by Tag

While branch-based job filtering is fairly straightforward (jobs run on all branches by default and filtering is only necessary to limit branches), [tag-based filtering](https://circleci.com/docs/2.0/workflows/#git-tag-job-execution) is a bit more complicated.
By default, workflow jobs do not run on tags _at all_. This means that if I want my deploy job to run only when I've tagged a release, then I must modify the `workflows` portion of my configuration:

<script src="https://gist.github.com/kpfefferle/4aaf85e8e00839260c6a446e75f244c8.js?file=4-circleci-tag-filters.yml"></script>

- I whitelist every pre-deploy job to run on all tags using regex `/.*/`. This runs the tests on tags, and allows me to verify that my tests all pass before deploying.
- I define two filters to control when the `deploy_production` job is run:
  - It runs only on tags that match my release tag format: `/v[0-9]+\.[0-9]+\.[0-9]+/`
  - It ignores all (`/.*/`) branches.

Now my deployment script will only run on tagged releases, and only after the previous jobs (including tests and linting) pass.
Since the `deploy_production` job requires both `run_tests` and `lint_js` to succeed, the workflow looks like this on tagged releases:

![CircleCI Workflow Deployment](/images/circleci-workflow-deploy.png)

### Additional Uses of CircleCI 2.0 Workflows

Once I begin breaking my CI builds down into smaller pieces, a number of other potential use cases become apparent:

- At $EMPLOYER, I used CircleCI workflows to deploy to multiple deploy targets simultaneously. We had two copies of our Ember application running in Staging: one against the Staging API server, and one against the Production API server. By running the deploy scripts for these environments concurrently, our CircleCI Staging build didn't take any longer to deploy than other branches with only one deploy target.
- Using the `--filter` argument for `ember test`, it would be possible to segment out different types of tests as separate workflow jobs and run acceptance tests separately from integration tests and unit tests.
- For addons using [ember-try](https://github.com/ember-cli/ember-try) to test against multiple versions of Ember, it's possible to break out each ember-try scenario into a separate job and run them in parallel. For my [ember-octicons](https://github.com/kpfefferle/ember-octicons) addon, see the CircleCI configuration [here](https://github.com/kpfefferle/ember-octicons/blob/38fae6b2ab0e20e9c666ae4e2a3e1af50b2c2a53/.circleci/config.yml) that results in the following workflow:

![CircleCI Addon Workflow](/images/circleci-workflow-addon.png)

If you try out CircleCI workflows with your Ember application or addon, I'd [love to hear](https://twitter.com/kpfefferle) what other ideas you come up with!

Like this post on CircleCI workflows and ready to get this sort of workflow working with your own codebase? 201 Created has worked on dozens of apps with Fortune 50 companies and Y-combinator startups. Visit [201-created.com](https://www.201-created.com/) or email [hello@201-created.com](mailto:hello@201-created.com) to talk with us.

---

- [CircleCI 2.0 Documentation](https://circleci.com/docs/2.0/)
- [CircleCI Blog: Persisting Data in Workflows: When to Use Caching, Artifacts, and Workspaces](https://circleci.com/blog/persisting-data-in-workflows-when-to-use-caching-artifacts-and-workspaces/)
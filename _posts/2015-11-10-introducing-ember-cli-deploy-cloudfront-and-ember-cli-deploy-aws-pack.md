---
layout: post
title: "Introducing ember-cli-deploy-cloudfront and ember-cli-deploy-aws-pack"
description: "Two new plugins for simple Ember deploys to CloudFront"
category: Ember
tags: [ember, aws, cloudfront, ember-cli-deploy]
---

Last week, I blogged about [deploying Ember to AWS CloudFront](/2015/11/01/deploying-ember-to-aws-cloudfront-using-ember-cli-deploy/). At the end of the post, I noted two improvements I'd like to make:

1. Automate the CloudFront cache invalidation step
2. Package up the ember-cli-deploy plugins needed into a plugin pack

Today, I'm happy to introduce two open source plugins to make the deploy process to AWS CloudFront even simpler.

## [ember-cli-deploy-cloudfront](https://github.com/kpfefferle/ember-cli-deploy-cloudfront)

The ember-cli-deploy-cloudfront plugin automates the CloudFront cache invalidation step. To use it:

1. Install the plugin

    <pre><code>$ ember install ember-cli-deploy-cloudfront</code></pre>

1. Add CloudFront configuration to `config/deploy.js`

    <pre><code>ENV.cloudfront = {
      accessKeyId: '<your-aws-access-key>',
      secretAccessKey: '<your-aws-secret>',
      distribution: '<your-cloudfront-distribution-id>'
    }</code></pre>

1. Run the ember-cli-deploy pipeline with the activation flag

    <pre><code>$ ember deploy production --activate</code></pre>

The plugin creates an invalidation with CloudFront for `/index.html` by default, though it can be configured to invalidate other CloudFront objects as well. See the [README](https://github.com/kpfefferle/ember-cli-deploy-cloudfront/blob/master/README.md) for more details.

## [ember-cli-deploy-aws-pack](https://github.com/kpfefferle/ember-cli-deploy-aws-pack)

Now that I've got a full set of ember-cli-deploy plugins suited to my deployment strategy, I've packaged them up into a [plugin pack](http://ember-cli.github.io/ember-cli-deploy/docs/v0.5.x/plugin-packs/) for easy installation as a group. To use the plugin pack in an Ember project (after you've configured S3 and CloudFront):

1. Install ember-cli-deploy and the plugin pack

    <pre><code>$ ember install ember-cli-deploy
    $ ember install ember-cli-deploy-aws-pack</code></pre>

1. Add deployment configuration secrets to `.env.deploy.production`

    <pre><code>AWS_KEY=[Access Key ID]
    AWS_SECRET=[Secret Access Key]
    PRODUCTION_BUCKET=[AWS Bucket]
    PRODUCTION_REGION=[AWS Region]
    PRODUCTION_DISTRIBUTION=[CloudFront Distribution ID]</code></pre>

1. Run the ember-cli-deploy pipeline

    <pre><code>$ ember deploy production</code></pre>

Notice that the `--activate` flag is no longer necessary since the plugin pack's `config/deploy.js` blueprint sets `ENV.pipeline.activateOnDeploy` to `true`. This can be adjusted as needed (along with any other deploy configuration) to suit the specific use case.

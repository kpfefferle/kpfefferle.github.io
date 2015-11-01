---
layout: post
title: "Deploying Ember with SSL on AWS"
description: ""
category: Ember
tags: [ember, emberjs, aws, s3, cloudfront, ssl]
---
{% include JB/setup %}

I've been working on a side project in [Ember](http://emberjs.com/), and I really want to serve the app with SSL. The app will be interacting with some 3rd party APIs and authenticating using 3rd party OAuth, so I want to serve it to users with encryption even though the HTML of an Ember app is served statically and the application code runs entirely in the browser.

To this point, I've been deploying the app on [Heroku](https://www.heroku.com/) with [heroku-buildpack-ember-cli](https://github.com/tonycoco/heroku-buildpack-ember-cli). While this approach is quick and easy to get up and running, Heroku charges $7/month to keep the dyno awake 24/7 and $20/month to add SSL. I'm not too excited to spend $27/month on a side project app.

I've been taking a look at AWS as an alternative. I already pay AWS each month to host remote backups on S3 and I've used S3 and CloudFront before as a CDN for app assets. What follows is the approach I ultimately ended up using.

## Step 1: Deploy ember-cli to S3

### Create buckets on S3

Most of the steps I'll follow on AWS are [well-documented](http://docs.aws.amazon.com/gettingstarted/latest/swh/website-hosting-intro.html).

1. Open the [AWS S3 Console](https://console.aws.amazon.com/s3/).
1. Click **Create a Bucket**.
1. In the **Create a Bucket** dialog box:
  1. Set the **Bucket Name**. I've set mine to match the app's subdomain at `app.[appdomain].com`.
  1. Select a **Region** close to your users (for me, this is US Standard).
  1. Click **Create**.


#### Bucket Permissions

I need to edit the bucket permissions to allow public read access to the files that I deploy.

1. Open the [AWS S3 Console](https://console.aws.amazon.com/s3/).
1. Select the `app.[appdomain].com` bucket, click **Properties**, click **Permissions**, and click **Add bucket policy**
1. Copy and paste the following policy into the Bucket Policy Editor (be sure to change `app.[appdomain].com` to match the name of your bucket):
```
{
  "Version":"2012-10-17",
  "Statement": [{
    "Sid": "Allow Public Access to All Objects",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::app.[appdomain].com/*"
  }]
}
```
1. Click **Save**.

### Create AWS Access Keys

I need to create a set of access keys that allow upload access to S3.

1. Open the [AWS IAM Console](https://console.aws.amazon.com/iam/).
1. Click **Users**, then click **Create New Users**.
1. In box **1**, type a name for the user, then click **Create**.
1. Click **Show User Credentials** and copy the Access Key ID and Secret Access Key into a file named `.env.deploy.production` in the root of the ember-cli app:
```
AWS_KEY=[Access Key ID]
AWS_SECRET=[Secret Access Key]
AWS_BUCKET=app.[appdomain].com
AWS_REGION=us-east-1
```
1. Click **close** twice to return to the list of users.
1. Click on the new user, click the **Permissions** tab, and click **Attach Policy**.
1. Select the policy named "AmazonS3FullAccess" and click **Attach Policy**.

### Install ember-cli-deploy

I've admired the [ember-cli-deploy](http://ember-cli.github.io/ember-cli-deploy/) project from a distance since [Luke Melia's great talk on deploying Ember apps at EmberConf](https://www.youtube.com/watch?v=4EDetv_Rw5U). This seems like a great chance to try it out.

```
$ ember install ember-cli-deploy
```

I'm going to deploy everything (including `index.html`) to S3, so I've also installed the following ember-cli-deploy plugins:

```
$ ember install ember-cli-deploy-build
$ ember install ember-cli-deploy-gzip
$ ember install ember-cli-deploy-manifest
$ ember install ember-cli-deploy-s3
```

Now I need to add the following configuration to `config/deploy.js` to load the config variables that I set in `.env.deploy.production` as well as tell ember-cli-deploy to upload all files (including `index.html`):

```
ENV.s3 {
  accessKeyId: process.env.AWS_KEY,
  secretAccessKey: process.env.AWS_SECRET,
  bucket: process.env.AWS_BUCKET,
  region: process.env.AWS_REGION,
  filePattern: "*"
}
```

Now I can upload my application build to S3 with one command:

```
$ ember deploy
```

### Enable Static Site Hosting on S3

To view the app, I need to enable static site hosting on my S3 bucket:

1. Open the [AWS S3 Console](https://console.aws.amazon.com/s3/).
1. Select the app.[appdomain].com bucket, click **Properties**, then click **Static Website Hosting**.
1. Click **Enable Website Hosting**.
1. Set **Index Document** to "index.html".
1. Click **Save**.

### Set S3 Routing Rules

If I click the **Endpoint** link to `app.[appdomain].com.s3-website-us-east-1.amazonaws.com`, I will now see my Ember app loaded in the browser! However, if I try to reload the page on any URL but the root, I get a 404 error. I need to configure routing rules so that S3 knows to route all requests to my `index.html`:

1. Under the **Static Website Hosting** settings, click **Edit Redirection Rules**.
1. Copy/paste the following rules into the textarea (replacing `app.[appdomain].com` with the actual bucket name):
```
<RoutingRules>
    <RoutingRule>
        <Condition>
            <HttpErrorCodeReturnedEquals>404</HttpErrorCodeReturnedEquals>
        </Condition>
        <Redirect>
            <HostName>app.[appdomain].com.s3-website-us-east-1.amazonaws.com</HostName>
            <ReplaceKeyPrefixWith>#!/</ReplaceKeyPrefixWith>
        </Redirect>
    </RoutingRule>
</RoutingRules>
```

Now if I click the **Endpoint** link to `app.[appdomain].com.s3-website-us-east-1.amazonaws.com`, I can reload any route URL and my Ember app will load the proper state without a 404!

## Distribute S3 Assets via CloudFront

## Add Custom Domain to CloudFront

## Add SSL to CloudFront

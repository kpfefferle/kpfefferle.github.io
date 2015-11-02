---
layout: post
title: "Deploying Ember to AWS with SSL"
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
1. Set the **Bucket Name**. I've set mine to match the app's subdomain at `app.[appdomain].com`.
1. Select a **Region** close to the app's users (for me, this is US Standard).
1. Click **Create**.


#### Bucket Permissions

I need to edit the bucket permissions to allow public read access to the files that I deploy.

1. Open the [AWS S3 Console](https://console.aws.amazon.com/s3/).
1. Select the `app.[appdomain].com` bucket, click **Properties**, click **Permissions**, and click **Add bucket policy**
1. Copy and paste the following policy into the Bucket Policy Editor (be sure to change `app.[appdomain].com` to match the name of the bucket):

<pre><code>{
  "Version":"2012-10-17",
  "Statement": [{
    "Sid": "Allow Public Access to All Objects",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::app.[appdomain].com/*"
  }]
}</code></pre>

1. Click **Save**.

### Create AWS Access Keys

I need to create a set of access keys that allow upload access to S3.

1. Open the [AWS IAM Console](https://console.aws.amazon.com/iam/).
1. Click **Users**, then click **Create New Users**.
1. In box **1**, type a name for the user, then click **Create**.
1. Click **Show User Credentials** and copy the Access Key ID and Secret Access Key into a file named `.env.deploy.production` in the root of the ember-cli app:

<pre><code>AWS_KEY=[Access Key ID]
AWS_SECRET=[Secret Access Key]
AWS_BUCKET=app.[appdomain].com
AWS_REGION=us-east-1</code></pre>

1. Click **close** twice to return to the list of users.
1. Click on the new user, click the **Permissions** tab, and click **Attach Policy**.
1. Select the policy named "AmazonS3FullAccess" and click **Attach Policy**.

### Install ember-cli-deploy

I've admired the [ember-cli-deploy](http://ember-cli.github.io/ember-cli-deploy/) project from a distance since [Luke Melia's great talk on deploying Ember apps at EmberConf](https://www.youtube.com/watch?v=4EDetv_Rw5U). This seems like a great chance to try it out.

    $ ember install ember-cli-deploy

I'm going to deploy everything (including `index.html`) to S3, so I've also installed the following ember-cli-deploy plugins:

    $ ember install ember-cli-deploy-build
    $ ember install ember-cli-deploy-gzip
    $ ember install ember-cli-deploy-manifest
    $ ember install ember-cli-deploy-s3

Now I need to add the following configuration to `config/deploy.js` to load the config variables that I set in `.env.deploy.production` as well as tell ember-cli-deploy to upload all files (including `index.html`):

    ENV.s3 {
      accessKeyId: process.env.AWS_KEY,
      secretAccessKey: process.env.AWS_SECRET,
      bucket: process.env.AWS_BUCKET,
      region: process.env.AWS_REGION,
      filePattern: "*"
    }

Now I can upload my application build to S3 with one command:

    $ ember deploy

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

<pre><code>&lt;RoutingRules&gt;
    &lt;RoutingRule&gt;
        &lt;Condition&gt;
            &lt;HttpErrorCodeReturnedEquals&gt;404&lt;/HttpErrorCodeReturnedEquals&gt;
        &lt;/Condition&gt;
        &lt;Redirect&gt;
            &lt;HostName&gt;app.[appdomain].com.s3-website-us-east-1.amazonaws.com&lt;/HostName&gt;
            &lt;ReplaceKeyPrefixWith&gt;#!/&lt;/ReplaceKeyPrefixWith&gt;
        &lt;/Redirect&gt;
    &lt;/RoutingRule&gt;
&lt;/RoutingRules&gt;</code></pre>

1. Click **Save**.

Now if I click the **Endpoint** link to `app.[appdomain].com.s3-website-us-east-1.amazonaws.com`, I can reload any route URL and my Ember app will load the proper state without a 404!

## Step 2: Distribute S3 Assets via CloudFront

At this point, I could change my DNS settings for `app.[appdomain].com` and point them at the S3 bucket since it's set up for static site hosting. But there are two reasons I want to press forward and distribute my Ember app through CloudFront. First, S3 does not support SSL for custom domains and CloudFront does. Second, CloudFront gives my application assets the speed boost that comes with being distributed to and delivered from CloudFront's edge locations around the world.

1. Open the [AWS CloudFront Console](https://console.aws.amazon.com/cloudfront/).
1. Under **Web**, click **Get Started**.
1. Fill **Origin Domain Name** with the *S3 Hosting Endpoint* (NOT the name of the bucket!!!). For this example, this is the `app.[appdomain].com.s3-website-us-east-1.amazonaws.com` address that I was testing at the end of the S3 setup.
1. Leave most of the settings in this form set to their defaults, but set **Default Root Object** to `index.html`. This will load my application when I visit the CloudFront root path.
1. Click **Create Distribution**.

It can take up to 15 minutes to create the CloudFront distribution. Once the **Status** column in the CloudFront list switches from "In Progress" to "Deployed", the distribution is complete. At this point, I can visit the CloudFront domain name at `[cloudfrontcode].cloudfront.net` and my Ember app should work here just as it did at the S3 Endpoint. If I refresh a route other than the root, it will redirect back to the S3 URL, but I will update the redirect rules once my custom domain is set up.

## Step 3: Use Custom Domain for CloudFront

Now that my Ember app is being served by CloudFront, I want to be able to access it using `app.[appdomain].com` instead of the unattractive `[cloudfrontcode].cloudfront.net`.

1. Open the [AWS CloudFront Console](https://console.aws.amazon.com/cloudfront/).
1. Select the CloudFront distribution I just created, then click **Distribution Settings**.
1. On the **General** tab, click **Edit**.
1. In **Alternate Domain Names (CNAMEs)**, enter the `app.[appdomain].com` custom domain that I want to use.
1. Click **Yes, Edit** to save this change.

It will take some time for this change to take effect. Once again, I can watch the **Status** column to change from "In Progress" to "Deployed". In the meantime, I'll set the DNS for `app.[appdomain].com` to point at CloudFront.

1. Open the domain's DNS settings (mine are managed on [DNSimple](https://dnsimple.com)).
1. Create a `CNAME` record that sets `app.[appdomain].com` as an alias for the CloudFront distribution at `[cloudfrontcode].cloudfront.net`.

Now when I visit `app.[appdomain].com`, I see my Ember app delivered via CloudFront! However, I still get redirected to the S3 Endpoint when I reload any non-root URL. I'll update the S3 Redirection Rules to change this:

1. Open the [AWS S3 Console](https://console.aws.amazon.com/s3/).
1. Select the app.[appdomain].com bucket, click **Properties**, then click **Static Website Hosting**.
1. Under the **Static Website Hosting** settings, open **Edit Redirection Rules** (if it's not already open).
1. Edit the  the `<HostName>` value to replace the full S3 Endpoint with just `app.[appdomain].com`:

<pre><code>&lt;RoutingRules&gt;
    &lt;RoutingRule&gt;
        &lt;Condition&gt;
            &lt;HttpErrorCodeReturnedEquals&gt;404&lt;/HttpErrorCodeReturnedEquals&gt;
        &lt;/Condition&gt;
        &lt;Redirect&gt;
            &lt;HostName&gt;app.[appdomain].com&lt;/HostName&gt;
            &lt;ReplaceKeyPrefixWith&gt;#!/&lt;/ReplaceKeyPrefixWith&gt;
        &lt;/Redirect&gt;
    &lt;/RoutingRule&gt;
&lt;/RoutingRules&gt;</code></pre>

1. Click **Save**.

Now non-root URLs will redirect properly to my custom domain. Note that CloudFront is **very** aggressive with caching, so since I tested the non-root redirects through the CloudFront before changing the redirect rules, I need to invalidate the current CloudFront cache *(these steps can be skipped if none of the non-root URLs were loaded through CloudFront before the redirect was updated)*:

1. Open the [AWS CloudFront Console](https://console.aws.amazon.com/cloudfront/).
1. Select the CloudFront distribution I just created, then click **Distribution Settings**.
1. Click the **Invalidations** tab, then click **Create Invalidation**.
1. Set **Object Paths** to `*` (all objects) and click **Invalidate**.

As with any CloudFront change, it will take a bit of time for the invalidation to occur across all of the CloudFront edge locations. I can monitor this again by watching the invalidation's **Status** column.

## Step 4: Add SSL to CloudFront

Now I can finally realize my ultimate goal: serving my Ember app over SSL. First, I must obtain an SSL certificate from any of a number of Certificate Authorities. I typically purchase my production-grade SSL certificates from [DNSimple](https://dnsimple.com) so I can manage them alongside my domain registration and DNS records, but for this side project I'll just use a free SSL certificate from [StartSSL](https://www.startssl.com).

From StartSSL, I get a few different files:

- `ssl.crt` contains my certificate
- `ssl.key` contains my private key
- `sub.class1.server.ca.pem` contains StartSSL's intermediate certificates
- `ca.pem` contains StartSSL's root certificate

Amazon requires a single file for StartSSL's certificates, so I run the following command to combine `sub.class1.server.ca.pem` and `ca.pem` into one file that contains both:

    $ cat sub.class1.server.ca.pem ca.pem > ca-bundle.pem

Uploading the certificate files requires the [AWS Command Line Interface](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html). If it's not already installed, it can be installed via Homebrew:

    $ brew install awscli
    $ aws configure

Now I can use the following command to upload my SSL certificate to AWS:

    $ aws iam upload-server-certificate --server-certificate-name [appdomain] --certificate-body file:///path/to/certificates/ssl.crt --private-key file:///path/to/certificates/ssl.key --certificate-chain file:///path/to/certificates/ca-bundle.pem --path /cloudfront/[appdomain]/

Be sure to change `[appdomain]` to something recognizable for the project.

Now I can tell CloudFront to use this SSL certificate for my custom domain:

1. Open the [AWS CloudFront Console](https://console.aws.amazon.com/cloudfront/).
1. Select the CloudFront distribution, then click **Distribution Settings**.
1. On the **General** tab, click **Edit**.
1. Under **SSL Certificate**, choose **Custom SSL Certificate (stored in AWS IAM)**.
1. In the dropdown, I select the certificate by the `[appdomain]` name I gave it in the aws-cli upload command.
1. Under **Custom SSL Client Support**, make sure that **Only Clients that Support Server Name Indication (SNI)** is selected. This option supports all modern browsers at no additional cost.
1. Click **Yes, Edit** to save this change.

Once again, it will take CloudFront a short while to update this change throughout its network.

## Deploying Future Versions

Whenever I've made changes to my Ember app that are ready to be deployed, I can deploy them with the same simple command:

    $ ember deploy

Fingerprinted assets like the app's CSS and JavaScript have unique names, so I don't need to worry about the CloudFront caching for these objects. I do, however, need to invalidate the CloudFront cache for my `index.html` (replace `[DISTRIBUTIONID]` with the CloudFront ID):

    // This first command only needs to be run once per aws-cli installation to enable the preview CloudFront commands
    $ aws configure set preview.cloudfront true

    $ aws cloudfront create-invalidation --distribution-id [DISTRIBUTIONID] --invalidation-batch "{\"CallerReference\": \"$(uuidgen)\", \"Paths\":{\"Quantity\":1,\"Items\":[\"/index.html\"]}}"

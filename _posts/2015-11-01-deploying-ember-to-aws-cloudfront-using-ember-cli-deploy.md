---
layout: post
title: "Deploying Ember to AWS CloudFront using ember-cli-deploy"
description: "SSL-friendly approach for deploying static Ember apps"
category: Ember
tags: [ember, aws, s3, cloudfront, ssl, ember-cli-deploy]
---

I am working on a side project in [Ember](http://emberjs.com/), and I want to serve the application with SSL. This application will be interacting with 3rd party APIs and authenticating using 3rd party OAuth, so I want to serve it to users with HTTPS encryption.

I've been deploying the application on [Heroku](https://www.heroku.com/) with [heroku-buildpack-ember-cli](https://github.com/tonycoco/heroku-buildpack-ember-cli). The Heroku approach is quick and easy to get up and running, but Heroku would charge $7/month to keep the application awake and $20/month to use SSL. I don't  want to spend $27/month on a side project if I don't have to.

I've been taking a look at Amazon Web Services (AWS) as an alternative. I already pay AWS each month to host remote backups on S3, and I've used S3 and CloudFront before as a CDN for Rails application assets. Here are the steps I am taking to deploy my static Ember application on AWS:

## Step 1: Deploy ember-cli to S3 with ember-cli-deploy

### Create a bucket on S3

Most of the steps I'm following to set up AWS are [well-documented](http://docs.aws.amazon.com/gettingstarted/latest/swh/website-hosting-intro.html):

1. Open the [AWS S3 Console](https://console.aws.amazon.com/s3/)
1. Click **Create a Bucket**
1. Set the **Bucket Name**; I've set mine to match the `app` subdomain I am using: `app.[appdomain].com`
1. Select a **Region** close to the app's expected users (for me, this is US Standard)
1. Click **Create**


### Edit Bucket Permissions

I need to edit the bucket permissions to allow public read access to the files that I upload:

1. Open the [AWS S3 Console](https://console.aws.amazon.com/s3/)
1. Select the `app.[appdomain].com` bucket, click **Properties**, click **Permissions**, and click **Add bucket policy**
1. Copy and paste the following policy into the Bucket Policy Editor (change `app.[appdomain].com` to match the name of the bucket):

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

1. Click **Save**

### Create AWS Access Keys

I need to create a set of access keys that allow upload access to S3:

1. Open the [AWS IAM Console](https://console.aws.amazon.com/iam/)
1. Click **Users**, then click **Create New Users**
1. In box **1**, type a name for the user (I used the name of my Ember app), then click **Create**
1. Click **Show User Credentials** and copy the Access Key ID and Secret Access Key into a file named `.env.deploy.production` in the root of the ember-cli app:

    <pre><code>AWS_KEY=[Access Key ID]
   AWS_SECRET=[Secret Access Key]
   AWS_BUCKET=app.[appdomain].com
   AWS_REGION=us-east-1</code></pre>

1. Click **close** (and again to confirm) to return to the list of users
1. Click on the new user, click the **Permissions** tab, and click **Attach Policy**
1. Select the policy named **AmazonS3FullAccess** and click **Attach Policy**

### Install the ember-cli-deploy addon

I've admired the [ember-cli-deploy](http://ember-cli.github.io/ember-cli-deploy/) project from a distance since [Luke Melia's great talk on deploying Ember apps at EmberConf](https://www.youtube.com/watch?v=4EDetv_Rw5U). This seems like a great chance to try it out.

    $ ember install ember-cli-deploy

I'm going to deploy everything (including my `index.html`) to S3, so I've installed the following [ember-cli-deploy plugins](http://ember-cli.github.io/ember-cli-deploy/docs/v0.5.x/plugins/):

    $ ember install ember-cli-deploy-build
    $ ember install ember-cli-deploy-gzip
    $ ember install ember-cli-deploy-manifest
    $ ember install ember-cli-deploy-s3

I need to add the following configuration to `config/deploy.js` to load the variables that I set in `.env.deploy.production` as well as tell ember-cli-deploy to upload all files (including `index.html`):

    ENV.s3 {
      accessKeyId: process.env.AWS_KEY,
      secretAccessKey: process.env.AWS_SECRET,
      bucket: process.env.AWS_BUCKET,
      region: process.env.AWS_REGION,
      filePattern: "*"
    }

Now I can upload my Ember application build to S3 with one command:

    $ ember deploy production

### Enable Static Site Hosting on S3

To use the application from S3, I need to enable static site hosting on my S3 bucket:

1. Open the [AWS S3 Console](https://console.aws.amazon.com/s3/)
1. Select the app.[appdomain].com bucket, click **Properties**, then click **Static Website Hosting**
1. Click **Enable Website Hosting**
1. Set **Index Document** to `index.html`
1. Click **Save**

## Step 2: Distribute S3 Assets via CloudFront

At this point, I could change my DNS settings to point `app.[appdomain].com` at the S3 bucket since it's set up for static site hosting (and the bucket name matches the subdomain). However, there are two reasons that I want to distribute my Ember application through CloudFront instead. First, S3 does not support SSL for custom domains. Second, CloudFront gives my application assets the speed boost that comes with being distributed to and delivered from CloudFront's edge locations around the world.

1. Open the [AWS CloudFront Console](https://console.aws.amazon.com/cloudfront/)
1. Under **Web**, click **Get Started**
1. Fill **Origin Domain Name** with the S3 Hosting Endpoint (NOT the name of the bucket); this is the `app.[appdomain].com.s3-website-us-east-1.amazonaws.com` address that I was testing in the browser at the end of my S3 setup
1. Leave most of the settings in this form set to their defaults, but set **Default Root Object** to `index.html`; this will load my application when I visit the CloudFront root path
1. Click **Create Distribution**

### Add CloudFront custom error response

1. Open the [AWS CloudFront Console](https://console.aws.amazon.com/cloudfront/)
1. Click on the **[Distribution ID]**
1. Click the **Error Pages** tab
1. Click **Create Custom Error Response**
1. Set **HTTP Error Code** to **404: Not Found**
1. Under **Customize Error Response**, click **Yes**
1. Set **Response Page Path** to `/index.html`
1. Set **HTTP Response Code** to **200: OK**
1. Click **Create**

It takes up to 15 minutes to create the CloudFront distribution. Once the **Status** column in the CloudFront list switches from "In Progress" to "Deployed", the distribution is complete. I visit the CloudFront domain name at `[cloudfrontcode].cloudfront.net`, and my Ember application works here just as it did at the S3 Endpoint.

## Step 3: Use Custom Domain for CloudFront

Now that my Ember application is being served by CloudFront, I want to access it using `app.[appdomain].com` instead of the unattractive `[cloudfrontcode].cloudfront.net`:

1. Open the [AWS CloudFront Console](https://console.aws.amazon.com/cloudfront/)
1. Select the CloudFront distribution I just created, then click **Distribution Settings**
1. On the **General** tab, click **Edit**
1. In **Alternate Domain Names (CNAMEs)**, enter the `app.[appdomain].com` custom domain that I want to use
1. Click **Yes, Edit** to save this change

It takes some time for this change to take effect. Once again, I watch for the **Status** column to change from "In Progress" to "Deployed". In the meantime, I set the DNS for `app.[appdomain].com` to point at CloudFront:

1. Open the domain's DNS settings (mine are managed on [DNSimple](https://dnsimple.com))
1. Create a `CNAME` record that sets `app.[appdomain].com` as an alias for the CloudFront distribution at `[cloudfrontcode].cloudfront.net`

I visit `app.[appdomain].com`, and I see my Ember application delivered via CloudFront.

## Step 4: Add SSL to CloudFront

I finally am reaching my ultimate goal: serving my Ember application over SSL. First, I obtain an SSL certificate through AWS Certificate Manager:

1. Open the [AWS Certificate Manager Console](https://console.aws.amazon.com/acm/)
1. Click **Get started** if no certificates exist or **Request a certificate** if there are existing certificates
1. Under **Domain name**, enter the application subdomain `app.[appdomain].com` that will be used
1. Click **Review and request**
1. Review the domain name, then click **Confirm and request**
1. Check the email address associated with the domain registration for a certificate approval email; in the email, click the link to **Amazon Certificate Approvals**
1. On the approval page, click **I Approve**

I tell CloudFront to use the new SSL certificate for my custom CloudFront domain:

1. Open the [AWS CloudFront Console](https://console.aws.amazon.com/cloudfront/)
1. Select the CloudFront distribution, then click **Distribution Settings**
1. On the **General** tab, click **Edit**
1. Under **SSL Certificate**, choose **Custom SSL Certificate (example.com)**
1. In the dropdown, I select the ACM certificate for `app.[appdomain].com`
1. Under **Custom SSL Client Support**, make sure that **Only Clients that Support Server Name Indication (SNI)** is selected; this option supports all modern browsers and does not add to the cost of using CloudFront
1. Click **Yes, Edit** to save this change

Once again, it takes CloudFront a short while to update this change throughout its network. Now that I am serving my Ember application over SSL, I want to be make HTTPS the default:

1. Open the [AWS CloudFront Console](https://console.aws.amazon.com/cloudfront/)
1. Select the CloudFront distribution, then click **Distribution Settings**
1. Click the **Behaviors** tab, select the Default behavior, then click the **Edit** button
1. Change the **Viewer Pretocol Policy** setting to **Redirect HTTP to HTTPS**
1. Click **Yes, Edit** to save this change

After CloudFront updates this change through its network, anyone accessing my Ember application over HTTP is automatically redirected to the HTTPS version.

## Step 5: Deploying Future Revisions

When I make changes to my Ember application that are ready to be deployed, I can easily deploy them with the ember-cli-deploy command:

    $ ember deploy production

Fingerprinted assets like the app's CSS and JavaScript have unique names, so I don't worry about CloudFront caching for these objects. I do, however, need to invalidate the CloudFront cache for my `index.html`. I can do this with the AWS CLI (replace `[distributionid]` with the proper CloudFront ID):

    // This first command only needs to be run once per aws-cli installation to enable the preview CloudFront commands
    $ aws configure set preview.cloudfront true

    $ aws cloudfront create-invalidation --distribution-id [distributionid] --invalidation-batch "{\"CallerReference\": \"$(uuidgen)\", \"Paths\":{\"Quantity\":1,\"Items\":[\"/index.html\"]}}"

*Note: This command assumes that your aws-cli user has CloudFront permissions.*

## Next Steps

- I really don't like having to run that ugly CLI command at the end to invalidate my `index.html`, so I'm working on an [ember-cli-deploy plugin](http://ember-cli.github.io/ember-cli-deploy/docs/v0.5.x/plugins/) to automate this step
- Once I've got the CloudFront invalidation step automated, I plan on creating an [ember-cli-deploy plugin pack](http://ember-cli.github.io/ember-cli-deploy/docs/v0.5.x/plugin-packs/) that contains all of the plugins needed for this deployment strategy

**UPDATE (11/10/2015):** These two enhancements are [now available](/2015/11/10/introducing-ember-cli-deploy-cloudfront-and-ember-cli-deploy-aws-pack/)

**UPDATE (2/9/2016):** Updated this post to use CloudFront custom error pages instead of S3 routing rules to handle loading non-root URLs [without a hash-based redirect](https://hashrocket.com/blog/posts/ember-on-s3-with-cloudfront-bash-the-hash)

**UPDATE (2/19/2016):** Updated this post to use [AWS Certificate Manager](https://aws.amazon.com/blogs/aws/new-aws-certificate-manager-deploy-ssltls-based-apps-on-aws/) to obtain an SSL certificate for CloudFront

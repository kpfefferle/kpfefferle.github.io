---
layout: post
title: "Deploying Ember to AWS Cheat Sheet"
description: "Implementation-only recipe for Ember deploy to AWS"
category: Ember
tags: [ember, aws, s3, cloudfront, ssl, ember-cli-deploy]
---

Today I set up my second Ember deployment to AWS. What follows is the consolidated recipe that comes from following the combination of my last [two](/2015/11/01/deploying-ember-to-aws-cloudfront-using-ember-cli-deploy/) [posts](/2015/11/10/introducing-ember-cli-deploy-cloudfront-and-ember-cli-deploy-aws-pack/):

## S3 Setup

### Create Bucket

1. Open the [AWS S3 Console](https://console.aws.amazon.com/s3/)
1. Click **Create a Bucket**
1. Set the **Bucket Name**; I've set mine to match the `app` subdomain I am using: `app.[appdomain].com`
1. Select a **Region** close to the app's expected users (for me, this is US Standard)
1. Click **Create**

### Edit Bucket Permissions

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

### Enable Static Site Hosting and Routing Rules

1. Within the bucket **Properties**, click **Static Website Hosting**
1. Click **Enable Website Hosting**
1. Set **Index Document** to `index.html`
1. Click **Save**

Make note of the `app.[appdomain].com.s3-website-us-east-1.amazonaws.com` endpoint displayed uther the **Static Website Hosting** settings. It will be needed later for CloudFront.

## Create SSL Certificate

1. Open the [AWS Certificate Manager Console](https://console.aws.amazon.com/acm/)
1. Click **Get started** if no certificates exist or **Request a certificate** if there are existing certificates
1. Under **Domain name**, enter the application subdomain `app.[appdomain].com` that will be used
1. Click **Review and request**
1. Review the domain name, then click **Confirm and request**
1. Check the email address associated with the domain registration for a certificate approval email; in the email, click the link to **Amazon Certificate Approvals**
1. On the approval page, click **I Approve**

## CloudFront Setup

### Create Distribution

1. Open the [AWS CloudFront Console](https://console.aws.amazon.com/cloudfront/)
1. Under **Web**, click **Get Started**
1. Fill **Origin Domain Name** with the S3 Hosting Endpoint (NOT the name of the bucket) `app.[appdomain].com.s3-website-us-east-1.amazonaws.com`
1. Set **Viewer Protocol Policy** to **Redirect HTTP to HTTPS**
1. Set **Forward Query Strings** to **Yes** (if the app uses query params)
1. In **Alternate Domain Names (CNAMEs)**, enter the `app.[appdomain].com` custom domain that I want to use
1. Under **SSL Certificate**, choose **Custom SSL Certificate (example.com)**
1. In the dropdown, select the ACM certificate for `app.[appdomain].com`
1. Set **Default Root Object** to `index.html`
1. Click **Create Distribution**

### Add CloudFront custom error response

This will handle requests for other routes within the Ember application [without using a hash-based redirect](https://hashrocket.com/blog/posts/ember-on-s3-with-cloudfront-bash-the-hash).

1. Open the [AWS CloudFront Console](https://console.aws.amazon.com/cloudfront/)
1. Click on the **[Distribution ID]**
1. Click the **Error Pages** tab
1. Click **Create Custom Error Response**
1. Set **HTTP Error Code** to **404: Not Found**
1. Under **Customize Error Response**, click **Yes**
1. Set **Response Page Path** to `/index.html`
1. Set **HTTP Response Code** to **200: OK**
1. Click **Create**

### Domain DNS

1. Open the domain's DNS settings (mine are managed on [DNSimple](https://dnsimple.com))
1. Create a `CNAME` record that sets `app.[appdomain].com` as an alias for the CloudFront distribution at `[cloudfrontcode].cloudfront.net`

## AWS Access Key Setup

1. Open the [AWS IAM Console](https://console.aws.amazon.com/iam/)
1. Click **Users**, then click **Create New Users**
1. In box **1**, type a name for the user (I used the name of my Ember app), then click **Create**
1. Click **Show User Credentials** and copy the Access Key ID and Secret Access Key into a file named `.env.deploy.production` in the root of the ember-cli app:

    <pre><code>AWS_KEY=[Access Key ID]
   AWS_SECRET=[Secret Access Key]
   PRODUCTION_BUCKET=[AWS Bucket]
   PRODUCTION_REGION=[AWS Region]
   PRODUCTION_DISTRIBUTION=[CloudFront Distribution ID]</code></pre>

1. Click **close** (and again to confirm) to return to the list of users
1. Click on the new user, click the **Permissions** tab, and click **Attach Policy**
1. Select the policies named **AmazonS3FullAccess** and **CloudFrontFullAccess**
1. Click **Attach Policy**

## ember-cli-deploy Setup

1. Install ember-cli-deploy and the plugin pack

    <pre><code>$ ember install ember-cli-deploy
   $ ember install ember-cli-deploy-aws-pack</code></pre>

1. Deploy Ember application

    <pre><code>$ ember deploy production</code></pre>

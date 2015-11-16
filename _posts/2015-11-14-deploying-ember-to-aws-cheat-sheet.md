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
1. Under the **Static Website Hosting** settings, click **Edit Redirection Rules**
1. Copy/paste the following rules into the textarea (replacing `app.[appdomain].com` with the proper domain name):

    <pre><code>&lt;RoutingRules&gt;
     &lt;RoutingRule&gt;
       &lt;Condition&gt;
         &lt;HttpErrorCodeReturnedEquals&gt;404&lt;/HttpErrorCodeReturnedEquals&gt;
       &lt;/Condition&gt;
       &lt;Redirect&gt;
         &lt;HostName&gt;app.[appdomain].com&lt;/HostName&gt;
         &lt;ReplaceKeyPrefixWith&gt;#/&lt;/ReplaceKeyPrefixWith&gt;
       &lt;/Redirect&gt;
     &lt;/RoutingRule&gt;
   &lt;/RoutingRules&gt;</code></pre>

1. Click **Save**

Make note of the `app.[appdomain].com.s3-website-us-east-1.amazonaws.com` endpoint displayed uther the **Static Website Hosting** settings. It will be needed later for CloudFront.

## SSL Certificate Setup

Obtain an SSL certificate for the domain. From StartSSL (for example), I get a few different files:

- `ssl.crt` contains my certificate
- `ssl.key` contains my private key
- `sub.class1.server.ca.pem` contains StartSSL's intermediate certificates
- `ca.pem` contains StartSSL's root certificate

Amazon requires a single file for the CA's certificates, so I run the following command to combine `sub.class1.server.ca.pem` and `ca.pem` into one file that contains both:

    $ cat sub.class1.server.ca.pem ca.pem > ca-bundle.pem

Uploading the certificate files requires the [AWS Command Line Interface](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html). If it's not already installed, it can be installed via Homebrew:

    $ brew install awscli
    $ aws configure

I use the following command to upload my SSL certificate to AWS:

    $ aws iam upload-server-certificate --server-certificate-name [appdomain] --certificate-body file:///path/to/certificates/ssl.crt --private-key file:///path/to/certificates/ssl.key --certificate-chain file:///path/to/certificates/ca-bundle.pem --path /cloudfront/[appdomain]/

This command requires the full `file://` path to each file. Be sure to change `[appdomain]` in both cases to something recognizable for the project.

## CloudFront Setup

### Create Distribution

1. Open the [AWS CloudFront Console](https://console.aws.amazon.com/cloudfront/)
1. Under **Web**, click **Get Started**
1. Fill **Origin Domain Name** with the S3 Hosting Endpoint (NOT the name of the bucket) `app.[appdomain].com.s3-website-us-east-1.amazonaws.com`
1. Set **Viewer Protocol Policy** to **Redirect HTTP to HTTPS**
1. Set **Forward Query Strings** to **Yes** (if the app uses query params)
1. In **Alternate Domain Names (CNAMEs)**, enter the `app.[appdomain].com` custom domain that I want to use
1. Under **SSL Certificate**, choose **Custom SSL Certificate (stored in AWS IAM)**; in the dropdown, select the certificate by the `[appdomain]` name I gave it in the aws-cli upload command
1. Set **Default Root Object** to `index.html`
1. Click **Create Distribution**

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

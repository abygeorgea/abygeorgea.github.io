---
layout: post
title: "Deploying Octopress to AWS S3 and CloudFront"
date: 2018-09-23 14:19:48 +1000
comments: true
categories: [Octopress, AWS]
keywords: Octopress, AWS
description: How to deploy octopress to AWS S3 and CloudFront
---

I was hosting my blog on github pages for past one year. Last week I decided to move hosting of my blog to AWS S3. There are obvious advantages of hosting a static site on S3. Moreover the cost of hosting is also minimal. There are many blogs in internet which explains the steps for hosting an octopress blog on S3. 

Since this is my first exposure to AWS world, I did had a learning curve to get this done. Below are highlevel steps involved in hosting in S3. 


*  Create an AWS login . Free plan was enough for my sites usage and traffic.
*  AWS recommends creating an IAM user for all activities instead of using root login. Hence I created an IAM user and gave permission to work on S3, Cloudfront, codecommit. Grab the AWS access KeyId and Secret Key from MyAccount > Security Credentials > Access Keys (for IAM user). 
*  Install and configure `s3cmd` for uploading to S3 bucket.  s3cmd is a free commandline tool to manage upload and retrieval of data from S3 bucket. Below are steps on Mac using homebrew for installing. 

```
# Install Homebrew 
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

# Install s3cmd
brew install s3cmd

# Verify installation
s3cmd --version

# Configure s3cmd
s3cmd --configure

# Configuration will require below information
* Access Key (use the access key from previous steps)
* Secret Key


``` 

*  Create S3 Bucket through s3cmd. Tutorial suggest creating S3 bucket with same domain name as static site. 


```
# Create new S3 Bucket
s3cmd mb s3://www.my-new-bucket-name.com
```
*  Login to AWS management console and verify the bucket exist. 
*  Navigate to properties and enable Static Website Hosting . Select index.html as index document.  This will give the direct link to blog once it is hosted. 
*  Also enable public access to read objects  in Permissions tab. 
*  Now, we need to modify the rake files to deploy to S3 Bucket. I was previously using Github pages and deploy was deploying to github. Hence made below changes to Rake File to deploy to S3 Bucket . It can be done by adding below details to Rake File. 


```
#Modify Rake File with below details - Change Default_deploy and add new variable

#deploy_default = "push" ( This is existing value. Hence commenting it)
deploy_default = "s3"
s3_bucket = "www.my-new-bucket-name.com"
# Replace the bucket name in above line with actual name

#Add below at end of rake file

desc "Deploy website via s3cmd"
task :s3 do
  puts "## Deploying website via s3cmd"
  ok_failed system("s3cmd sync --acl-public --reduced-redundancy --skip-existing --cf-invalidate public/* s3://#{s3_bucket}/")
end

#skip-existing allows to upload only changed files. This will help to reduce the contents pushed across to S3 and will help to reduce cost
# However I still need to test out how the deleted files are refelected
# Cf-Invalidate will help to push the latest changes to cloudfront


```
*  Run `rake deploy` to deploy the website to AWS S3 bucket.
*  Configure domain DNS to point to the AWS site( the link generated while enabling static website hosting) . We need to configure corresponding DNS entires in domain provider(in my case CrazyDomain).
*  At this point we can even use AWS Cloud Front. I followed the steps mentioned [here](https://stormpath.com/blog/ultimate-guide-deploying-static-site-aws) and [here](https://medium.com/@esfoobar/setting-up-an-https-static-site-using-amazon-aws-7ab270c4e277) for setting up a cloudfront and corresponding SSL certificates. Process is straight forward and easy to follow. Only place I struggled is while configuring dns entries in crazy doman as part of dns validation step during SSL certificate generation. I could not find the place to enter NAME for CNAME field. It is named as Subdomain in crazydomain.
*  Finally setup CNAME in domain provider to redirect to cloudfront url for our blog





While playing around with AWS, I noticed that AWS codecommit is always free for normal user ( eventhough usage limitation apply ) . I found that usage limitation is pretty high and I may not have to worry about that. Hence I decided to use AWS code commit for keeping blog's source repository . (Partly because Github doesn't support private repo on free plan). Process was straight forward as we do with any other source control system like Bitbucket, gitlab or github. Only catch I found was , we need to seperately create Https Credentials/SSH keys for AWS codecommit in IAM. The user name and password is different from normal login. Once everything was setup, I just changed the remote repository details on my local and pushed it through. 




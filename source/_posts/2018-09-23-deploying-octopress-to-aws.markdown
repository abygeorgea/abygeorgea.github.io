---
layout: post
title: "Deploying Octopress to AWS"
date: 2018-09-23 14:19:48 +1000
comments: true
categories: [Octopress, AWS]
keywords: Octopress, AWS
description: How to deploy octopress to AWS
---

Below are highlevel steps to deploy octopress blogs to AWS S3. Below are high level steps which I followed

*  Create an AWS login
*  Grab the AWS access KeyId and Secret Key from MyAccount > Security Credentials > Access Keys ( or use IAM users)
*  Install and configure `s3cmd` for uploading to S3 bucket.  

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
* Access Key
* Secret Key


``` 

*  Create S3 Bucket through s3cmd. 

```
# Create new S3 Bucket
s3cmd mb s3://www.my-new-bucket-name.com
```
*  Login to AWS management console and verify the bucket exist. Navigate to properties and enable Static Website Hosting . Select index.html as index document.  Also enable public access to read objects  in Permissions tab.  
*  Modify Rake File to deploy to S3 Bucket . It can be done by adding below details to Rake File. 


```
#Modify Rake File with below details - Change Default_deploy and add new variable

#deploy_default = "push" ( This is existing value)
deploy_default = "s3"
s3_bucket = "www.my-new-bucket-name.com"

#Add below at end of rake file

desc "Deploy website via s3cmd"
task :s3 do
  puts "## Deploying website via s3cmd"
  ok_failed system("s3cmd sync --acl-public --reduced-redundancy public/* s3://#{s3_bucket}/")
end



```
*  Run `rake deploy` to deploy the website to AWS S3 bucket.
# AWS-Static-Website

Hosting a Static Website on AWS using Terraform
This project uses Terraform to host a static website on AWS. The website is stored in an S3 bucket, served via a CloudFront distribution for CDN and SSL termination, and uses Route 53 for DNS management.

## Why i Use AWS?

Scalability: AWS services can handle traffic spikes efficiently.
Global Reach: CloudFront CDN ensures fast content delivery globally.
Cost-Effective: Pay-as-you-go pricing makes it affordable.
Security: AWS provides robust security features including SSL/TLS encryption.

## Architecture Overview

Amazon S3: Stores the static website files (HTML, CSS, JS).
Amazon CloudFront: Serves the website using a CDN, providing SSL termination and caching.
Amazon Route 53: Manages the domain name.
AWS Certificate Manager (ACM): Manages SSL certificates for HTTPS.

## Prerequisites
An AWS account.
Terraform CLI installed.
A text editor - Visual Studio Code
Your static website source code hosted on GitHub.

##Step-by-Step Guide

1. Clone Your GitHub Repository Locally

**bash
git clone <your-github-repo-url>
cd <your-github-repo-directory>

2. Set Up AWS

Create an S3 bucket for the website.
Create a CloudFront distribution to serve the website and handle SSL.
Set up Route 53 to manage your domain name.
Use AWS Certificate Manager (ACM) to manage SSL certificates.

3. Create Terraform Configuration

Create a new directory for your Terraform configuration and add the following files:

main.tf
hcl
Copy code
provider "aws" {
  region = "us-east-1" # Use the appropriate region
}

resource "aws_s3_bucket" "website_bucket" {
  bucket = "<your-bucket-name>"
  acl    = "public-read"

  website {
    index_document = "index.html"
    error_document = "error.html"
  }
}

resource "aws_s3_bucket_object" "website_files" {
  for_each = fileset("<path-to-your-website-files>", "**/*")

  bucket = aws_s3_bucket.website_bucket.id
  key    = each.value
  source = "<path-to-your-website-files>/${each.value}"
  acl    = "public-read"
}

resource "aws_cloudfront_distribution" "cdn" {
  origin {
    domain_name = aws_s3_bucket.website_bucket.bucket_regional_domain_name
    origin_id   = "S3-Website"

    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.origin_access_identity.cloudfront_access_identity_path
    }
  }

  enabled             = true
  is_ipv6_enabled     = true
  comment             = "CloudFront distribution for my website"
  default_root_object = "index.html"

  aliases = ["<your-domain-name>"]

  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-Website"

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }

    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 3600
    max_ttl                = 86400
  }

  price_class = "PriceClass_All"

  viewer_certificate {
    acm_certificate_arn = aws_acm_certificate.website_cert.arn
    ssl_support_method  = "sni-only"
  }
}

resource "aws_cloudfront_origin_access_identity" "origin_access_identity" {
  comment = "Origin Access Identity for CloudFront"
}

resource "aws_acm_certificate" "website_cert" {
  domain_name       = "<your-domain-name>"
  validation_method = "DNS"
}

resource "aws_route53_zone" "website_zone" {
  name = "<your-domain-name>"
}

resource "aws_route53_record" "www" {
  zone_id = aws_route53_zone.website_zone.zone_id
  name    = "<your-domain-name>"
  type    = "A"

  alias {
    name                   = aws_cloudfront_distribution.cdn.domain_name
    zone_id                = aws_cloudfront_distribution.cdn.hosted_zone_id
    evaluate_target_health = false
  }
}
variables.tf
hcl
Copy code
variable "aws_region" {
  description = "The AWS region to deploy to"
  default     = "us-east-1"
}

variable "domain_name" {
  description = "The domain name for the website"
}

variable "bucket_name" {
  description = "The name of the S3 bucket"
}
provider.tf
hcl
Copy code
provider "aws" {
  region = var.aws_region
}

4. Initialize and Apply Terraform Configuration

Initialize Terraform

bash
terraform init
Plan the Changes

bash

terraform plan -var="domain_name=<your-domain-name>" -var="bucket_name=<your-bucket-name>"
Apply the Configuration

bash

terraform apply -var="domain_name=<your-domain-name>" -var="bucket_name=<your-bucket-name>"

5. Upload Website Files to S3
If your website files are not automatically uploaded by Terraform (depending on your aws_s3_bucket_object resource), you can use the AWS CLI:

bash

aws s3 sync <path-to-your-website-files> s3://<your-bucket-name>

6. Validate Your Website

Check DNS Configuration: Ensure your domain name is correctly pointing to the CloudFront distribution using Route 53.
Visit Your Website: Go to <your-domain-name> and verify that your website is accessible over HTTPS.

How It Works
Route 53: The domain name is managed and routed to the CloudFront distribution.
CloudFront CDN: Ensures fast delivery of the website content globally and handles SSL termination.
S3 Bucket: Stores the static files (HTML, CSS, JS).
ACM: Manages SSL certificates for secure HTTPS connections.

Skills Gained
Using Terraform for AWS infrastructure.
Creating and hosting a static website on AWS.
Utilizing S3, CloudFront, and Route 53.
Setting up and managing SSL certificates with ACM.
Deploying a secure and scalable web architecture.

This detailed README provides a comprehensive guide on how to host your static website on AWS using Terraform, including the purpose and benefits of each step and service used.

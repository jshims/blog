---
title: "Conversion of Images to Web2 in AWS part 2"
datePublished: Fri Oct 20 2023 05:14:30 GMT+0000 (Coordinated Universal Time)
cuid: clny5pca500010amf8wmm8pmu
slug: conversion-of-images-to-web2-in-aws-part-2
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/DnXqvmS0eXM/upload/ba18d2ed7048198a5fdf1f6dcf3eb35f.jpeg
tags: aws, lambdaedge

---

When [converting images to WebP in our cloud](https://jshims.hashnode.dev/converting-images-to-webp-in-aws), one issue was that it did not effect images that were uploaded before adding the conversion logic. Thus, we needed a way to convert all the images in our S3 bucket.

### Problem

S3 contained images that were not in WebP format

### Issue

Our team decided to serve all images as WebP to reduce the rendering time on our website.

### Solution

Attach a Lambda@Edge to the origin response from our S3 bucket to check if the image was in WebP format. If not, send the data to our Simple Notification Service to which the Lambda in charge of converting the image subscribes.

### Implementation

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697778683420/6002ad9b-c577-4f4e-998f-1a8fa24abcc3.png align="center")

1. User requests image from CloudFront (CDN)
    
2. CloudFront requests image from S3
    
3. Lambda@Edge intercepts response and checks whether the image is WebP
    
    * If the image is not WebP creates a message to AWS SNS.
        
    * Lambda function in charge of converting image to WebP subscribes to the SNS and converts the image.
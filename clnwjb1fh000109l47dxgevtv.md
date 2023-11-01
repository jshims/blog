---
title: "Converting images to WebP in AWS"
seoTitle: "Converting images to WebP in AWS"
seoDescription: "Converting images to WebP in AWS"
datePublished: Thu Oct 19 2023 01:59:45 GMT+0000 (Coordinated Universal Time)
cuid: clnwjb1fh000109l47dxgevtv
slug: converting-images-to-webp-in-aws
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/DnXqvmS0eXM/upload/ba18d2ed7048198a5fdf1f6dcf3eb35f.jpeg
tags: aws, webp, aws-lambda, aws-sns

---

### Problem

Our website had a slow rendering time.

### Issue

The biggest issue that our team faced was that images were slowing down the total rendering time.

### Solution

Our team decided to reduce the size of images that were being displayed to the users. We decided to change the image type from jpg, png to WebP rather than resizing or reducing the quality of images that the users uploaded to our platform. [According to Google, by serving users WebP images, we could reduce the size by 25~34%](https://developers.google.com/speed/webp#:~:text=WebP%20is%20a%20modern%20image,in%20size%20compared%20to%20PNGs.).

### Solutions

We came up with the following solutions:

1. API server converts the images to WebP and uploads them to AWS S3 bucket
    
2. After the images are uploaded to S3, the creation of an object triggers SNS/Lambda which then converts the images and saves them to S3
    
3. When the user requests the images, a Lambda function intercepts the response, converts the image on the fly and returns it to the user
    

Our team decided on solution #2 since we viewed it as the most flexible one. By picking #2 we would decouple the logic of converting images to a serverless function, thereby making it more maintainable and upgradeable. If our company decided to add other processes such as resizing the images and saving them separately on S3, it would be much easier to plugin it to a serverless function. Moreover, by separating the logic to Lambda, our API server would be able to return a response more quickly to the user since it does not have to concern itself with additional logic.

As for #3, we did consider using Lambda@Edge to intercept the origin response from S3, convert the image to WebP, and cache it to CloudFront whilst returning the converted image to the user. However, as of this point, Lambda@Edge has a limitation of returning images up to 1MB. That limitation does not work with our use case. For those interested in a solution similar to #3 I recommend reading these two blogs from Amazon to see if it fits your use case.

* [Resizing Images with Amazon CloudFront & Lambda@Edge](https://aws.amazon.com/ko/blogs/networking-and-content-delivery/resizing-images-with-amazon-cloudfront-lambdaedge-aws-cdn-blog/)
    
* [Image Optimization using Amazon CloudFront and AWS Lambda](https://aws.amazon.com/blogs/networking-and-content-delivery/image-optimization-using-amazon-cloudfront-and-aws-lambda/)
    

### Implementation

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697677820712/5e2635d1-935b-43bb-a343-58cf9e87f6a5.png align="center")

The diagram above is a summary of how we decided to implement our conversion logic.

1. API server receives a request to upload an image
    
    * API server saves the image to S3
        
2. S3 triggers a create object event and sends a message to AWS SNS
    
3. AWS Lambda function subscribes to the SNS and receives said message
    
    * Converts the image to WebP
        
    * Overrides the original S3 image
        
    * Invalidates CloudFront cache to respond with converted WebP image
        

> Precautions
> 
> 1. SNS does not have a batch function meaning that if multiple images are uploaded it will spawn multiple lambda functions. A consideration may be adding [SQS to batch requests and using fewer lambda instances](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html).
>     
> 2. Overwriting the original S3 object will be invoked by creating a new object. This will trigger another message to SNS/Lambda. S3 warns you of a recursion problem when using the same S3 bucket for input and output. We dealt with the issue by checking the ContentType of the image being sent to the lambda function. We overwrote the original image to be backward compatible with other images whose endpoints are saved on our DB.
>     
> 
> ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697678121692/bc7ec364-873e-47cb-8456-fe31e9a76ec0.png align="center")

To implement the process above you would need the following:

* S3 bucket
    
* CloudFront
    
* SNS
    
* Lambda (Cloud9)
    
    * IAM role with cloudfront:CreateInvalidation policy
        
    * IAM role with s3:PutObject && GetObject policy
        

I will try to roughly outline what is needed since the implementation of each process may differ when you do it. I will assume that you have created a CloudFront distribution and linked it with a S3 as the origin server.

First, create a AWS Lambda to convert the image. When creating a Lambda it is important to give it the proper permissions. Thus, create an IAM role providing it permission to get objects from S3, put objects to S3, and invalidate the cache in CloudFront. It will look similar to this:

```json
{
    "Version": "version",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "cloudfront:CreateInvalidation",
            "Resource": "..."
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject"
            ],
            "Resource": "..."
        },
        {
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "..."
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": [
                "..."
            ]
        }

    ]
}
```

When writing the logic/code, I used Cloud9. However, you can choose whatever IDE is convenient for you. Write the code to convert the image, upload it to S3, and invalidate the CloudFront cache. I will share some code snippets in case you choose to use a similar environment to mine.

> architecture: x86 system
> 
> runtime: node.js 18.x
> 
> <div data-node-type="callout">
> <div data-node-type="callout-emoji">💡</div>
> <div data-node-type="callout-text">node.js 18.x by default uses ECMAscript. In this context, it means that you need to import your dependencies rather than require them. As a mainly Kotlin developer, I first did not understand this nuance and took some time to figure it out.</div>
> </div>

```javascript
/**
* import your dependencies and create a S3 and CloudFront object
* sharp is used to resize or convert your image
*/
import AWS from "aws-sdk";
import sharp from 'sharp';
const s3 = new AWS.S3({...});
const cloudFront = new AWS.CloudFront({...});

// when lambda is invoked, it calls handler
export const handler = async (event) => {
/**
* SNS sends a message. To see how it sends the object I would reference AWS SNS documentation.
* As of this post it was under Sns.Message
* get the S3 bucket and key (path) to image
*/
  const message = event.Records[0].Sns.Message;
  const msgObj = JSON.parse(message);
  const bucket = msgObj.Records[0].s3.bucket.name;
  const key = decodeURIComponent(msgObj.Records[0].s3.object.key.replace(/\+/g, ' '));
  const getParams = {
    Bucket: bucket,
    Key: key,
  };   

/**
* from here I surrounded the logic with a try/catch
*/
  const s3Object = await s3.getObject(getParams).promise();

  if (s3Object.ContentType.includes('webp') || s3Object.ContentLength == 0) {
    return { status: 208 };
  }
   
  let processedImage = 0;
  processedImage = await sharp(s3Object.Body)
            .rotate()
            .toFormat('webp', { quality: quality })
            .toBuffer();

// overwrite original image (same key)
  const uploadParams = {
    Bucket: bucket,
    Key: key,
    Body: Buffer.from(processedImage, 'binary'),
    ContentType: 'image/webp',
  };
  await s3.upload(uploadParams).promise();
        
  const invalidationParams = {
    DistributionId: "CloudFront distribution id",
    InvalidationBatch: {
      Paths: {
        Quantity: quantity,
        Items: [
          '/' + key
        ],
      },
      CallerReference: randomString,
    },
  };
  await cloudFront.createInvalidation(invalidationParams).promise();

  return { status: 200 }
// ...
}
```

Next, create a Simple Notification Server (SNS). You can do this by creating a Topic. After creating the topic, click on it to go to its dashboard and create a subscription. When creating a subscription choose the protocol as AWS Lambda and add your lambda endpoint.

Lastly, test your work. Either manually upload an image to S3 or use your working API. The creation of an object to S3 will trigger a message to SNS, your lambda will receive the said message and work its magic, and lastly, your images will be converted and CloudFront will distribute WebP images to users.

Happy programming :)
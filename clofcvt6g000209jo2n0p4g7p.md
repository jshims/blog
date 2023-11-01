---
title: "413 payload too large error when uploading image(s)"
datePublished: Wed Nov 01 2023 06:07:34 GMT+0000 (Coordinated Universal Time)
cuid: clofcvt6g000209jo2n0p4g7p
slug: 413-payload-too-large-error-when-uploading-images
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/DnXqvmS0eXM/upload/ba18d2ed7048198a5fdf1f6dcf3eb35f.jpeg
tags: nginx, clientmaxbodysize

---

### Problem

413 payload too large error occurred when users were trying to upload their images.

### Issue

Since our API server had a large enough limit to uploadable image sizes and had configured AWS Elastic Beanstalk nginx proxy server's configuration `client_max_body_size`, we did not know what the issue was.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">AWS Elastic Beanstalk uses nginx as a reverse proxy to map your application to your Elastic Load Balancing load balancer. For more info read <a target="_blank" rel="noopener noreferrer nofollow" href="https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-se-nginx.html" style="pointer-events: none">AWS documentation</a>.</div>
</div>

### Solution

Check the (actual) `client_max_body_size` set on EC2 instances.

I did not have a clear solution in mind, however, knew that it had to be a configuration issue. After searching for a command to check the actual configuration set on our API servers EC2 instance I came across [a post on AWS knowledge center](https://repost.aws/knowledge-center/elastic-beanstalk-nginx-configuration).

```bash
$ sudo nginx -T | egrep -i "client_max_body_size"
```

By connecting to our team's EC2 instances on the AWS console, I found out that it was a configuration issue on our frontend.

### Implementation

Our frontend was deployed using nginx which had it's configuration. By changing it's nginx configuration (nginx.conf) and setting the `client_max_body_size` to an appropriate size, we fixed the issue.
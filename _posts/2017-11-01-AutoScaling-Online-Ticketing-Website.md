---
layout: post
title:  "AutoScaling Online Ticketing Website"
date:   2017-11-01 16:16:01 -0600
categories: jekyll update
---
	| NodeJS | jQuery | Bootstrap CSS | JavaScript |

>Website : <http://honser.github.io>  

.  
.  
#### Problems
Certain times of year are the ultimate stress test for e-commerce, and so many retail sites fail to prepare at all. Websites that aren’t prepared pay the price. If you fail to handle the crush of traffic, the purchases are certainly going to be made from someone else, essentially doubling the damage of your downtime through the enrichment of your competition.
  
  
#### Objectives
- Services should not be stopped no matter how much traffic comes in.
- Payment must always be made accurate and clear.
- Real-time seats status.

#### Resources
- Auto Scaling & ELB(Elastic Load Balancer)
: Auto Scaling and ELB are suitable for maintaining high availability and distributing loads balancedly when we have sudden traffic spikes.

- S3(Simple Storage Service)
: S3 is good to store HTML and JavaScript Files.

- RDS(MySQL)
: RDS is more appropriate than DynamoDB for payment accuracy. DynamoDB is a database focused on availability and partition tolerance. Differently, RDS always ensures consistency.

- ElastiCache(Redis) & Socket.io
: Publish and Subscribe functions make it possible for scaled EC2 instances to share seats status in real-time. Socket.io is a JavaScript library. It enables real-time bidirectional communication between web clients and servers.

- CloudFront
: CloudFront is a global content deliver network(CDN) service. It can speed up the delivery of static contents. Real-time communication using Socket.io  makes 

- CloudWatch
: CloudWatch is required to use the AutoScaling service, since that service uses the monitoring data to figure out when to launch or terminate instances.

- IAM(Identity and Access Management)
: IAM enables EC2 instances to securely access to AWS services and resources.

- AMI(Amazon Machine Images)
: An Amazon Machine Image (AMI) provides the information required to launch an instance.



#### Architecture

![architecture]({{ site.baseurl }}/images/Diagram.jpg)
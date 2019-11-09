---
layout: post
title:  "AutoScaling Online Ticketing Website"
category: articles
tags: [NodeJS,CSS,AWS,Project]
---

>Website : <http://onlineticketsales.ml>  

---
 
## Problems
Certain times of year are the ultimate stress test for e-commerce, and so many retail sites fail to prepare at all. Websites that are not prepared pay the price. If you fail to handle the crush of traffic, the purchases are certainly going to be made from someone else, essentially doubling the damage of your downtime through the enrichment of your competition.
<br>
<br>

## Objectives
- Services should not be stopped no matter how much traffic comes in.
- Payment must always be made accurate and clear.
- Real-time seats status.
<br>
<br>

## Resources
- Auto Scaling & ELB(Elastic Load Balancer)
: Auto Scaling and ELB are suitable for maintaining high availability and distributing loads balancedly when we have sudden traffic spikes.

- S3(Simple Storage Service)
: S3 is good to store HTML and JavaScript Files.

- RDS(MySQL)
: RDS is more appropriate than DynamoDB for payment accuracy. DynamoDB is a database focused on availability and partition tolerance. Differently, RDS always ensures consistency.

- ElastiCache(Redis) & Socket.io
: Publish and Subscribe functions make it possible for scaled EC2 instances to share seats status in real-time. Socket.io is a JavaScript library. It enables real-time bidirectional communication between web clients and servers.

- CloudFront
: CloudFront is a global content deliver network(CDN) service. It can speed up the delivery of static contents. Real-time communication using Socket is possible. 

- CloudWatch
: CloudWatch is required to use the AutoScaling service, since that service uses the monitoring data to figure out when to launch or terminate instances.

- IAM(Identity and Access Management)
: IAM enables EC2 instances to securely access to AWS services and resources.

- AMI(Amazon Machine Images)
: An Amazon Machine Image (AMI) provides the information required to launch an instance.
<br>
<br>

## Architecture

![architecture]({{ site.baseurl }}/images/Diagram.jpg)
<br>
<br>

## Operation
1. Connect to www.onlineticketsales.ml and select a seat.  
2. EC2 instance obtains seat status from RDS.
3. After you click a seat, the payment window pops up. The seat will be reserved and inaccessible by others until the window is closed.
4. Click Proceed or Cancel button. 
5. The EC2 instance sends updated information to RDS. 
6. The instance send a message to ElastiCache for notifying every EC2 instance in AutoScaling group of the seat status(Publish).  
7. Every EC2 instance in AutoScaling group receives the message from ElastiCache, and then informs connected users of updated information.
<br>
<br>

## Simulation

![login]({{ site.baseurl }}/images/login.png)
![payment]({{ site.baseurl }}/images/payment.png)
![booked]({{ site.baseurl }}/images/booked.png)

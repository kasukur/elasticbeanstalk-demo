# Elastic Beanstalk Demo: Auto Scaling with a shared load balancer

## Table of Contents
1. [Introduction](#intro)
2. [Create a Security Group](#SG)
3. [Create an Application Load Balancer](#ALB)
4. [Create Elastic Beanstalk Admin App](#Admin)
5. [Create Elastic Beanstalk Forum App](#Forum)
6. [Auto Scaling Verification](#verify)
7. [Clean Up](#cleanup)
8. [Summary](#summary)
9. [Referrals](#referrals)

<a name="intro"></a>

### Introduction

One of the users of the AWS Certified Global Community had this query which caught my attention and made me want to check it out and see If I could help solve their problem. While doing this, I actually learned about shared load balancer with Elastic Beanstalk application and found it quite interesting, so I would like to share my learning with you.

With Elastic Beanstalk, you can quickly deploy and manage applications in the AWS Cloud without having to learn about the infrastructure that runs those applications. Elastic Beanstalk reduces management complexity without restricting choice or control. You simply upload your application, and Elastic Beanstalk automatically handles the details of capacity provisioning, load balancing, scaling, and application health monitoring. However, it becomes a bit difficult when we need to customise configurations such as Application Load Balancer, Listeners...etc

The user wanted to deploy two Elastic Beanstalk applications, let's say one for an **Admin** application and another one for **Forum** application. The problem the user was facing was particularly during auto-scaling - there seemed to be two instances created instead of one when she terminated one instance.

I'd like to share the demo below to show how to configure two different elastic Beanstalk applications using a shared load balancer and verify auto-scaling.

> Note: We can only have a maximum of **5** Target Groups per Action per Application Load Balancer.

## Demo
Let's get started with the demo.

<a name="SG"></a>

### Step 1. Create a Security Group

1. We are going to create a Security Group for Elastic Beanstalk

2. Navigate to EC2 > Security Groups > Create a new security group for your ALB, and set the following values:
   * Name: `EBSSG`
   * Add two Inbound rules to allow `HTTP` traffic from `0.0.0.0/0` (IPV4) and `::/0` (IPV6)

    _EBS Security Group_
    ![EBS-ALB-SG](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6ow00nvk1llzwfa3u5ng.png)

---

### Step 2. Create an Application Load Balancer

1. Navigate to EC2 > Load Balancers.
2. Click Create Load Balancer.
3. Click the Create button under the Application Load Balancer and set the following values:
    * Name: `SharedALB`
    * Scheme: `internet-facing`
    * IP address type: `ipv4`
    * Load Balancer Protocol: `HTTP`
    * Port: `80`
    * Leave the default `VPC`.
    * Select `us-east-1a` and `us-east-1b` AZs.
4. Click Next: Configure Security Settings. 
    * Note: Ignore the warning as we are not using HTTPS.
5. Select `EBSALB` and Click Next
6. Configure Routing, enter the following values and then Click Next:
    * Name: `EBS-TG`
    * Target type: `Instance`
    * Protocol: `HTTP`
    * Port: `80`
    * Path: `/index.html`
8. Click Next: Register Targets.
9. Click Next: Review.
10. Click Create
11. Wait until the Application Load Balancer `SharedALB` is Active.


    _Shared Application Load Balancer_
    ![Shared_ALB](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rruj98wd3fxifag1ni37.png)

    _Configure Security Options_
    ![EBS-Admin2](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ymp205xhhv8dku2refev.png)

    _Configure Security Groups_
    ![EBS-Admin3](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0b75en00vzk8cly5f2kp.png)

    _Configure Routing_
    ![EBS-Admin4](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5dept3yz5bifvhu6dv32.png)

    _Register Targets_
    ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y0og2wte1ux6q9g3o6iw.png)

    _Review_
    ![EBS-Admin6](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zunhyvwm94xct1v6912a.png)
    
    _Application Load Balancer is Active_
    ![ALB_Active](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rkbdrfldc0964t31c9rv.png)

    _Initial Listener Rules_
    ![Initial_Listener_Rules](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9fg5rpvlppp07cqztsm7.png)

---

<a name="Admin"></a>

### Step 3. Create Elastic Beanstalk Admin App
1. Navigate to Elastic Beanstalk > Environments.
2. Click `Create a new environment`.
3. Select `Web server environment`
4. Enter the following values:
    * Name: `Admin`
    * Platform: `Node.js`
    * Select `Upload your code` and upload [nodejs_admin.zip](https://github.com/kasukur/elasticbeanstalk-demo/blob/main/nodejs_admin.zip)
4. Click `Configure more options`
    * Note: Step 6, 7 and 8 should be done in the same order.
5. Select `High availability` under `Presets`.
6. Click `Edit` next to `Network` and select same two `Availability Zones` for `Load Balancer`, `Instance settings` and `Database settings`.
7. Click `Edit` next to `Load Balancer`
    * Load balancer type: `Application Load Balancer`
    * Type: `Shared`
    * Load balancer ARN: Select the ARN of `SharedALB`
    * Select `default` under Processes and click `Actions` > `Edit` > enter Path as `/admin.html` and click `Save`
    * Click `Add Rule` under `Rules`, Enter `Name` as `admin`, `PathPattern` as `/*`, `Process` as `default` and click `Add`
8. Click `Save`.
9. Click `Create Environment`.
10. Wait for the environment to be create and then click on the application URL.

> Note: If you need to make any changes to the code and create a zip on a Mac OS

```bash
cd /nodejs_admin
zip ../nodejs_admin -r * .[^.]*
```

_EBS Admin 1_
![EBS_Admin1](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6732crlvx8mlh13k5a31.png)

_EBS Admin 2.1_

![EBS_Admin2.1](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jsmjlbadqkl839fo8mrd.png)

_EBS Admin 2.2_

![EBS_Admin2.2](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/l1lwmhzcfsv88utykmh4.png)

_EBS Admin 3.1_
![EBS_Admin3.1](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o9d67h2hvq6x7czo3wwa.png)

_EBS_Admin 3.2_
![EBS_Admin3.2](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f0ogdh9zck0hzti8o61l.png)

_EBS_Admin 3.3_

![EBS_Admin3.3](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/l0lrjkntsat52ccmo9kj.png)

_EBS Admin 4.1_
![EBS_Admin4.1](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tlspioc97hzlb5pgef75.png)

_EBS Admin 4.2_

![EBS_Admin4.2](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ux9s752o75et8g7g3qu3.png)

_you might notice this error if you do not select the shared load balancer_
![Caution_Error](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nfowpm7zjbu1h2afrq7l.png)

---

<a name="Forum"></a>

### Step 4. Create Elastic Beanstalk Forum App

1. Navigate to Elastic Beanstalk > Environments.
2. Click `Create a new environment`.
3. Select `Web server environment`
4. Enter the following values:
    * Name: `Forum`
    * Platform: `Node.js`
    * Select `Upload your code` and upload [nodejs_forum.zip](https://github.com/kasukur/elasticbeanstalk-demo/blob/main/nodejs_forum.zip)
4. Click `Configure more options`
    * Note: Step 6, 7 and 8 should be done in the same order.
5. Select `High availability` under `Presets`.
6. Click `Edit` next to `Network` and select same two `Availability Zones` for `Load Balancer`, `Instance settings` and `Database settings`.
7. Click `Edit` next to `Load Balancer`
    * Load balancer type: `Application Load Balancer`
    * Type: `Shared`
    * Load balancer ARN: Select the ARN of `SharedALB`
    * Select `default` under Processes and click `Actions` > `Edit` > enter Path as `/forum.html` and click `Save`
    * Click `Add Rule` under `Rules`, Enter `Name` as `admin`, `PathPattern` as `/*`, `Process` as `default` and click `Add`
8. Click `Save`.
9. Click `Create Environment`.
10. Wait for the environment to be create and then click on the application URL.

> Note: If you need to make any changes to the code and create a zip on a Mac OS

```bash
cd /nodejs_forum
zip ../nodejs_forum -r * .[^.]*
```

_EBS Forum 4.3_

![EBS_Forum4.3](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hcdqha83enxj22ty7ov1.png)

---

<a name="Verify"></a>

### Step 5. Auto Scaling Verification
1. Navigate to `EC2` > `Instances`
2. Select `Admin-env instance`
3. Click `Actions` > `Instance State` and `Terminate`
4. Wait for a few minutes, the auto scaling group will launch a new instance for Admin application, which can be checked under `Auto Scaling Groups` > `Activity`
   Note: There should only be one instance launched but not two.
5. Similarly `Terminate` the `Forum-env` and there should new instance created in a few minutes.

_EBS Environments_
![EBS-Envs](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cagtajdm2vj85gftnarp.png)

_EBS Admin Application_
![EBS-Admin](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4trpqxzq3rd3t8s15lzv.png)

_EBS Admin Application_
![EBS-Forum](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2klp6jc2ddse395t0nza.png)

_Application Load Balancer Listener Rules_
![ALB-Listener-Rules](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/51mpd425gomme4bg6pes.png)

_EC2 Instances_
![EC2_Instances](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ga2384u4i4pq3otoau3b.png)

_EBS Admin Auto Scaling Activity_
![EBS-Admin-AS-1](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/46dt2dhh2rmr4plkgi5g.png)

_EBS Forum Auto Scaling Activity_
![EBS-Admin-AS-2](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y135lt2z9amox1513eze.png)

_EC2 Instances: Only one new Admin instance_
![EBS-Admin-AS-3](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i7mtg181ord9j2v869sf.png)

_EC2 Instances: Forum Instance terminated manually and new instance created by Auto Scaling_
![EBS-Admin-AS-4](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/63qeuqr53n9dhcjoqmwx.png)

_EBS Forum Auto Scaling Activity_
![EBS-Admin-AS-5](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y5yegdqkobwgv4umcfyn.png)

---

<a name="cleanup"></a>
### Clean Up

1. Select each environment under `Elastic Beanstalk` > `Environments`, Select `Actions` > `Terminate Environment`.
2. Select each application under `Elastic Beanstalk` > `Applications`, Select `Actions` > `Delete Application`.
3. Delete `SharedALB` under `Load Balancers`
4. Delete Security Group `EBSSG`.
5. After both environments are terminated, Empty and Delete S3 bucket prefixed with `elasticbeanstalk-us-east-1-...`

---

<a name="summary"></a>
### Summary
- We learned how to deploy and configure different settings using Elastic Beanstalk.
- We also learned about shared application load balancer, which can save costs.

---

<a name="referrals"></a>
### Referrals
- [Configuring a shared Application Load Balancer using the Elastic Beanstalk console] (https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-cfg-alb-shared.html)
- The application code is from [Tutorials and samples](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/samples/nodejs.zip) and I have added admin.html and forum.html.
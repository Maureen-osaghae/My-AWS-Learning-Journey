<h1>How to Deploy Amazon CloudFront for Optimized Content Delivery, Caching  and Application  Security <h/2>
<h3>Lab overview and objectives</h3>
In this lab, I'll set up an Amazon CloudFront distribution to minimize latency for café website visitors and ensure secure HTTPS delivery. 
I will also protect access to the website and its REST API endpoints by implementing AWS Web Application Firewall, which offers web application firewall capabilities. Lastly, I'll deploy a CloudFront function for the site and tweak the expiration times for cached content. 
<h3>What I did:</h3>
After completing this lab, I mastered skills that offer several benefits:
  
• Create a CloudFront distribution to cache Amazon Simple Storage Service (Amazon S3) objects: This reduces load times and bandwidth costs by serving content from the edge, closer to users.

• Configure a website hosted on Amazon S3 to be available through HTTPS using CloudFront:  This enhances security by ensuring all data transfers are encrypted, protecting user data from interception.

• Secure access to the CloudFront distribution based on the network origin of the request: This prevents unauthorized access, adding an additional layer of security by geo-restricting content.

• Secure a REST API endpoint based on the network origin of the request using AWS WAF: This shields the API from malicious traffic, improving reliability and safety of the application.

• Configure a CloudFront function to affect website behavior from the edge location: This enables dynamic content manipulation without server-side processing, speeding up response times and reducing server load.

• Adjust max-age caching settings on a CloudFront distribution:  This optimizes content delivery by controlling how long content is cached, reducing server requests for unchanged content, thus enhancing performance and reducing costs.

<h2>Business Scenario</h2>
Sofía is pleased with how her café website development project is coming along. She has developed the core serverless application that displays menu items on the website. She also integrated the coffee suppliers web application into the main site and is using Amazon ElastiCache features for the suppliers part of the site.
However, she knows that some essential features are still missing. One feature is that the website still runs on HTTP and does not yet support HTTPS. 
Sofía also wants to ensure that the website will load quickly for users globally. She knows that AWS has many Regions and Availability Zones, but they also have edge locations, which are even closer to users around the globe. She decides to host the café website on a proper content delivery network (CDN), and she has opted to use the CloudFront service.

In this lab, I will play the role of Sofía to continue to develop the café's web application.
When I start the lab, my architecture will look like the following diagram, with several preconfigured resources.

<img width="402" alt="image" src="https://github.com/user-attachments/assets/1f3dc75f-027c-45b5-9d69-f654b28f7812" />

By the end of this lab, I will have created the architecture in the following diagram. The highlighted portion shows the part of the architecture that I will work on in the lab

<img width="422" alt="image" src="https://github.com/user-attachments/assets/e276cf9d-bcce-485a-ac5c-5f49185391ec" />
<h2>Task 1: Preparing the development environment</h2>
In this first task, I will configure my AWS Cloud9 environment and also run the script to recreate the work I completed in previous labs. 
Before proceeding to the next step, I have to verify that the AWS CloudFormation stack creation process for this lab has successfully completed.

In a new browser tab, I navigate to the CloudFormation console.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/3e562d52-2f49-497c-917c-7ad020972850" />

• In the left navigation pane,  choose Stacks.

• For the stack with "ACD_2.0" in the Description column,  verify that the Status says CREATE_COMPLETE. 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/4a5c5a0d-55c6-4db4-9ad5-835f8979f744" />

If it doesn't show CREATE_COMPLETE yet, I waited until it does. Because this stack is creating an Amazon Relational Database Service (Amazon RDS) database instance, it might take about 5 minutes to complete.

Next, is to connect to the AWS Cloud9 integrated development environment (IDE) named Cloud9 Instance.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/94d437db-f802-4cc6-9e6c-040f8aebac34" />

Downloaded and extract the files that I need for this lab. 

To Download and extract the files that I need for this lab.  I will run the following command: 

    wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCDEV-2-91558/09-lab-cloudfront/code.zip -P /home/ec2-user/environment
<img width="959" alt="image" src="https://github.com/user-attachments/assets/05e97a22-b829-4c91-9953-5b6d428164e5" />

The code.zip file is downloaded to the AWS Cloud9 instance. The file is listed in the left navigation pane.
To Extract the file: 

    unzip code.zip

<img width="571" alt="image" src="https://github.com/user-attachments/assets/bef95988-0824-438f-a14a-fb44bd91796c" />

Run a script to upgrade the version of Python and the AWS CLI that are installed on the Cloud9 instance. The script also recreates the work that I completed in earlier labs into this AWS account.
        
          chmod +x ./resources/setup.sh && ./resources/setup.sh
• Set permissions on the script so that I can run it, and then run the script:

<img width="956" alt="image" src="https://github.com/user-attachments/assets/66308691-e72b-446b-a648-96307b09178e" />

When prompted for an IP address, enter the IPv4 address that the internet uses to contact my computer.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/cbcccfa4-928b-4171-ba8c-051341ff6581" />

Analysis: The CloudFormation template that ran when I started this lab created resources in the AWS account. The script I ran also created resources. Between the two of them, the following resources have been created to replicate what I built in previous labs:

• An S3 bucket with an associated bucket policy. The bucket contains the café website code.

• An Amazon DynamoDB table populated with menu data.

• A REST API configured using Amazon API Gateway.

• An AWS Lambda function that retrieves data from DynamoDB when invoked.

• A Memcached cluster that caches supplier data from Amazon Aurora Serverless for the suppliers application. A café/node-web-app Docker image, which is stored in the Amazon Elastic Container Registry (Amazon ECR).

• An AWS Elastic Beanstalk environment and application that runs an EC2 instance named MyEnv. The EC2 instance hosts a Docker container created from the Docker image that is stored in Amazon ECR.

• An Aurora Serverless database running MySQL on Amazon RDS, which contains the supplierdb database, which stores coffee supplier information.

Let me verify the version of AWS CLI installed. In the AWS Cloud9 Bash terminal (at the bottom of the IDE), run the following command:

    aws --version
<img width="828" alt="image" src="https://github.com/user-attachments/assets/306c1503-eb07-49a2-8916-1d055f254785" />

The output indicate that version 2 is installed 

To verify that the SDK for Python is installed.  Run the following command:

    pip show boto3
<img width="817" alt="image" src="https://github.com/user-attachments/assets/e3e6ee94-b987-40bf-956c-de1cba76a087" />

I note the metadata settings on the objects stored in the S3 bucket, and then verify that I can access the café website. Navigate to the Amazon S3 console. Choose the link for the bucket

<img width="959" alt="image" src="https://github.com/user-attachments/assets/17c21ed2-067b-4036-be07-0dc217dee252" />

Choose the index.html link.
<img width="959" alt="image" src="https://github.com/user-attachments/assets/7b7d1c60-35d2-46a8-9cf6-c4362a0e04ea" />

Scroll down to the Metadata section.
Two key-value pairs are listed. The Cache-Control key-value pair has a value of max-age=0. This was set when I ran setup.sh in an earlier step. Line 21 of that script ran the command
       
       aws s3 cp ./resources/website s3://$bucket/ --recursive --cache-control "max-age=0" 
to set this metadata value on every file that it uploaded to the bucket.

<img width="953" alt="image" src="https://github.com/user-attachments/assets/3803cc49-c4ac-4563-998b-be54425d1459" />















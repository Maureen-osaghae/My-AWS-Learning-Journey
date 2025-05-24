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

At the top of the page, open the Object URL in a new browser tab. The café website displays. 

<img width="823" alt="image" src="https://github.com/user-attachments/assets/f1c8c7c2-9ea9-493a-ae7e-fd409c62b6ce" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/ce369b43-0d76-47ec-8aef-d310c3a95b42" />

 I will return to it later in this lab.

 <h2>Task 2: Configuring a distribution for static website content</h2>

 In this task, I will configure the café website, which is hosted on Amazon S3, to be available through a CloudFront distribution. With the distribution, I can enable HTTPS access to the website.

 <h3>Detailed analysis: </h3>
Sofía Recall from earlier labs that static website hosting, which would make the site available at https://.s3-website.amazon.com, is not configured on the café website S3 bucket. Instead, she access the website by loading the Object URL of the index.html file from the bucket, which is in the form http://.s3.amazon.com/index.html. Therefore, the site is not currently accessed using HTTPS. Sofía hasn't worried about this until now, because she knew she would integrate other AWS services into the website design and would use CloudFront as part of the final application design. 

I will Begin to configure a CloudFront distribution for the café website hosted on Amazon S3.
        
◦  Navigate to the CloudFront console.

◦  Choose Create a CloudFront distribution.

This step is very crucial,  I will configure several settings for the distribution over the next few steps. I pay careful attention not to miss any steps. 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/68f81948-c1b7-45c9-bbdc-816ef674458a" />

First is to Configure the Origin settings for the distribution.

◦  Origin domain: Search for and choose the S3 bucket that has -s3bucket in the name. This bucket contains the website code.

<img width="955" alt="image" src="https://github.com/user-attachments/assets/b066621f-16d9-4a56-a52f-c6616eb0c82e" />

• In the Origin access settings. 

◦ Choose Legacy access identities.

◦ Choose Create new OAI.

◦ Name: Enter access-identity-cafe-website

◦ Choose Create.
<img width="535" alt="image" src="https://github.com/user-attachments/assets/fb7724b6-10c2-4a63-9a88-3f383e652590" />

◦ Bucket policy: Choose Yes, update the bucket policy.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/9381d402-ac49-4383-b1da-bbec1b480fe1" />

In the Add custom header section, choose Add header and configure:

◦ Header name: Enter cf

◦ Value: Enter 1

 Note: The custom header is not an important requirement for this lab, but it will be useful in a later lab.

 <img width="959" alt="image" src="https://github.com/user-attachments/assets/cdd719fa-7a3d-417d-a0f2-c89a9725e2ce" />

Configure the Default cache behavior settings.

◦ Path pattern: Keep the Default (*) setting. This means that all requests will go to the origin.
        
◦ Viewer protocol policy: Choose Redirect HTTP to HTTPS. This will help users find the website, even if they load the HTTP URL.

◦ Allowed HTTP methods: Keep the default GET, HEAD setting.
        
◦ Restrict viewer access: Keep the default No setting. This will be a public website.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/4f80bf3a-a65f-4982-9eec-a76e9d99e785" />

Configure the Cache key and origin requests settings.

◦ Choose Legacy cache settings.

◦ Keep the settings for Headers, Query strings, and Cookies as None.

◦ Object caching: Choose Use origin cache headers.
         
Note: Recall the metadata key-value pair with key Cache-Control and value max-age=0. This metadata is set on all of the objects stored in the origin S3 bucket. By choosing Use origin cache headers for this distribution, I ensure that this distribution will inherit this setting applied to all of the objects in the Amazon S3 origin.

◦ Under Web Application Firewall (WAF) choose Do not enable security protections. 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/e3636272-db5e-4637-b0db-11ddd1742475" />

Configure the Settings section of the distribution.
    
• Price class: Keep the default Use all edge locations setting.
    
• Alternative domain name (CNAME): Do not enter anything.

Note: I could specify a unique domain name here; however, this lab doesn't have one.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/0c680d95-3bab-4c2a-ba8c-7015034ab7d3" />

Custom SSL certificate: Do not enter anything.
      
Note: If I owned a domain and had a TLS or SSL certificate from a certificate authority, I would upload it here. For this lab, I will use the default CloudFront certificate (*.cloudfront.net), 

so I do not need to specify a custom certificate.

• Supported HTTP versions: Select HTTP/2.

• Default root object: Enter index.html

• IPv6: Off

Note: This is the café website homepage hosted in the S3 bucket.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/2070e614-0629-4d56-b053-564799600aa9" />

Finally, choose Create distribution. While the distribution is being created, Last modified at the top of the page displays Deploying as shown in the following image.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/a70ee8c8-55bd-4d36-a3a7-e6ca9add5831" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/338b7e6d-2be1-4a26-bc3d-e001719ef245" />

Note: It might take 5 to 15 minutes for the deployment to complete. However, I can continue to the next step in the lab. 

Update the bucket policy.

• In a new browser tab, navigate to the Amazon S3 console.

• Choose the link for the bucket that has -s3bucket in the name.

• Choose the Permissions tab.

• In the Bucket policy section, choose Edit.

In the Policy code section, delete lines 4 through 17, which are highlighted in the following image. 

<img width="955" alt="image" src="https://github.com/user-attachments/assets/1279385d-711f-4c3f-98f4-7f833fec5803" />

In the Bucket policy section, choose Edit. In the Policy code section, I delete lines 4 through 17, which are highlighted in the following image. 

![image](https://github.com/user-attachments/assets/c877eb78-93a5-41d1-aa43-b44e3f5318c2)

<img width="959" alt="image" src="https://github.com/user-attachments/assets/5de3bb24-39d1-4b74-84ac-0d9c2c5e4af6" />

At the bottom of the page, choose Save changes.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/b90b5e1d-ee8d-40b1-9897-e24526615e2a" />

I will now retest access to the café website after the update to the bucket policy. Return to the browser tab where the café website is open, and refresh the browser page. I no longer see the site. Instead, I see an AccessDenied error similar to the following image.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/1bfd46b5-5edf-496f-bed5-ac8d649d0e1e" />

Note: This outcome is expected, because I removed the lines from the bucket policy that granted s3:GetObject access for requests coming in from my IP address. I don't want anyone to access the website directly using the Amazon S3 URL. Instead, I want users to access the site through CloudFront.

Next is to verify that the CloudFront distribution is now enabled.

•  Return to the CloudFront console.

• In the left navigation pane, choose Distributions.

•  Verify that the distribution I created now has a Status of Enabled, as shown in the following image.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/2c308266-4afe-4a07-8d54-f92974e64943" />

To Test the CloudFront distribution.

•  Choose the link for the distribution ID.

•  Copy the Distribution domain name URL and open it in a new browser tab.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/23b2e687-d292-4c0a-aa78-303136a5cda1" />

The café website displays. 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/8b5b52ba-4665-41fe-a976-4cf7bed6ee16" />

I try to load the website on a mobile device that is not connected to the same network as my computer. For example, the device is connected to the internet through a cellular connection and not the same WiFi network that my computer is connected to. I notice that you can still access the site.
      
Tip: An alternate way to test that the café website is available from another network is to use the AWS Cloud9 IDE. In the terminal, run wget <distribution-domain-name> where <distribution-domain-name> is the distribution URL. If the HTTP request returns an HTTP status code of 200, then the site is available from the AWS Cloud9 instance, which runs on a different network than my computer.

    wget https://d2lqhwjgthgncl.cloudfront.net

<img width="938" alt="image" src="https://github.com/user-attachments/assets/d2efb64c-a77c-41f1-83d3-6b98a8a93933" />

I have successfully created a CloudFront distribution. The website is running on a secure HTTPS connection, and I secured the site so that it is only available through CloudFront. 

<h2>Task 3: Securing network access to the distribution using AWS WAF</h2>










































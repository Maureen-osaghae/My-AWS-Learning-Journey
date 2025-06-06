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

In this task, I will configure the website so that it can only be accessed from a specific IP address range again. In the café scenario, this is the IP address range that Sofía uses to connect to the internet.

Sofía recall that the S3 bucket policy enforced this previously. Now that the website is configured to be accessed through CloudFront, Sofía needs to find an alternative way to implement this. <img width="854" alt="image" src="https://github.com/user-attachments/assets/5a3e1f5d-4642-45e3-a9e2-da10053dae17" />

Sofía will use the AWS WAF service to create an access control list (ACL) to restrict access to the café website. Playing the role of Sofía, First, I will create an IP set for my IP address. In the AWS Management Console, I search for waf and choose WAF & Shield.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/900d2816-b649-46ef-92c7-144e1b034cef" />

In the left navigation pane, I choose IP sets.

• Choose Create IP set and configure: In the left navigation pane, choose IP sets.

• Choose Create IP set and configure:

◦ IP set name: Enter office

◦ Description: Enter office IP
        
◦ Region: Choose Global (CloudFront).
        
◦ IP addresses: Enter <ip-address>/32 where <ip-address> is my public IPv4 address, as identified by whatismyipaddress.com.
          
◦ Choose Create IP set.

<img width="854" alt="image" src="https://github.com/user-attachments/assets/ec9b6b6d-7c97-4a7f-bb1a-a2e28b4e1991" />

![image](https://github.com/user-attachments/assets/77bb1da3-ec74-4022-b420-0f6ec18b113d)

<img width="959" alt="image" src="https://github.com/user-attachments/assets/b94b9209-2ea3-4852-9730-ec3496b24c54" />

Begin to create a web ACL.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/45263b6f-a317-4dd5-8e4c-3dfaf57f5514" />

• In the left navigation pane, choose Web ACLs.

• Choose Create web ACL.

• In the Web ACL details section, configure:

◦ Resource type: Choose CloudFront distributions.
        
◦ Name: Enter cafe-website-office-only-during-dev
        
◦ Description: Enter Allow access to the cafe website through CloudFront from the cafe office
    
• CloudWatch metric name: Enter cafe-website-office-only-during-dev

<img width="959" alt="image" src="https://github.com/user-attachments/assets/003b987a-5c3a-4446-8651-0bba0703bac2" />

In the Associated AWS resources section, configure:

◦ Choose Add AWS resources.

◦ Select the CloudFront distribution that you created.

◦ Choose Add.
        
◦ Select the CloudFront distribution again, and then choose Next.

<img width="605" alt="image" src="https://github.com/user-attachments/assets/5e0f2d3b-dfb4-4bb6-bc7b-0ed64453b295" />

Add a rule to the web ACL configuration to allow requests from the office IP set. In the Rules section, choose Add rules, Add my own rules and rule groups.
   
• Rule type: Choose IP set.

• Name: Enter only_office_please

• IP set: Choose the office IP set that I just created.

• IP address to use as the originating address: Keep the default Source IP address setting.

• Action: Choose Allow.

• Choose Add rule.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/9698922c-b851-4b47-95c8-39b58653f29a" />

Set the rule priority, configure metrics, and create the web ACL.

◦  Choose the only_office_please rule.

◦ Choose Next.

◦ Keep all of the default metrics settings, and choose Next again.

◦ Review the settings, and at the bottom of the page, choose Create web ACL.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/8bedf007-dd7b-4667-b774-220e1c5c48dc" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/afaf5a14-130d-450f-a27e-1d074a9db0de" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/02070506-d3d2-4701-af3e-d9e50e587f26" />

Confirm that the web ACL configuration has been applied to the CloudFront distribution.
       
◦ Return to the CloudFront console.

◦ In the left navigation pane, choose Distributions.

◦ Note: The Last modified column might display Deploying for the distribution. The deployment will complete within about 5 minutes; however, I can proceed to the next step without waiting.

◦ Choose the link for the distribution ID.

◦ In the Security section, I notice that the distribution now has an AWS WAF value.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/852655d0-787f-417b-8f34-8cce6ec8cdab" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/2731e591-0dfc-44dd-a953-5d4725a36f6b" />

Test the AWS WAF configuration that was applied to the CloudFront distribution. Return to the browser tab where the café website is open, and refresh the browser page. The website displays, because my computer's IP address is in the IP set that I specified when I configured the web ACL.
Next, I try to load the website on a mobile device that is not connected to the same network as your computer. For example, the device is connected to the internet through a cellular connection and not the same WiFi network that my computer is connected to.

Next, I try to load the website on a mobile device that is not connected to the same network as my computer. For example, the device is connected to the internet through a cellular connection and not the same WiFi network that my computer is connected to.
I could not access the site through a different network now.

![image](https://github.com/user-attachments/assets/cf0dceea-c75c-4684-aacd-20a0b8744e1e)

I have blocked direct access to the website through Amazon S3, configured a global CDN, and also secured the website to prevent anyone who isn't using my IP address from viewing the café website during this development phase.

<h3>Analysis: </h3>
At this point, because my CloudFront distribution URL has an obscure value, someone would need to both guess the URL and use my IP address to access the website. Doing this would be difficult for anyone other than myself. However, to truly secure the site, I should configure a login system. In a later lab, I will use the Amazon Cognito service to implement authentication and to secure parts of the website.

</h2>Task 4: Securing a REST API endpoint using AWS WAF</h2>

The café website is now configured so that it can only be viewed from the café office network (my IP address). However, the REST API URLs that the website's AJAX use are not yet secured and could be invoked from anywhere on the internet.
In this task, I will secure access to one of the REST API endpoints.

Create a regional AWS WAF IP set. 

• Return to the WAF & Shield console.

• In the left navigation pane, choose IP sets.

• Choose Create IP set and configure:

◦ IP set name: Enter office_regional

◦ Description: Enter IP of the office for API Gateway

◦ Region: Choose US East (N. Virginia).
        
◦ IP addresses: Enter <ip-address>/32 where <ip-address> is your public IPv4 address, as identified by whatismyipaddress.com.

<img width="678" alt="image" src="https://github.com/user-attachments/assets/c0dabf08-b801-4fd1-8211-5653e6b14b4f" />

Choose Create IP set.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/710862b4-bc5f-499d-99e3-6bae583a5d9c" />

Begin to create a regional web ACL.
   
• In the left navigation pane, choose Web ACLs.
   
• Choose Create web ACL.
    
• In the Web ACL details section, configure:
       
◦ Resource type: Choose Regional resources.

◦ Region: Choose US East (N. Virginia).

◦ Name: Enter website-api-gw-office-only-during-dev
        
◦ Description: Enter To allow us to access the API GW calls used by the website from the office
        
◦ CloudWatch metric name: Enter website-api-gw-office-only-during-dev

<img width="959" alt="image" src="https://github.com/user-attachments/assets/d37d7e68-794a-4c29-8191-100a06264609" />

In the Associated AWS resources section, configure:

◦ Choose Add AWS resources.

◦ For Resource type, select Amazon API Gateway.

◦ Select the ProductsApi - prod API Gateway resource.

◦ Choose Add.
        
◦ Select the ProductsApi - prod resource again, and then choose Next.

<img width="632" alt="image" src="https://github.com/user-attachments/assets/350e733c-3870-4e8c-855f-27b32f621e21" />

Add a rule to the web ACL configuration to allow requests from API Gateway.

◦ In the Rules section, choose Add rules, Add my own rules and rule groups.

◦ Rule type: Choose IP set.

◦ Name: Enter ip_for_apigw

◦ IP set: Choose the office_regional IP set that I just created.

◦ IP address to use as the originating address: Keep the default Source IP address setting.

◦ Action: Choose Allow.

◦ Choose Add rule.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/00ca72e0-6150-4d5f-a948-3c1475827581" />

Update the new web ACL rule to block any requests that don't match the rule.

◦ In the Rules section, select the ip_for_apigw rule.

◦ In the Default web ACL action section, for Default action, choose Block.

◦ Choose Next.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/6b59a41f-c9d3-42b5-93e5-5577f52b1a1e" />

Set the rule priority, configure metrics, and create the web ACL.

◦ Choose the ip_for_apigw rule.

◦ Choose Next.

◦ Keep all of the default metrics settings, and choose Next again.

◦ Review the settings, and at the bottom of the page, choose Create web ACL.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/f1982e17-17ac-4631-8f66-49f9aeb6c5f1" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/714de6cd-3572-42d2-a05b-4adb87988f87" />

When the web ACL creation process is complete, check the resources associated with the ACL.

◦ Choose the link for the website-api-gw-office-only-during-dev ACL, which you just created.

◦ Choose the Associated AWS resources tab.

◦ Confirm that the ProductsApi - prod resource is listed. If it isn't, add it:

▪ Choose Add AWS resources.

▪ Choose the ProductsApi - prod resource.

▪ Choose Add.
    
• In a new browser tab, go to the API Gateway console.

<img width="956" alt="image" src="https://github.com/user-attachments/assets/a2dbb9b1-ba66-441f-afe2-271f37ce9507" />

Test the new ACL from my computer.
  
• In a new browser tab, go to the API Gateway console.

• Choose the link for the ProductsApi.

• In the left navigation pane, choose Stages.

• In the Stages navigation pane, expand the prod stage.

• Under /bean_products, choose GET.

<img width="953" alt="image" src="https://github.com/user-attachments/assets/32f21c60-a24c-45c6-8c39-912a201fb1bc" />

I copy the Invoke URL, which has the format https://osztaz88f3.execute-api.us-east-1.amazonaws.com/prod/bean_products, and load the URL in a new browser tab. 

A JSON-formatted document with product information displays. This is the expected behavior. 

<img width="698" alt="image" src="https://github.com/user-attachments/assets/f33b4663-bbfb-48fe-846e-4dc1b9eea14c" />

Test the new ACL from another network.

 On the browser tab on my computer where I just opened the invoke URL, I open the context menu (right-click) for the page and choose Create QR Code for this page as shown in the following image.

A pop-up window displays a QR code as shown in the following image. I will keep the window open and continue to the next step.

<img width="237" alt="image" src="https://github.com/user-attachments/assets/a8b74e45-42f4-4982-8823-ff8bb8778d93" />

Verify that my mobile device is not connected to the same network as my computer.
    
• On my mobile device, I use the camera app or a QR Code reader app to scan the QR Code on my computer.
   
• The app prompts me to load the link. The page displays {"message": "Forbidden"}. This is the expected behavior.

![image](https://github.com/user-attachments/assets/617c1db2-0fe1-4c39-944a-a1e6a76ddf34)

The AJAX URI that the café website uses is now also secured so that it can only be invoked from the café network (my computer). In a later lab,  I will secure access to the coffee suppliers web application. 

<h3>Analysis: </h3>
To use AWS WAF to make the suppliers application available only from the café's office IP address, I would need to use an Application Load Balancer. To use a load balancer, I would need to upgrade the Elastic Beanstalk application. However, because the suppliers website is not meant for public access, even when the website moves to production, this method would not be sufficient to try to secure the website. So I decides to leave the suppliers website configuration as it is for now. In a later lab, I will implement authentication to properly secure that part of the site. 

<h2>How the café plans to use a CloudFront function</h2>
 This section explains how the café will use a CloudFront function. I don't have any steps to complete until I get to the next task.
Frank is pleased that the café website has been secured and cannot be accessed outside of the café's office network during this development phase. However, it seems that an effective cloud developer's work is never done because Frank has one more request.

He can't decide which promotional image to use on the homepage. He likes the apple pie image, but he also likes the pictures of his homemade pastries. He has asked Sofía (me) if it's possible to randomly display an image from a collection when the site is loaded.

Sofía first considered using JavaScript code on the client side to randomize the image. However, she chatted with Faythe, the AWS developer who often comes into the café to get a morning coffee. Faythe mentioned that Sofía could use the CloudFront Functions feature to achieve the same result.
Sofía could configure a CloudFront function to be invoked each time a request comes into (or out of) CloudFront. Therefore, she could manage the randomize logic at the edge rather than bloating the website with randomizing code logic.

To invoke the function, Sofía can add code such as the following to the website:

 function changeImage(){
        
        var pastry_name_str = "apple_pie";

        if(document.cookie !== "" && document.cookie.split('=')[1] !== ""){
            pastry_name_str = document.cookie.split("=")[1];
        }
        $("[data-role2='special_highlight']")
                .css({
                    "background-image": "url(/images/items/" + pastry_name_str + ".png)"
                });
    }

This code checks for the existence of a cookie, which CloudFront Functions will add. If the cookie exists, the website will dynamically swap the image to display. 
The CloudFront Functions code has already been written and made available for me in the resources/website/scripts.main.js file, so I don't need to update the website code in this lab. However, I will need to send a special cookie to the CloudFront distribution. The cookie will reference an image that can be used to override the default image on the website. If the cookie isn't found, the website will display the apple pie image. 

<h2>Task 5: Configuring a CloudFront function on the website</h2>

In this final task, I will create a new CloudFront function, add code to the function to run each time that the café website is loaded and then test the website to verify that the desired result has been achieved.

To Create a CloudFront function.
   
• Navigate to the CloudFront console.
    
• In the left navigation pane, choose Functions.
    
• Choose Create function.
    
• For Name, enter random_image_header

<img width="959" alt="image" src="https://github.com/user-attachments/assets/476a950d-05ee-4ead-8562-f087fb1d69e1" />

In the Function code section, replace the existing code with the following code:

    function handler(event) {
      var
        pastry_name_arr = [
            "apple_pie",
            "apple_pie_slice",
            "chocolate_chip_cupcake",
            "strawberry_cupcake",
            "blueberry_jelly_doughnut",
            "plain_bagel"
        ],
        random_int = Math.floor(Math.random() * (pastry_name_arr.length)),
        response = event.response,
        date = new Date(),
        attr_str = "",
        distro_str = "dp880v3pwdoto"; //change this
    
    date.setTime(+ date + (365 * 86400000)); //24 \* 60 \* 60 \* 100
    
    attr_str = "Secure; Path=/; Domain=" + distro_str + ".cloudfront.net; Expires=" + date + ";";
    
    response.cookies = {
        "the_image": {
            "value": pastry_name_arr[random_int],
            "attributes": attr_str
        }
    };
    
    return response;
    }

<img width="959" alt="image" src="https://github.com/user-attachments/assets/a5275f46-c4af-4b5e-9d01-d606e91e4cfe" />

<img width="959" alt="image" src="https://github.com/user-attachments/assets/237675d7-45aa-4931-a3f1-24ecafd32d95" />

In the code I pasted,  replace the distro_str value of dp880v3pwdoto on line 15, with your unique distribution string value. 

Test the function.

◦ Choose the Test tab.

◦ For Event Type, choose Viewer Response.

◦ Keep the other default settings, and at the bottom of the page, choose Test function.

◦ A results area appears at the bottom of the page. Confirm that the Status is 200 OK. The output should show that a single random item from the pastry_name_array was returned in the cookie.

<img width="779" alt="image" src="https://github.com/user-attachments/assets/6602e68a-30fd-478a-8667-34e0811487a4" />

<img width="956" alt="image" src="https://github.com/user-attachments/assets/5641e855-123b-47ac-a321-e8a9f1dc09c3" />

I publish the CloudFront function, and associate it with the CloudFront distribution.
   
• Choose the Publish tab.
   
• Choose Publish function.

A message at the top of the page indicates that the random_image_header function was successfully published.

<img width="956" alt="image" src="https://github.com/user-attachments/assets/0cd243c8-552c-410f-ba9b-5164aded98f3" />

In the Associated distributions section, choose Add association and configure:
        
◦ Distribution: Choose the distribution.
        
◦ Event type: Choose Viewer Response.
        
◦ Cache behavior: Select Default (*).
        
◦ Choose Add association.

<img width="408" alt="image" src="https://github.com/user-attachments/assets/8b63d3a0-a0c4-47ce-8a80-82be02e46951" />

In the left navigation pane, choose Distributions. I notice that the distribution is being deployed again.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/27ae5a59-7508-4e52-bae0-fb56b25268fe" />

To Test the functionality on the café website. I Return to the café website browser tab, and refresh the page a few times. Each time I refresh, the graphic that appears to the right of the cup of coffee should change.

<img width="956" alt="image" src="https://github.com/user-attachments/assets/a3820174-ce9a-4573-b200-85d0c18b26c9" />

<img width="958" alt="image" src="https://github.com/user-attachments/assets/56e794a2-b6d7-487e-bdf7-8edaa35362c9" />

I have successfully implemented a CloudFront function on the website.

<h2>Task 6: Adjusting the cache duration</h2>

Recall from earlier in the lab that all objects in the website S3 bucket have a metadata key-value for Cache-Control with the value set to max-age=0. In addition, recall that when I configured the CloudFront distribution, I chose to use the legacy cache settings and let the origin cache headers determine the object caching behavior. I also configured the CloudFront distribution with a path pattern of *, which means that the distribution will forward all object requests to the origin (the S3 bucket). 

During development, these configurations helped Sofía to immediately notice the effect of any changes made. However, now that the distribution has been tested and found to be stable, Sofía has decided to adjust the caching settings. 

Confirm the cache settings that are in place on the café website.

• Return to the café website browser tab.

• Open the context menu (right-click) on the page and choose Inspect (if using Chrome) .

<img width="959" alt="image" src="https://github.com/user-attachments/assets/1059bc55-ae9f-403b-8471-b3c8e13e2149" />

In the developer tools area that appears,  choose the Network tab. Next, refresh the café website by using the browser's refresh icon. A list of all the files and accessed locations displays in the developer tools area.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/bcd9c254-1d8d-4cd4-bb4f-0d7e9548bcb8" />

At the top of the file list, choose the CloudFront distribution URL (the URL ends in cloudfront.net).

Analyze the Response Headers information.
       
◦ Notice the header cache-control: max-age=0.
       
◦ Also notice the header x-cache: RefreshHit from cloudfront. (In Firefox, it says Miss from cloudfront.) This means that a matching cache key wasn't found in the cache.

<img width="957" alt="image" src="https://github.com/user-attachments/assets/e12dde2e-d324-4029-bc6f-4ff5f1917828" />

Sofía has a functional CloudFront distribution, but the configuration isn't taking advantage of the caching features. She could remove the Cache-Control metadata from all of the S3 objects and then update the behavior settings in the CloudFront distribution to use the CloudFront CachingOptimized managed policy. That would apply a default time to live (TTL) of 86,400 seconds (24 hours) to the requested objects. However, that cache setting would be too long for testing purposes.

Sofía decides to begin by applying a caching file expiration time of 3 minutes for testing purposes. She decides to make this change by updating the origin response headers. In this case, the origin is the S3 bucket, and the response headers are configured by updating the Cache-Control key-value pairs set on the Amazon S3 objects. After Sofía finishes testing, she can remove the Cache-Control metadata from the objects in the S3 bucket and use the CloudFront cache settings to control the cache file expiration behavior.

To edit the Cache-Control header set on each object in the S3 bucket.
    
• Navigate to the Amazon S3 console.
    
• Choose the link for the bucket that has -s3bucket in the name.
    
• To select all of the files in the bucket, check the box at the top left of the objects list, to the left of the Name column, as shown in the following image.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/8dbcdbfa-40b5-4e79-814a-f39edb529ad0" />

To edit metadata
Choose Actions, Edit metadata.

• Choose Add metadata and configure:

◦ Type: Choose System defined.

◦ Key: Choose Cache-Control.

◦ Value: Enter max-age=180 (This is 180 seconds, or 3 minutes.)

<img width="959" alt="image" src="https://github.com/user-attachments/assets/851b3e05-8326-4a79-a404-d743a7b8fa6c" />

At the bottom of the page, choose Save changes.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/080a0b73-1736-4282-9d03-016efd8876ee" />

Test the effects of updating the caching settings on the CloudFront origin (the S3 bucket).

• Reload the café website.
   
• In the developer tools area, notice that the new max-age setting of 180 seconds was applied to the cache.
   
• Notice that x-cache now says Hit from cloudfront, as shown in the following image.
      
A hit indicates that the cache key matches the request, and the object was served from the CloudFront edge cache.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/7dc5f77b-de0c-4d07-a944-742527e1ba89" />

Refresh the webpage a few times to observe how it behaves. I also notice that the image at the top right of the page still changes on each page refresh, even though the page is fully cached.

<h2>Conclusion</h2>
The site will load faster, because CloudFront uses edge locations and can cache web content. By using AWS WAF features, I updated the site so that it is only available on the café office network during this development. I learned to modify how long website files are cached. Finally, I used the CloudFront Functions feature to rotate which dessert is highlighted on the website. Not bad for a day's work! Thanks to AWS re/Start Post Graduate training. 

<h5>© 2025 Amazon Web Services, Inc. and its affiliates. All rights reserved. This work may not be reproduced or redistributed, in whole or in part, without prior written permission from Amazon     Web Services, Inc. Commercial copying, lending, or selling is prohibited.</h5>












      
































      












































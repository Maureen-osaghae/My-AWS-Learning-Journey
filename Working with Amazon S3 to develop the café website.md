<h1>Securing and Deploying Static Websites on Amazon S3 with IP-Based Access Control </h1>
<h2>Lab overview and objectives</h2>
In this lab, you use Amazon Simple Storage Service (Amazon S3) to host a static website. You will also implement architectural best practices to protect and manage your data.

<h3>What I Learned:</h3>
<ol>
    <li> Create a static website by using Amazon S3. </li>
    <li> Apply a bucket policy on an S3 bucket to configure customized data protection.</li>
    <li> Upload objects to an S3 bucket by using the AWS SDK for Python (Boto3).</li>
    <li> Configure the website that is hosted on Amazon S3 to be accessible only from a specific IP address range, and test the configuration.</li>
</ol>

<h3>Business Scenario</h3>

Sofía is eager to start building a website for the café. She has some Python development skills and she's learning more about how to develop solutions on the cloud. Nikhil is a secondary school student who also works at the café. He has some skills in graphic design and a strong interest in learning about cloud computing.
The café has a single location in a large city. It mostly gains new customers when someone walks by, notices the café, and decides to try it. The café has a reputation for high-quality desserts and coffees, but their reputation is limited to people who have visited, or who have heard about them from their customers.

<img width="383" alt="image" src="https://github.com/user-attachments/assets/b9d96218-d83d-43ec-9d75-77ae962db30b" />

Sofía suggests to Frank and Martha (her parents and the owners of the café) that they should expand community awareness of what the café has to offer. The café doesn’t have a web presence, and it doesn’t currently use any cloud computing services. However, that situation is about to change.
In this lab, you will play the role of Sofía and take the first steps needed create a website for the café. It's time to get started!
When you start the lab, the following resources are already created for you in the AWS account:

<img width="416" alt="image" src="https://github.com/user-attachments/assets/a0128611-a495-4387-af8e-bf593ef8b0be" />

At the end of this lab, your architecture should look like the following example: 

<img width="427" alt="image" src="https://github.com/user-attachments/assets/3eb41124-745a-4ba7-a91f-20820210d75a" />

<h3>A business request for the café: Create a static website</h3>

Sofía mentions to Nikhil that she would like to build a proof-of-concept website for Frank and Martha that will showcase the café's offerings visually.
 It would also provide business details, such as the location of the store, business hours, and telephone number. Nikhil is eager to see how she will build the website, and he agrees to create the graphics that she will use.
For this first challenge, you will take on the role of Sofía. You will use the AWS Command Line Interface (AWS CLI) and the SDK for Python to configure Amazon S3 so that it hosts a basic website for the café.

<h2>Task 1: Connecting to the AWS Cloud9 IDE and configuring the environment</h2>

In this first task, you will configure the AWS Cloud9 environment to support the development that you will work on during the rest of the lab.
       Connect to the AWS Cloud9 IDE.
      <Ol>
       <li> From the Services menu, search for and select Cloud9.</li>
          You should see an existing IDE that's named Cloud9 Instance.
        <li>  Choose Open. </li> 
          The AWS Cloud9 IDE loads in a new browser tab.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/489c6ef5-a6d1-4858-b459-4867adf44336" />

<h2>Install the AWS SDK for Python.</h2>

In the AWS Cloud9 bash terminal (located at the bottom of the IDE), run the following commands:
sudo pip install boto3

<img width="698" alt="image" src="https://github.com/user-attachments/assets/bad44c93-ab54-4031-a113-e9cccaacc469" />

Download and extract the files that you will need for this lab.
In the same terminal, run the following command:

wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCDEV-2-44869/02-lab-s3/code.zip -P /home/ec2-user/environment

he code.zip file is downloaded to the AWS Cloud9 instance. The file is listed in the left navigation pane.
<img width="956" alt="image" src="https://github.com/user-attachments/assets/712c3d41-160d-492b-8664-c5781007e715" />

    • Run the following commands to extract the file:
unzip code.zip

<img width="959" alt="image" src="https://github.com/user-attachments/assets/d0c3de0d-929f-4699-8053-c3c44e7e0d2a" />

Tip: You will use the files that you downloaded and extracted later in this lab. 
Verify the AWS CLI version 2.
    • Verify the CLI version:
aws --version

<img width="533" alt="image" src="https://github.com/user-attachments/assets/37e83b0a-3754-4ec3-8635-336de850fdf5" />

<h2>Task 2: Creating an S3 bucket by using the AWS CLI</h2>

In this task, you will create an S3 bucket to host your website. You will complete this task by using the AWS CLI
       In the AWS Cloud9 Bash terminal, run the following command, replacing <bucket-name> with your bucket name.
       For the bucket name, use the following items, separated by dashes (-):
       <ol>
          <li>Your initials in lowercase</li>
          <li>Today's date in the format YYYY-MM-DD</li>
          <li>The word s3site</li>
          </ol>

For example, Sofía Martínez might name the bucket sm-2022-08-26-s3site. mo-2024-12-11-s3site
aws s3api create-bucket --bucket mo-2024-12-11-s3site --region us-east-1

The terminal should confirm that the bucket was created by returning output similar to this image example: 
<img width="606" alt="image" src="https://github.com/user-attachments/assets/71be265c-98e7-4808-9991-a9d8d7d60353" />

<span>Tip:</span>

Copy the bucket name to a text file in the AWS Cloud9 IDE or on your computer so that you can easily copy it during later steps in this lab. The bucket name does not include the leading slash (/) that was returned in the location name-value pair.

Use the AWS Management Console to confirm that the bucket was created, and observe the current permissions settings on the bucket.
    • In the AWS Management Console, choose Services and select S3. You should see that the bucket was created.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/2e529a6e-5943-4aad-a4a8-2a23036ff08c" />

The Access column for the bucket indicates that Bucket and objects are not public.
Choose the bucket name and then choose Permissions. Notice that Block all public access is turned on.

<img width="950" alt="image" src="https://github.com/user-attachments/assets/ee4b0589-a1db-4919-afde-e6549a5d95ee" />

Choose Edit, de-select Block all public access.
<ol>
    <li> Select Block public access to buckets and objects granted through new access control lists (ACLs).</li>
    <li>Select Block public access to buckets and objects granted through any access control lists (ACLs).</li>
    <li>Select Block public and cross-account access to buckets and objects through any public bucket or access point policies.</li>
</ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/1523b11a-d80d-43c4-ac9d-985d6b7308f0" />

    • Choose Save changes.
    
In the confirm settings dialog, type in confirm and choose Confirm. Note: you will configure the bucket with a bucket policy later in this lab.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/8971cb3f-7ecb-4468-b4cd-2d99baef2fde" />

Note: you will configure the bucket with a bucket policy later in this lab.
Scroll further down and observe that the bucket currently doesn't have a bucket policy attached to it.

<h2>Task 3: Setting a bucket policy on the bucket by using the SDK for Python</h2>

Now that Sofía has created the bucket, she wants to set permissions on the bucket. She wants Frank, Martha, and Nikhil to be able to access the first version of the website that she's building so that they can see the progress she's making. However, she doesn't want to expose the site to the outside world yet. She will accomplish this objective by configuring a bucket policy on the bucket. This bucket policy will enforce a condition. 

       Create the policy document.
  <ol>
          <li> In the AWS Cloud9 IDE, in the navigation pane, choose the Cloud9 Instance directory.</li>
          <li> Choose File > New File and then choose File > Save. </li>
          <li> Name the empty file website_security_policy.json and choose Save. </li>
          <li> In the website_security_policy.json file, paste the following code: </li>
  </ol>   

<img width="527" alt="image" src="https://github.com/user-attachments/assets/c78daabc-6474-4fdf-a84f-1027c1f3aa42" />

      {
            "Version": "2008-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": "*",
                    "Action": "s3:GetObject",
                    "Resource": [
                        "arn:aws:s3:::<bucket-name>/*",
                        "arn:aws:s3:::<bucket-name>"
                    ],
                    "Condition": {
                       "IpAddress": {
                            "aws:SourceIp": [
                                "<ip-address>/32"
                            ]
                        }
                    }
                },
                {
                    "Sid": "DenyOneObjectIfRequestNotSigned",
                    "Effect": "Deny",
                    "Principal": "*",
                    "Action": "s3:GetObject",
                    "Resource": "arn:aws:s3:::<bucket-name>/report.html",
                    "Condition": {
                        "StringNotEquals": {
                           "s3:authtype": "REST-QUERY-STRING"
                       }
                   }
                }
            ]
     }



In the policy document, replace all three <bucket-name> entries with your actual bucket name.
Finally, replace <ip-address> with the IP address that is being used to connect your computer to the internet. You can find your IP address by visiting whatismyip.com 

Note: you won't be able to view the website if you use 0.0.0.0 as the allowed aws:SourceIp, because Amazon S3 is smart enough to recognize that means public access, and since you just set the bucket permissions to Block all public access, S3 will block your access. Therefore ensure you use your specific IPv4 address. 

<h3>Analysis:</h3> 
This policy document consists of two statements. The first statement allows GetObject requests from your IP address. The second statement denies access to an object in the bucket that's named report.html, unless a specific condition is met. The report.html file doesn't exist in the bucket yet. However, in a later lab, you will upload the report.html file to the bucket and configure presigned URL access to it. The later lab will explain the details. 

Apply the bucket policy to your bucket by using the SDK for Python.
<ol>
    <li>In the navigation pane, expand the python_3 directory and open the permissions.py file (which contains starter code).</li>
    <li>Edit the file by replacing <bucket-name> with the name of your bucket.</li>
    <li>Save the changes.</li>
</ol>
Save the file. 

<img width="710" alt="image" src="https://github.com/user-attachments/assets/f40f46cd-e6c1-4ebb-9b09-bb5a3a826a4d" />

 Finally, in the terminal, navigate to the python_3 directory and run the following code:
 
cd python_3
python3 permissions.py

If the command completed successfully, you should see the message DONE in the terminal output.

<img width="956" alt="image" src="https://github.com/user-attachments/assets/ef64b5dd-5430-4b56-92be-bab06b2198a6" />

<h2>Task 4: Uploading objects to the bucket to create the website</h2>

Now that Sofía has configured permissions on the bucket, she needs to upload the website objects to the bucket. Fortunately, Sofía created the website code some time ago, and it's in the resources directory in the AWS Cloud9 IDE. Nikhil created the graphics for the website, and he's excited to see what the website will look like. Sofía is now ready to upload the files. To complete this task, she will use the AWS Cloud9 terminal to issue a simple recursive file upload command and disable caching (since the website is still being developed).

Run the code in the terminal. You should still be in the python_3 directory. Be sure to replace <bucket-name> below with your actual bucket name

aws s3 cp ../resources/website s3://<bucket-name>/ --recursive --cache-control "max-age=0"

<img width="959" alt="image" src="https://github.com/user-attachments/assets/9a2a8169-154e-47b6-8118-7e0c10a5d797" />

<h2>Task 5: Testing access to the website</h2>

Now that Sofía has uploaded the website files to the S3 bucket, she must test the site and verify that it loads when a user accesses it through a virtual secure endpoint. 
Load the website in a browser tab. Return to the browser tab with the Amazon S3 console. Choose your bucket name, and then choose Objects.
    
If the files you just uploaded do not display, choose the refresh icon to view them.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/829ca968-5af9-4bf0-a697-a3e9946b7c27" />

Choose the index.html file.

Copy the Object URL. It will be in the following format. 

https://<bucket-name>.s3.amazonaws.com/index.html

<img width="959" alt="image" src="https://github.com/user-attachments/assets/ed8ad46b-e957-4596-9e75-05c7d6881ba7" />

Choose the index.html file.
Copy the Object URL. It will be in the following format. https://<bucket-name>.s3.amazonaws.com/index.html

<img width="959" alt="image" src="https://github.com/user-attachments/assets/c12aaf98-ed1f-415c-85ff-4cefa8819643" />

Note: In this lab scenario, the S3 bucket you created is intentionally not configured for static website hosting (a feature available in the bucket properties). Instead, you will access the website using the Object URL of the index.html file. 

Verify that your website displays by pasting that full URI into your browser. Ensuring you are on the same network (as it will block anything other than the IPv4 you specified earlier) 

<img width="959" alt="image" src="https://github.com/user-attachments/assets/054690e9-e8b7-427d-ab94-ed2624e4c173" />

Try to access the same URL from a location outside of your network.

 Ways that you can test outside access:
 
 If you have mobile phone with a cellular network connection, try loading the same URL in a browser on the mobile phone.

Another way to test is by running the following command in the AWS Cloud9 Bash terminal (where <bucket-name> is the actual bucket name):

<img width="958" alt="image" src="https://github.com/user-attachments/assets/0230f2ee-7b62-410c-88e8-2519d5031588" />

<h3>Analysis: </h3>

Regardless whether you load the page from another location or use the curl command from the Cloud9 instance, an attempt is made to retrieve the page that you specified. It should return an AccessDenied error because your mobile device or the AWS Cloud9 instance (whichever one you chose to use) connects to the internet by using a different IP address than your computer.
The essential point is that you should only be able to access the website if the device uses the IPv4 address that's specified in the S3 bucket policy. In the café story, this IPv4 address is the café's network IP address.

Test that the website is loading. Back in the browser tab where the website loads correctly, choose Login on the top right of the header.
An alert will display saying No API to call This behavior is also expected at this point in the course.

<img width="952" alt="image" src="https://github.com/user-attachments/assets/777781d6-dc9c-4df8-9755-e75b737f06ba" />

<h3>Analysis:</h3>

In the next lab, you will begin to create an application programming interface (API) that the JavaScript code in this website will interact with. For now, however, that API doesn't exist yet.

<h2>Task 6: Analyzing the website code</h2>

Nikhil watched as Sofía configured the website. Now that the website is hosted and working, he asks Sofía to explain the code behind the site.
      In the AWS Cloud9 IDE, browse to the resources > website directory and open the index.html file.
  <ol>
          <li>Notice that this HTML page references a number of CSS files in the head section</li>
          <li>Towards the end of the body section, the code references a few JavaScript (.js) files.</li>
          <li>Some of these JavaScript files define the essential features of the site.</li>
          <li>One of these files is named config.js another one of them is named pastries.js.</li>
  </ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/4fb87ceb-a092-4d43-bbbd-2d382a61a499" />

Open the config.js file.
      It contains an object literal that looks like this:

      <img width="728" alt="image" src="https://github.com/user-attachments/assets/de109a5a-c678-46f0-a2fa-630698790fad" />

 As you continue through the labs in this course, at the appropriate time (for example, when you add Amazon API Gateway and Amazon Cognito to the website implementation) you will set values for these keys and overwrite the contents of this file on S3 with the changes. Then the behavior of the website will adapt to the enhancements accordingly. 

 Open the pastries.js file. 
 
 Notice the logic defined in the loadAllItems function, that if the API_GW_BASE_URL_STR has a value of null (which it currently does, as you saw in config.js), then this function will print the items contained in the file named all_products.json.
 
  Observe the print Items function details that are defined further down in the code.

  <img width="956" alt="image" src="https://github.com/user-attachments/assets/b1366ade-b2f2-46a8-b08c-aca95b8ec37a" />

  <img width="959" alt="image" src="https://github.com/user-attachments/assets/6473cb0d-c5a0-4026-830c-a77f8a9e0068" />

  Open the all_products.json file.
  <ol>
  </li> Notice that this JSON file contains information about all the products that display on the website.<Li>
  For now, this product information is stored in this file hosted on Amazon S3, however, in a later lab, you will migrate this product information into a database.
  <li> Optionally, browse the other files that make up the website, to get a sense of how the website code works. The files have been left unobfuscated so you can read them if you wish. However you do not need to understand this application code in order to complete this course.</li>

  <img width="882" alt="image" src="https://github.com/user-attachments/assets/5b7cf178-b271-4b7d-bdc0-1f7d4b471c4b" />

  <img width="946" alt="image" src="https://github.com/user-attachments/assets/e2a5b56f-454b-4455-a638-f518d695838b" />

  <img width="959" alt="image" src="https://github.com/user-attachments/assets/328a1210-3d1b-4583-b9bc-cdd0638069dd" />

  <h2> Update from the café </h2>

Sofía is pleased that she now has a basic website that's hosted on Amazon S3. Because it's still a work in progress, she made sure to configure the site so that it's only accessible from the café's network. Frank and Martha are both pleased with the results so far.

Sofía takes a minute to relax, but she's also already looking forward to the next task. She's planning to start developing the components that will make the site run dynamically by using REST API calls to a database backend (AWS Lambda and DynamoDB).





























    


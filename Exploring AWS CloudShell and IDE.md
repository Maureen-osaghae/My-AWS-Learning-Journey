<h1> Exploring AWS CloudShell and IDE </h1>
<h3>Lab overview and objectives</h3>

In this lab, you will take on the role of Sofía. You will connect to an AWS CloudShell environment and explore its capabilities. You will also connect to Visual Studio Code Integrated Development Environment (VS Code IDE), and explore the layout and its functionality.

<span>What I learned:</span>
<ol>
      <li>Connect to AWS CloudShell and run AWS Command Line Interface (AWS CLI) commands and AWS SDK code from it.</li>
      <li>Connect to the VS Code IDE and explore its functionality.</li>
      <li>Copy files to and from Amazon Simple Storage Service (Amazon S3), CloudShell, and VS Code IDE.</li>
      <li>Install the AWS SDK for Python (Boto3) on an VS Code IDE instance.</li>
      <li>Use the VS Code IDE to create and run code files.</li>
</ol>

<h2>Business Scenario</h2>
Frank and Martha are a married team who own and operate a small café business that sells desserts and coffee.

<img width="302" alt="image" src="https://github.com/user-attachments/assets/a24d4271-76af-4c11-9c8e-a5e23a7241a3" />

Their daughter, Sofía, works at the café. Sofía is pursuing a degree in cloud computing at a local university in the evenings and on the weekends. She has Python development skills and is learning more about how to develop solutions in the cloud. 

Sofía is eager to start developing a web presence for the café. She thinks that before she starts coding, it would be a good idea to decide on a development environment for developing and running her code. She decides to explore at least two options that are available on AWS. 

When you start the lab, the only pre-created resource in the AWS account is an empty S3 bucket.

<img width="301" alt="image" src="https://github.com/user-attachments/assets/0f55d852-c0ac-41c2-b27f-96affababd18" />


However, by the end of this lab, you will have explored VS Code IDE and performed the actions that are shown below: 

<img width="640" alt="image" src="https://github.com/user-attachments/assets/98bc7184-d28c-4a18-9901-777ef5b03931" />

<h2>Task 1: Exploring AWS CloudShell</h2>

<img width="650" alt="image" src="https://github.com/user-attachments/assets/001f1165-a4bf-4aba-ab5a-cc81b3915889" />

In the AWS Management Console, at the top of the screen, choose the AWS CloudShell icon. 

<img width="956" alt="image" src="https://github.com/user-attachments/assets/4ba1a7b7-f6f0-4925-b0f3-376f5d9b6f18" />

A new browser tab opens with the AWS CloudShell interface.
If a "Welcome to AWS CloudShell" pop-up window opens, choose Close.
It might take 1–2 minutes for the terminal to become available.
You should be able to access a terminal window with a prompt.

<img width="958" alt="image" src="https://github.com/user-attachments/assets/06375fbd-96f7-4119-9122-926a4f3e8d2e" />

Verify that the AWS CLI is installed.
<ol>
          <li>At the CloudShell prompt, run the following command: aws --version</li>
          <li>In the output after aws-cli, the version indicates that CloudShell is using AWS CLI version 2.x.x by default.</li>
<ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/290f6e4c-eb26-4ab0-b595-1981a5be9d82" />


Test the ability to run an AWS CLI command.
<ol>
        <li>Run the following simple AWS CLI command: aws s3 ls</li>
        <li>A list of the S3 buckets that exist in the account is returned.</li>
</ol>

An empty sample bucket was automatically created when you started the lab. The bucket name should appear in the result set.


<img width="959" alt="image" src="https://github.com/user-attachments/assets/6a29aacd-4ca4-434f-821b-c74cb780b65e" />       

  From the Actions menu, choose Tabs layout > Split into columns. A second terminal window opens. 
  
  This action demonstrates that you can have multiple terminal panels open at the same time.

<img width="956" alt="image" src="https://github.com/user-attachments/assets/dfb69f5d-455d-401d-b025-cda85585a93e" />


Test the ability to run SDK for Python code.
     Open the context (right-click) menu for the following link, and download the file to your computer:
    
    list-buckets.py (https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCDEV-2-44869/01-lab-cloud9/s3/list-buckets.py)

From the Actions menu, choose Files > Upload file, and then choose Select file. 
      In the File Upload window, scroll to the list-buckets.py file that you downloaded, choose it, and then choose Open.

 <img width="959" alt="image" src="https://github.com/user-attachments/assets/b6f640e8-e725-4c29-994b-be7c7c55365f" />

  Choose Upload.
<ol>
      <li>Close the File upload successful notification.</li>
      <li>In the terminal on the right side, run the following command: cat list-buckets.py</li>
      </ol>
      The output shows the contents of the file that you uploaded:

 <img width="959" alt="image" src="https://github.com/user-attachments/assets/0cd61c7e-9e52-40fd-9bc4-4fe8f6e210fe" />

  In the terminal on the right, run the SDK for Python code by issuing the following command: python3 list-buckets.py
      The name of the S3 bucket is returned.
      Compare this output with the AWS CLI command output in the terminal on the left.
      You have now used two different programmatic ways to retrieve a list of the S3 buckets that exist in the AWS account.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/074c0bfe-7afd-4293-aaad-d6bfa8c7ece1" />


Copy a file from CloudShell to an S3 bucket. Copy the name of the bucket that includes -sample-bucket- in the name. To copy the list-buckets.py file to the bucket, go to the terminal on the left and run the following command (replace <bucket-name> with your actual bucket name):
      aws s3 cp list-buckets.py s3://<bucket-name>
      If the upload is successful, an output similar to the following example is returned:
      upload: ./list-buckets.py to s3://<bucket-name>/list-buckets.py

<img width="472" alt="image" src="https://github.com/user-attachments/assets/932dbfde-a764-461b-aaff-e6e33dafea69" />


When you use AWS CloudShell, you have persistent storage of 1 GB for each AWS Region at no additional cost.
The persistent storage is located in your home directory ($HOME) and is private to you. If you run the df -H /home command in a terminal, the amount of storage that's available in your CloudShell environment is returned.
Data in your home directory persists between sessions. If you must store more than 1 GB, you can access an S3 bucket from CloudShell.

<img width="488" alt="image" src="https://github.com/user-attachments/assets/03f72bdc-23bf-47b8-87d8-7265d26d15bb" />


<h2>Update from the café</h2>

After playing the role of Sofia in this lab, I was impressed with how quickly I could run commands and code in AWS CloudShell. I can already envision how I can use it to run PowerShell and other scripts. However, to build a website, she wants to use a fully-featured integrated development environment (IDE) where she can visually write, edit, run, and debug her code. CloudShell provides the vi and vim terminal-based text editors, but it doesn't provide many of the additional features that Sofía is looking for.
Faythe, a friend of Sofía, is an experienced AWS developer and consultant. When Faythe visited the café this morning to buy pastries, Sofía mentioned her search for a development environment and how she explored CloudShell. 

Faythe was impressed that Sofía knew about CloudShell and agreed that it's a useful tool.
Meanwhile, Faythe suggested exploring the features of VS Code IDE. "Based on your description of the features that you want to use, I think you will like it! You can get started quickly by going to the VS Code IDE service page and launching an instance."
Sofía is eager to explore VS Code IDE which is what you will do next!

<h2>Task 2: Exploring VS Code IDE</h2>

In this task, you will connect to VS Code IDE and learn how to use it. You will also interact with Amazon Simple Storage Service (Amazon S3) from the VS Code IDE. Finally, you will author a Hello World webpage in VS Code IDE and host the webpage in Amazon S3. 
Copy values from the table for the following and paste it into an editor of your choice for use later.

<ol>
      <li>LabIDEURL</li>
      <li>LabIDEPassword</li>
      <li>In a new browser tab, paste the value for LabIDEURL to open the VS Code IDE.</li>
      <li>On the prompt window Welcome to code-server: </li>
      <li>Enter the value for LabIDEPassword you copied to the editor earlier</li>
      <li>Choose Submit to open the VS Code IDE similar to below.</li>
</ol>

Note: User Interface similar to the following is displayed.

<img width="860" alt="image" src="https://github.com/user-attachments/assets/fd9922de-319c-48c7-a3fe-3a702e3f57a7" />


Observe the VS Code IDE user interface (UI).

The bottom of the screen is a Bash terminal. This terminal provides functionality that's similar to AWS CloudShell.

The left side of the screen is the navigation pane, which shows the file system. This VS Code IDE environment runs on an Amazon Elastic Compute Cloud (Amazon EC2) instance that's now running in your AWS account.
          
Copy a file from Amazon S3 to your local storage in VS Code IDE by using the Bash terminal to run an AWS CLI command. 

<ol>
      <lI>In the Bash terminal get the name of your bucket by running the following command: aws s3 ls</li>
      <li>While copying values to the IDE, if prompted by the browser, choose Allow. </li>
      <li>IDE may also show different informatory prompts during your lab, Close the prompt by choosing X on the prompt window.</li>
      <li>Next, download the list-buckets.py file from Amazon S3 to the local storage on VS Code IDE by running the following command.</li>
      <li>Replace <bucket-name> with your actual bucket name. Also be sure to include the period (.) at the end of the command, which indicates that the file should be downloaded to the current directory.</li>
</Ol>

The list-buckets.py file should now be listed in the navigation pane.

<img width="958" alt="image" src="https://github.com/user-attachments/assets/619a6e26-06d2-4990-a140-f7566449a9d7" />

Open a code file that uses the SDK for Python and run it.

Double-click the list-buckets.py file so that it opens in the file editor window. The code displays.

In the Bash terminal window, run the following command to run the code:

python3 list-buckets.py

<img width="940" alt="image" src="https://github.com/user-attachments/assets/00befdd1-d107-4194-9378-ebdc3edfb85d" />


Open a code file that uses the SDK for Python and run it.

Double-click the list-buckets.py file so that it opens in the file editor window. The code displays. In the Bash terminal window, run the following command to run the code:

      python3 list-buckets.py
      python3 list-buckets.py


Create a new file and upload it to Amazon S3 by using the VS Code IDE CLI.

<ol>
      <li> In the navigation pane, choose menu, then choose File > New Text File.</li>
      <li>In the empty editor, add the following text: <body> Hello World. </body>
      <li>From the navigation pane, choose menu, then choose File > Save.</li>
      <li>Enter the file name index.html</li>
</ol>

<img width="830" alt="image" src="https://github.com/user-attachments/assets/2993ac47-564b-4041-ac58-18cadf368536" />


<ol>
      <li>Choose OK to create file at the location /home/ec2-user/environment/ .</li>
      <li>Run the following command to upload the file to S3 bucket, replace with your bucket name:
      aws s3 cp index.html s3://<bucket-name>/index.html</li>
      <li>If the command is successful, you will see the message similar to the following </li>
      upload: ./index.html to s3://333333333-sample-bucket-638296109966/index.html</li>
</ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/fde8d571-3af4-4bd3-9fbf-79a20610c55e" />


<h2>Update from the café</h2>

<img width="445" alt="image" src="https://github.com/user-attachments/assets/c985f4fc-b586-4726-af4f-465b9fc19723" />

ASofía is pleased that she identified an IDE that has the features that she needs to develop the café website. She likes that VS Code IDE offers a graphical text editor, a file browser, a terminal for running AWS CLI commands, and code that uses the AWS SDKs. She's also glad that she knows about the features of CloudShell because she can open it from the AWS Management Console. CloudShell also provides some features that are similar to VS Code IDE but without the need to run an EC2 instance. 
In the next lab, Sofia will use VS Code IDE to accomplish her development objectives.

      import boto3
      
      BUCKET_NAME = 'YOUR_BUCKET_NAME'
      FILE_NAME = 'index.html'
      
      # setup s3 client named s3_client
      s3_client = boto3.client('s3')
      
      # create a function to put s3 bucket ownership controls with Rules set to BucketOwnerPreferred
      def put_bucket_ownership_controls():
          response = s3_client.put_bucket_ownership_controls(
              Bucket=BUCKET_NAME,
              OwnershipControls={
                  'Rules': [
                      {
                          'ObjectOwnership': 'BucketOwnerPreferred'
                      },
                  ]
              }
          )
          return response
          
      # create a function to set public access block values to false
      def set_public_access_block():
          
          response = s3_client.put_public_access_block(
              Bucket=BUCKET_NAME,
              PublicAccessBlockConfiguration={
                  'BlockPublicAcls': False,
                  'IgnorePublicAcls': False,
                  'BlockPublicPolicy': False,
                  'RestrictPublicBuckets': False
              }
          )
          return response
          
      # create a function to allow public access to FILE_NAME
      def allow_public_access_to_file():
          response = s3_client.put_object_acl(
              Bucket=BUCKET_NAME,
              Key=FILE_NAME,
              ACL='public-read'
          )
          return response
          
      # call the functions
      put_bucket_ownership_controls()
      set_public_access_block()
      allow_public_access_to_file()
      
      <h2> Lab Complete </h2>
      © 2024 Amazon Web Services, Inc. and its affiliates. All rights reserved. This work may not be reproduced or redistributed, in whole or in part, without prior written permission from Amazon Web Services, Inc. Commercial copying, lending, or selling is prohibited.
      






     










    









                                     

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









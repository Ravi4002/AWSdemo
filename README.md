Building a Math Operation Web App only using AWS Services
Only got AWS...no worries...that's all you'll need!!
Table of contents
Introduction
Tools and Services Used
Technical Architecture
Step-by-Step Implementation
Step 1: Setting Up the Front end with AWS Amplify
Step 2: Creating the Lambda Function
Step 3: Setting Up DynamoDB
Step 4: Creating a REST API with API Gateway
Step 5: Integrating Everything
Conclusion
Introduction
In this blog post, I'll walk you through the creation of a web application using various AWS services. This project involves deploying a front end using AWS Amplify, writing a Lambda function for computations, storing results in DynamoDB, managing permissions with IAM, and creating a REST API using API Gateway. By the end of this tutorial, you'll have a fully functional web app that computes mathematical operations and stores the results.

Tools and Services Used
AWS Amplify: For deploying the front end.

AWS Lambda: For back end computations.

Amazon DynamoDB: For storing computation results.

AWS IAM: For managing permissions and policies.

Amazon API Gateway: For creating a REST API to invoke the Lambda function.



Technical Architecture


Step-by-Step Implementation
Step 1: Setting Up the Front end with AWS Amplify
Create a New Amplify App

Go to the AWS Amplify " in the AWS Console and click "Get Started" under "Deploy."

Connect Your Repository

Connect your code repository (e.g., GitHub, Git Lab, Bit bucket) or choose "Deploy without Git provider" for manual deployments. In this case we will choose manual deployment.



Deploy Front end Code

Upload your front end code, which in this case is an HTML file for power base exponent calculations. Deploy it manually through the Amplify console by following the prompts.



Step 2: Creating the Lambda Function
Create Lambda Function

Navigate to the AWS Lambda console and click "Create function." Choose "Author from scratch," name your function, and select a runtime (e.g., Python 3.12).

Write Lambda Code


Copy

Copy
 import json
 import math
 import logging

 logger = logging.getLogger()
 logger.setLevel(logging.INFO)

 def lambda_handler(event, context):
     logger.info(f"Received event: {event}")

     try:
         base = int(event['base'])
         exponent = int(event['exponent'])
         mathResult = math.pow(base, exponent)

         response = {
             'statusCode': 200,
             'body': json.dumps('Your result is ' + str(mathResult))
         }
         logger.info(f"Response: {response}")
         return response
     except KeyError as e:
         logger.error(f"KeyError: {e}")
         return {
             'statusCode': 400,
             'body': json.dumps(f'Missing key: {str(e)}')
         }
     except ValueError as e:
         logger.error(f"ValueError: {e}")
         return {
             'statusCode': 400,
             'body': json.dumps(f'Invalid value: {str(e)}')
         }
Step 3: Setting Up DynamoDB
Create a DynamoDB Table

Go to the DynamoDB console and create a table named test-app-db with a primary key of ID (String). Remember to copy and store you ARN for the table as it will be used in IAM policies.



Add Permissions

Ensure your Lambda function can read and write to the DynamoDB table by configuring the IAM policies.



Step 4: Creating a REST API with API Gateway
Create API

In the API Gateway console, create a new REST API.

Set Up Method

Create a POST method for your API and link it to your Lambda function. Configure the method request to accept query string parameters (base and exponent).



Deploy API

Deploy the API to a stage. Note down the invoke URL, as it will be used in your front end code.

Step 5: Integrating Everything
Front end Code Integration

Update your front end code to make a request to the API Gateway endpoint:

Note: Change your API gateway accordingly from this code if you want to reuse


Copy

Copy

 <!DOCTYPE html>
 <html>
 <head>
     <meta charset="UTF-8">
     <title>To the Power of Math!</title>
     <!-- Styling for the client UI -->
     <style>
     h1 {
         color: #FFFFFF;
         font-family: system-ui;
         margin-left: 20px;
         }
     body {
         background-color: #222629;
         }
     label {
         color: #86C232;
         font-family: system-ui;
         font-size: 20px;
         margin-left: 20px;
         margin-top: 20px;
         }
      button {
         background-color: #86C232;
         border-color: #86C232;
         color: #FFFFFF;
         font-family: system-ui;
         font-size: 20px;
         font-weight: bold;
         margin-left: 30px;
         margin-top: 20px;
         width: 140px;
         }
      input {
         color: #222629;
         font-family: system-ui;
         font-size: 20px;
         margin-left: 10px;
         margin-top: 20px;
         width: 100px;
         }
     </style>
     <script>
         // callAPI function that takes the base and exponent numbers as parameters
         var callAPI = (base,exponent)=>{
             // instantiate a headers object
             var myHeaders = new Headers();
             // add content type header to object
             myHeaders.append("Content-Type", "application/json");
             // using built in JSON utility package turn object to string and store in a variable
             var raw = JSON.stringify({"base":base,"exponent":exponent});
             // create a JSON object with parameters for API call and store in a variable
             var requestOptions = {
                 method: 'POST',
                 headers: myHeaders,
                 body: raw,
                 redirect: 'follow'
             };
             // make API call with parameters and use promises to get response
             fetch("https://rpbsgbv8ul.execute-api.us-east-1.amazonaws.com/dev", requestOptions)
             .then(response => response.text())
             .then(result => alert(JSON.parse(result).body))
             .catch(error => console.log('error', error));
         }
     </script>
 </head>
 <body>
     <h1>TO THE POWER OF MATH!</h1>
     <form>
         <label>Base number:</label>
         <input type="text" id="base">
         <label>...to the power of:</label>
         <input type="text" id="exponent">
         <!-- set button onClick method to call function we defined passing input values as parameters -->
         <button type="button" onclick="callAPI(document.getElementById('base').value,document.getElementById('exponent').value)">CALCULATE</button>
     </form>
 </body>
 </html>
 index.html
 Displaying index.html.
Lambda function Integration

Note: Change your DB table name accordingly from this code if you want to reuse


Copy

Copy
 import json
 import math
 import logging
 import boto3
 from time import gmtime, strftime

 logger = logging.getLogger()
 logger.setLevel(logging.INFO)

 dynamodb = boto3.resource('dynamodb')
 table = dynamodb.Table('test-app-db')
 now = strftime("%a, %d %b %Y %H:%M:%S +0000", gmtime())

 def lambda_handler(event, context):
     logger.info(f"Received event: {event}")

     try:
         base = int(event['base'])
         exponent = int(event['exponent'])
         mathResult = math.pow(base, exponent)

         response = table.put_item(
         Item={
             'ID': str(mathResult),
             'LatestGreetingTime':now
             })

         response = {
             'statusCode': 200,
             'body': json.dumps('Your result is ' + str(mathResult))
         }
         logger.info(f"Response: {response}")
         return response
     except KeyError as e:
         logger.error(f"KeyError: {e}")
         return {
             'statusCode': 400,
             'body': json.dumps(f'Missing key: {str(e)}')
         }
     except ValueError as e:
         logger.error(f"ValueError: {e}")
         return {
             'statusCode': 400,
             'body': json.dumps(f'Invalid value: {str(e)}')
         }
Deploy the Front end Again

After integrating the front end with the back end, deploy the updated code using AWS Amplify. Follow the same steps as in Step 1 to manually upload and deploy the updated front end code.

Conclusion
This project showcases the power and flexibility of AWS Amplify, Lambda, DynamoDB, IAM, and API Gateway in building and deploying scalable web applications. Feel free to experiment with additional features and enhancements!

For more awesome projects about AWS and its services, follow this blog page, also consider following me on LinkedIn. Want to know more about me!! follow me on Instagram!!









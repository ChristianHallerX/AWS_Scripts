# Random User Data Streaming

This project shows how data can be streamed into AWS using AWS Data Streams.


Randomuser_Stream.png



1. A good service that simulates such data is randomuser.me The service can be customized a lot with an API, but the standard settings are more than enough to get started.
The request requires a specification how many users (results) are supposed to be delivered in a JSON file.
Until recently, the service allowed many calls in the free tier, but that has been throttled now.

https://randomuser.me/api/?exc=login&results=5

We can however request a large number of results at once and submit them one-by-one as a stream instead of making many small requests.


2. Create Kinesis Data Stream in AWS

Give it a name. Since our payload is very small (in kilobytes per put) it is enough to use a single shard. Provision.



3 Decide if you want to run the requests locally or spin up an EC2 Instance.

Since randomuser.me provides data, but does not submit it to Kinesis, we have to create our own producer that sends off of random user data.

The local (desktop) variant will require you to install all Python libraries and make use of AWS credentials. In EC2 we first have to install Python, upload the script, and set up IAM roles for the EC2.

Desktop script running in infinite loop: AWS_Kinesis_Producer_Script.ipynb

After a few minutes, Kinesis should show activity in the "Monitoring" tab.



4. Kinesis Analysics for subsetting/pre-processing using SQL query

Provision a new Kinesis Analytics service and select the Data Stream as input.

Input and output table names are set by AWS, they just have to be integrated in a query.

The query will subset only males 18 and older and provide their age, names, and coordinates.

Kinesis_Analytics_Query.sql

We need Firehose to deliver the data to S3

Set a Destination: Kinesis Firehose, promt to create a new Firehose service.



5. Kinesis Firehose delivery stream configuration

"Step 1: Name and Source" Choose a new Firehose name.

"Step 2: Process records" However, we need a lambda function that adds a "new line" at the end of each record.

This job can be done with a Lambda. Thus, enable "Transform source records with Lambda", then "Create New".



6. Transform record with Lambda function

Selct blueprint "General Lambda Processing" and add a name to function.

For role select "Create new role with basic Lambda permissions". Then create function.

On the next screen we modify the code. We add the "\n" new line char and code it in base64, then concatenate it with the record. That's all.

index.js

Next, increase timeout to 1 minute, just in case this may take longer to execute. Then save the modifications and close the Lambda tab.



7. Back in Kinesis Firehose

Where left off to create a new Lambda function, we can now select the created one.

"Step 3: Configure Settings" Select Destination: S3

Create new S3 bucket and name it.

Buffer Size: 1 MB (max. every 1 MB data is delivered to S3)

Buffer interval: 60 seconds (max. every 60 seconds Firehose delivers to S3)

Create New Role 'firehose delivery role' for S3 bucket access.

Finish Firehose by clicking "Create delivery stream"


8. Back to Kinesis Analytics

Under "Connect to Destination" select the Firehose Delivery Stream that was just created.

Select the in-applicationstream (SQL name) used in Kinesis Analytics, in this case DESTINATION_USER_DATA.

Create standard access permissions, save and continue.

Now everything should be set up, connected, and working.


9. Check S3 bucket for incoming data.

Wait until some json files were written.

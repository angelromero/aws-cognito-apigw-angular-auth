# aws-cognito-apigw-angular-auth

An AngularV4-based web app to demonstrate different API authentication options using Amazon Cognito and API Gateway with an AWS Lambda and Amazon DynamoDB backend that stores user details.


1. Upload the file "sam/lambda.zip" to a S3 bucket of choice and add the bucket details to the "sam/sam.yaml" SAM Template (Resources->CognitoDemoFunction->Properties->CodeUri). The bucket should be in the same region as all resources.

2. Package the template with the following command and execute the resulting 'aws cloudformation deploy' output using the AWS CLI:

```
aws cloudformation package --template-file sam.yaml --output-template-file sam-output.yaml --s3-bucket <bucket>
```

The next command should be something similar to:

```
aws cloudformation deploy --template-file /Users/username/GitHub/aws-cognito-apigw-angular-auth/sam/sam-output.yaml --stack-name CognitoAPIGWDemo --capabilities CAPABILITY_IAM
```

CloudFormation will automatically create and configure the following resources in your account:

* Cognito User Pool, including app client and group
* Cognito Identity Pool
* Cognito IAM Roles for User Pools group and Identity Pool Authentication Provider
* API Gateway API
* Lambda Function
* DynamoDB Table

3. Generate a Google API ID following the instructions on http://docs.aws.amazon.com/cognito/latest/developerguide/google.html 

4. Go to the CloudFormation console, select the stack created on item 2 and open the OUTPUTS tab. All resources we'll need will be there. Use the information to fill up the details under RESOURCE IDENTIFIERS of the file "src/aws.service.ts" including the region.

5. Go to the Cognito Console, select the Identity Pool created by CloudFormation and click on EDIT IDENTITY POOL. 

6. Go to the AUTENTICATION PROVIDERS section, select the tab GOOGLE+, click on the UNLOCK button and add the details on the Google API ID generated on step 4. Save the changes.

7. Click on EDIT IDENTITY POOL once more, go to the AUTENTICATION PROVIDERS section, select the COGNITO tab. Under AUTHENTICATED ROLE SELECTION select the option CHOSE ROLE FROM TOKEN and under ROLE RESOLUTION select DENY. Save the changes.

8. Create 2 users on your User Pool using a command similar to the AWS CLI command bellow for both of them then confirm the user in the Cognito User Pools console:

```
aws cognito-idp sign-up \
--client-id <<App Client ID>> \
--username jdoe \
--password P@ssw0rd \
--region <<region>> \
--user-attributes '[{"Name":"given_name","Value":"John"},{"Name":"family_name","Value":"Doe"},{"Name":"email","Value":"jdoe@myemail.com"},{"Name":"gender","Value":"Male"},{"Name":"phone_number","Value":"+61XXXXXXXXXX"}]'
```

9. Add one of the users to the group called "cip-group" in the Cognito User Pool

10. Execute the following commands from a terminal:

```
npm install -g @angular/cli
ng new aws-cognito-apigw-angular
cd aws-cognito-apigw-angular
aws apigateway get-sdk --rest-api-id <RestApiId from CloudFormation OUTPUTS> --stage-name demo --sdk-type javascript apigwsdk.zip
unzip apigwsdk.zip
```

(From Windows unzip skip the last command and unzip the SDK file manually)

11. Copy the folder "/src", the file "package.json" and the folder "apiGateway-js-sdk" from step 3 to the newly created folder "aws-cognito-apigw-angular" from the last step, overwriting the existing files

12. From the folder "aws-cognito-apigw-angular" execute the following commands in a terminal:

```
npm install 
npm start
```

13. Access http://localhost:4200/ in a browser then log in with the user above

14. Authentication should now work as:

* The Google authenticated user should only be able to access the API resource /google
* The Cognito User Pools user member of the group "cip-group" should only be able to access the API resource /cip
* The Cognito User Pools user without any group membership should only be able to access the API resource /cup


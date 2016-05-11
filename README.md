# class-booking
Additional Wordpress development on original Appointzilla Class Booking plugin.

## SMS Integration Configuration

1. You should have basic knowledge of AWS. Have an Amazon AWS account & be logged in to the console. Select the appropriate region for your needs.
2. Under Services, find SNS under Mobile Services.
3. Create Topic.  Enter a topic name (eg "<mysitename>-schedule-notifications") and display name.
4. Create one or more subscriptions.  Select the Protocol as "SMS" and endpoint as the cellphone #.
5. Confirm the subscription by text message on the phone.
6. Copy the Topic ARN.
7. Find Lambda under the main AWS Services list.
8. Click Create a Lambda function & skip blueprint.
9. Name the function something like "mySMSNotificationHandler" and select the latest NodeJS runtime.
10. Enter the code from below. Be sure to insert your **Topic ARN** in your code.

```
var AWS = require("aws-sdk");

exports.handler = function(event, context, callback) {
    var name = JSON.stringify(event.name, null);
    var email = JSON.stringify(event.email, null);
    var phone = JSON.stringify(event.phone, null);
    var note = JSON.stringify(event.note, null);
    var eventText = JSON.stringify(event, null, 2);
    var fullEventText = 'New scheduled drive request received from: ' + name +
        ' contact: ' + email + ' ' + phone + ' note: ' + note;
    console.log("Received event:", eventText);
    var sns = new AWS.SNS();
    var params = {
        Message: fullEventText,
        TopicArn: "<your Topic ARN here>"
    };
    sns.publish(params, function(err, data) {
        if (err) {
            console.log(err, err.stack); // an error occurred
            callback("error");
        } else {
            console.log(data);           // successful response
            callback(null, "OK");
        }
    });
};
```

11. Use the "index.handler" Handler
12. Under Role, select create new Basic Execution Role and assign the default oneClick_lambda_basic_execution role and click allow.
13. Under Advanced Settings, apply 128 MB memory cap and a 3 sec timeout. Select "No VPC"
14. After you're done with your Lambda function, you'll create an API Gateway to hit it from the Internet. Find API Gateway under the main services.
15. Create API and a Resouce, and link it to your existing Lambda function. Use a POST method. Deploy the API & test. The instructions are here: http://docs.aws.amazon.com/apigateway/latest/developerguide/getting-started.html
..* The Method Execution Integration Request requires a Body Mapping Template Content-Type application/json that is:
```
#set($inputRoot = $input.path('$'))
{
  "name" : "$inputRoot.name",
  "email" : "$inputRoot.email",
  "phone" : "$inputRoot.phone",
  "note" : "$inputRoot.note"
}```
..* The Method Execution Integration Response should have a blank Default Mapping as status 200 response and a second non-default 500 response with Lambda Error Regex as `error.*`
..* The Method Execution Method Response should have a 200 and 500 response configured with no headers and empty models.
..* Example Swagger Import file:
```{
  "swagger": "2.0",
  "info": {
    "version": "2016-04-12T00:43:28Z",
    "title": "<yourTitle>"
  },
  "host": "<host>",
  "basePath": "/<basePath>",
  "schemes": [
    "https"
  ],
  "paths": {
    "/sms": {
      "post": {
        "consumes": [
          "application/json"
        ],
        "produces": [
          "application/json"
        ],
        "responses": {
          "200": {
            "description": "200 response",
            "schema": {
              "$ref": "#/definitions/Empty"
            }
          },
          "500": {
            "description": "500 response",
            "schema": {
              "$ref": "#/definitions/Empty"
            }
          }
        },
        "x-amazon-apigateway-integration": {
          "responses": {
            "default": {
              "statusCode": "200"
            },
            "error.*": {
              "statusCode": "500"
            }
          },
          "passthroughBehavior": "when_no_match",
          "httpMethod": "POST",
          "uri": "<your URI>",
          "requestTemplates": {
            "application/json": "#set($inputRoot = $input.path('$'))\r\n{\r\n  \"name\" : \"$inputRoot.name\",\r\n  \"email\" : \"$inputRoot.email\",\r\n  \"phone\" : \"$inputRoot.phone\",\r\n  \"note\" : \"$inputRoot.note\"\r\n}"
          },
          "type": "aws"
        }
      }
    }
  },
  "definitions": {
    "Empty": {
      "type": "object"
    }
  }
}
```



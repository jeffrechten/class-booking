# class-booking
Additional Wordpress development on original Appointzilla Class Booking plugin.

=== Class Booking ===
Creators: a.ankit, FARAZFRANK
Contributor: jeffrechten
Tags: booking, calendar, booking calendar, class booking, coaching ,class booking calendar, wordpress calendar , course booking, course
Requires at least: 3.3+
Tested up to: 4.5.2
Stable tag: 0.5
License: GPLv2 or later
License URI: http://www.gnu.org/licenses/gpl-2.0.html

Easily Take And Manage Class Booking on your Wordpress Blog

== Description ==

Class Booking is a simple but effective plugin which enables you to take Class Bookings on your wordpress blog. If you are a coach / consultant/ Language Teacher etc, you can harness the power of Class Booking Plugin. Simply unzip and upload appointment-calendar directory to `/wp-content/plugins/` directory and activate the plugin. 

If you run Online course then this Plugin will work for you. 

Use shortcode [APCLASS] to insert calendar into any post or page. 


Thats it, you can now start taking Bookings on your wordpress site.  
No need to use third party  booking services, everything can be managed from inside the wordpress admin panel. 

If you face any problem in using the plugin pls ask in the [Forums](http://wordpress.org/support/plugin/class-booking)

Brought to you fine folks at [Appointzilla] (http://appointzilla.com)

= Features =

* Create Class g: Coaching Class/ COnsultation Class etc.
* Add/Edit/Manage Booking from Admin Interface.
* Insert Booking Calendar into any Post/Page using ShortCode [APCALASS].
* Get Email Notifications on Booking.
* Get Booking Confirmation Emails on Booking Approved/Cancelled by Admin.
* Text messaging notification integration via AWS SNS.




== Installation ==

This section describes how to install the plugin and get it working.

1. Unzip archive and upload the entire folder to the `/wp-content/plugins/` directory
2. Activate the plugin through the 'Plugins' menu in WordPress
3. Use ShortCode [APCALASS] to insert booking calendar into any page /post
4. Start taking Class Bookings


== Frequently Asked Questions ==
We are currently working on the documentation of Class Booking Lite

== Screenshots ==

1. Screenshot 1
2. Screenshot 2
3. Screenshot 3
4. Screenshot 4
5. Screenshot 5
6. Screenshot 6
7. Screenshot 7
8. Screenshot 8
9. Screenshot 9 
10. Screenshot 10
11. Screenshot 11
12. Screenshot 12
13. Screenshot 13
14. Screenshot 14


== Changelog ==

= 0.5 =
* Fixed bug that showed the wrong time on notification emails
* Fixed bug that allowed multiple people to sign up for one class when they each load the calendar before someone completes the process
* Fixed a bug in the JS that broke the plugin with WP 4.5.
* Added fature to allow SMS messaging notifications for admin via AWS SNS.

= 0.4 =
* Fixed Date Selection Issue
* Fixed Notification Issue

= 0.3 =
Fixed the entry update issue

= 0.2 =
Fixes the time display issue. We now hide the incorrect time.
= 0.1 =
This version provides basic functionality to take and manage Class Bookings on wordpress blog.


## SMS Integration Configuration Help

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
  * The Method Execution Integration Request requires a Body Mapping Template Content-Type application/json that is:
```
#set($inputRoot = $input.path('$'))
{
  "name" : "$inputRoot.name",
  "email" : "$inputRoot.email",
  "phone" : "$inputRoot.phone",
  "note" : "$inputRoot.note"
}
```
  * The Method Execution Integration Response should have a blank Default Mapping as status 200 response and a second non-default 500 response with Lambda Error Regex as `error.*`
  * The Method Execution Method Response should have a 200 and 500 response configured with no headers and empty models.
  * Example Swagger Import file:
```
{
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



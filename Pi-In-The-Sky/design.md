# Pi In The Sky

I have collected a lot of Raspberry Pis over time. We also have a
lot of strangers in our home at any given time. This project is to
allow the same sort of surveillance rigor inside the home as we do
outside... to match the [Ring][1] by our doors.

Having said that ... Wi-Fi cameras are not cheap. However, through
various discounts, I have created a nightvision capable wi-fi camera
with a Raspberry Pi Zero W for $25 (with various discounts).

The machine is a very capable device, hoever, I instantly noticed the
problem with wrangling the video streams into a single cohesive
experience. This is where the __Pi In The Sky__ project was conceived.

## Components

There are not many components to this service:

1. Website
1. API
1. Notifications / messages
1. Proxy server (to handle individual streams)
1. Embedded Pi software to communicate to API

## Infrastructure Phase 1

In order to save operating costs, the entire application will run on AWS
serverless technology. The entire worflow can be API driven with the
exception of the video stream. Unfortunately, the capabilities of the
Raspberry Pi + plus browser really break down in this area.
This is intentionally labeled _Phase 1_ to explain that the proxy server
will be my home machine deployable onto potential EC2 hosts in the future.

1. S3 - House video streams, HTML, JS
1. Lambda - Single function that handles all API communication
1. API Gateway - Exposes endpoints to Lambda function
1. SQS - Message passing between API to proxy server to initiate video streams
1. DynamoDB - Application DB
1. Home machine - proxy server that embedded Pi long pull for updates
1. AWS IoT - software updates, device administrations

IoT potentially makes sense as a general administration level add-on, but
not entirey sure how useful it would be from an application sense. More
investigation is required.

## Sign-up

Sign-up is a typical OAuth sign-in using Google. Google is our authoritative
user store.

![Sign-up Workflow][2]

__Legend__

1. User goes to http://myapp.com
1. JS application is rendered, prompting sign-in. Sign-in URL is generated from API.
1. OAuth workflow is initiated by redirecting to Google
1. User grants auth, and redirected to API
1. API checks for record, asks for code
1. User record is stored permanently, web sessions are generated.

[1]: https://ring.com/
[2]: Pi-In-The-Sky_Signup.png

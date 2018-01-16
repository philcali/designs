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

## Registration

Registration is interesting, since the physical machine is outside of my control.
As a vendable solution, I envision this being a package that people install on Pis
they or a custom OS distribution at most.

Registration starts on the Pi. The Pi prompts the user to complete the registration
using a URL provided, which initiates the entire OAuth handshake. Once the OAuth handshake
is completed, the device will be registered. A registered device will show up on the
homepage dashboard as a configurable device.

![Registration Workflow][3]

__Legend__

1. User installs embedded server on Pi w/camera
1. Pi prompts user for register device, starts OAuth workflow
1. API redirects user to Google
1. User authorizes applications, application displays registration code
1. Code is pasted in Pi console
1. Pi is registered through API, and is therefore connected to User account.

## Management

Managing the Pis is a very straight forward CRUD / monitoring UI for all of the
installed cameras. Some interesting configuration can live here:

- Notification
- Motion vector
- Time lapse recordings
- General device health
- Auto-updates
- Plugins

![Management][4]

__Legend__

1. Authenticated User interacts with website
1. Website communicates dynamic information to / from API
1. API funnels calls to Lambda, which interact with DynamoDB
1. Device record modifications are funneled to a SQS queue
1. Information is proxied to proxy service, where Pi are updated

## Motion Capture

The brains of the motion detection live on the Pi with the camera. The Pi itself
is really only doing two things at any given time:

1. Streaming video into application memory, and detecting motion vectors
1. Polling at a location on the Proxy server for changes in settings

Why doesn't the Pi poll against SQS or the API directly? Two reasons:

1. Communicating with AWS directly smells like a leaky abstraction to me.
There's no reason to expose the underlying infrastructure to installed clients.
2. API Gateway does not presently support long-polling.

Another case for IoT can be made here: a Pi can be registered as a _thing_,
and basically start a direct socket connection to an IoT stream. Once again:
I want to avoid a direct AWS dependency in the Pi software.

The proxy server is used again for live streaming video, as you will see in a later
diagram.

![Motion Capture][5]

__Legend__

1. Pi detects motion, and buffers 20 seconds of video
1. Video is uploaded to API using the binary content-type
1. Lambda function handling the upload writes to DynamoDB (motion data, timestamp, device info, etc)
1. Lambda writes video to S3
1. S3 put event is triggered upon successful upload.
1. Lambda handling put event, updates DynamoDB record status.

## Live Video

What is the point of having this setup without supporting live video!? The solution for
live video is incredibly awkward, and involves a long running middleman to ingest the
video stream from the Pi, and pump the output to a read socket in use by the browser. We
revive the Proxy server for this solitary function.

Why not use Kinesis Video Streams? KVS is an extremely practical application for
stream ingestion, but the use of such streams has to be justified financially and
supplied with an analytical function (ie: facial detection, motion detection, etc) The
second phase of the project could in theory replace the proxy server with an ELB over
Kubernetes, that act as video producers to Kinesis. Or perhaps the Pi itself could
produce video to the Kinesis stream and live video is split to the browser socket.

Digressing a bit: The function I want to achieve costs me only the energy to run personal
computer, but allows the ability to adopt sofisticated services down the road for
enhanced scalability and features.

![Live Streaming][6]

__Legend__

1. Authenticated user interacts with website on S3.
1. Communication is handled through the API.
1. A _Live Stream_ request is queued to SQS
1. The proxy server creates a _Live Stream_ session for the device, connecting a browser websocket for incoming data.
1. Long-polling Pis receive a message from the proxy server that a _Live Stream_ session has been, and begins sending video data to the proxy server.
1. Connected websocket connection in browser is receiving proxied video data in real time.


[1]: https://ring.com/
[2]: images/Pi-In-The-Sky_Signup.png
[3]: images/Pi-In-The-Sky_Registration.png
[4]: images/Pi-In-The-Sky_Management.png
[5]: images/Pi-In-The-Sky_Motion.png
[6]: images/Pi-In-The-Sky_Live.png

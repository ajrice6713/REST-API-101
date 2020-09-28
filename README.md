# REST API's 101
This repository intends to serve as a 101 overview of REST API's with a focus on the Bandwidth product API's to provide context and real-world examples.

## Table Of Contents
* [API 101](#api-101)
  * [What is an API](#so-what-is-an-api-anyways)
  * [Anatomy of an HTTP Request](#anatomy-of-an-http-request)
    * [Methods](#methods)
    * [URLs](#urls)
    * [Headers](#headers)
    * [Body](#body)
  * [Authentication](#authentication)
  * [RESTful Methods](#restful-methods)
  * [Request and Response in REST](#request-and-response-in-rest)
    * [2xx Responses](#2xx-responses)
    * [4xx Responses](#4xx-responses)
    * [5xx Responses](#5xx-responses)
  * [Synchronicity (Not the Police album)](#synchronicity-not-the-police-album)
    * [Synchronous](#synchronous)
    * [Asynchronous](#asynchronous)
  * [Webhooks](#webhooks)
  * [XML vs. JSON](#xml-vs-JSON)
    * [XML](#xml)
    * [JSON](#json)
* [The Bandwidth API's](#the-bandwidth-apis)
  * [Account Configuration](#account-configuration)
    * [Users](#users)
    * [Sub-Accounts (Sites)](#sub-accounts-sites)
    * [Locations (Sippeers)](#locations-sippeers)
    * [Applications](#applications)
  * [IRIS](#iris)
  * [Voice](#voice)
    * [BXML](#bxml)
  * [Messaging](#messaging)
  * [e911](#e911)
  * [Subscriptions](#subscriptions)
  * [Common User Mistakes](#common-user-mistakes)
  * [Bandwidth API FAQ's](#bandwidth-api-faqs)
    * [Rate Limits](#rate-limits)
* [Resources and Tools](#resources-and-tools)
* [Glossary](GLOSSARY.md)

## API 101

### So What is an API, Anyways?
Consider eating out at a restaurant - you're given a menu and the restaurant is equipped to prepare a certain subset of meals. How do you (the user) convey your order to the chef (backend code) that prepares the food? Thankfully, this restaurant comes equipped with a server (API)! The server, like an API, inspects your order (request) to make sure its something on the menu, and passes it off to the chef to process once they confirm that whats ordered is on the menu<sup>[1](https://www.mulesoft.com/resources/api/what-is-an-api)</sup>. An API, or Application Programming Interface, can be thought of as a set of rules and regulations that define interactions between programs. The API determines what requests can be made to a service, what those requests need to look like, and what the expected response will be to each request.

API's are everywhere, and we interact with them on a daily basis. Posting to social media, for example, flows through an API in the form of an HTTP (web) request. You, the user, fill in text and media fields in a user interface (UI) and click 'submit.' Once submitted, the UI makes an HTTP API request which validates the content in the text and media fields, among other things, before your new content is actually posted to the service. The Bandwidth Dashboard UI sits on top of the IRIS API - meaning that requests made in the Bandwidth Dashboard pass through the IRIS API, and we can use built-in browser tools to inspect these requests and see them being made in real-time!

### Anatomy of an HTTP Request
So now that we know what an API is, how do we use it? Well, it all starts with a request - an HTTP request to be exact. Think of this as the part where you tell the waiter exactly what it is you want. The request could be asking the waiter what the specials are (think GET `/specials`), telling them what you want to order (POST `/orders`), and telling them to change your order when you inevitably change your mind a few minutes later (PUT `/orders/myOrder`). To interact with an API, you would send an HTTP request that contains a method, a URL, and possibly other information like headers and a body.

#### Methods
We will discuss HTTP methods in more detail in a [later section](#restful-methods), but, at a high level, the method lets the API know what kind of request you are making. Like the restaurant example, you would make a `GET` request to the server to get more information about a menu item, a `POST` request to place your order, a `PUT` request to modify it, and a `DELETE` request to cancel it.

#### URLs
The URL in the request determines where the headers and body information is sent. In an API, URL's are generally comprised of 2 parts; a base URL and an an endpoint. Lets look at an IRIS URL for example:

`https://dashboard.bandwidth.com/api/accounts/{accountId}/sites/{siteId}/sippeers/{sippeerId}/tns`  

The base URL for IRIS is `https://dashboard.bandwidth.com/api/`, and the endpoint in this example is `accounts/{accountId}/sites/{siteId}/sippeers/{sippeerId}/tns`. The base URL is the location of the service we are accessing, and the endpoint is the function of that service we are looking to utilize. The method determines how we use that function.

#### Headers
Each HTTP request can include optional headers as well that can act in a plethora of ways and provide the API with information about the request. Standard format for headers is `headerName: value`.

Some of the most common headers include `Authorization`, `Content-Type`, and `Content-Length`. Many headers are set automatically by the service building the request, but in some instances they need to be added/modified for the request to work.

#### Body
Lastly, certain methods require a request body. The body contains the resource information we are looking to add or modify. The structure of the body varies between API's and endpoints, but a good API will validate the information in the body and make sure it falls in line with what is expected. If the body is malformed or contains something that the API doesn't expect, the user will see an error in the response to their HTTP request. A sample JSON request to send a text message using the Bandwidth API looks like this:

```JSON
{
    "to"            : ["+19198675309"],
    "from"          : "+19195551234",
    "text"          : "This is a test text messsage",
    "applicationId" : "1234ab56-789c-01de-2f34-56g7h8i901j2",
    "tag"           : "test message"
}
```

### Authentication
A major component of API requests is authentication. You wouldn't want someone using Facebook's API to make posts in your name, so to protect you, Facebook requires your username and password to be included in the request to change anything on your account. To verify the identity of the requester, some API's require some form of unique identification in the form of an `Authorization` header in the HTTP request.

Bandwidth uses basic authentication and requires both the username and password to be sent in the Authorization header in a specific format - `username:password` - which then needs to be encoded in base64. A complete HTTP Header would look like this: `Authorization: Basic dXNlcm5hbWU6cGFzc3N3b3Jk`. More information on Bandwidth's credentials requirements can be found [here](https://dev.bandwidth.com/guides/accountCredentials.html).

### RESTful Methods
As stated above, the method defined in the request determines the action taken at the endpoint. Bandwidth's API is what is known as a REST API (compared to a SOAP API, more information on the differences between the two can be found [here](https://smartbear.com/blog/test-and-monitor/soap-vs-rest-whats-the-difference/)).

There are four main methods utilized in Bandwidth's REST API: GET, PUT, POST, and DELETE.

| Method | Description                                   |
|--------|-----------------------------------------------|
| GET    | Get information about an existing resource(s) |
| POST   | Create a new resource                         |
| PUT    | Modify an existing resource                   |
| DELETE | Delete an existing resource                   |

There are other HTTP methods available in a REST API, but since they are not used within Bandwidth, we won't cover them in this guide. More detailed information can be found [here](https://assertible.com/blog/7-http-methods-every-web-developer-should-know-and-how-to-test-them).

### Request and Response in REST
Once your request has been constructed - it's time to send it! At a very high-level, this is done using the internet with some magic sprinkled in, and once the request has been received by the API you targeted, it will send back a response. The response, like your request, includes headers and a body, as well as a status code to easily distinguish between a successful and failed request. In general, you can expect to receive a body with information relevant to your request or the status code received.

It is important to note that just because your request was received, what you were requesting may not have been processed yet, and the successful response merely indicates that the API received your request and that it passed the necessary checks needed to be passed onto the backend code. This behavior is asynchronous and will be discussed in detail in a later section.

Here are some of the common status codes that Bandwidth returns.

##### 2xx Responses
A 2xx response generally indicates a successful request.

| Code | Description           | Explanation                                                                |
|------|-----------------------|----------------------------------------------------------------------------|
| 200  | OK                    | Generic 'OK' response - the request completed successfully                 |
| 201  | Created               | New resource was created                                                   |
| 202  | Accepted              | Request has been received but not yet acted upon                           |
| 204  | No Content            | There is no content (body) in the response - but the headers may be useful |

##### 4xx Responses
A 4xx response generally indicates an error on the user side

| Code | Description           | Explanation                                                                |
|------|-----------------------|----------------------------------------------------------------------------|
| 400  | Bad Request           | Something in the request was malformed, or there was a syntax issue        |
| 401  | Unauthorized          | The server was unable to authenticate the credentials provided             |
| 403  | Forbidden             | The credentials were authenticated, but this user does not have permission |
| 404  | Not Found             | The resource you are trying to access does not exist                       |
| 429  | Too Many Requests     | You are making too many requests to the service at one time (rate limit)   |

##### 5xx Responses
A 5xx response generally indicates an error on the server side

| Code | Description           | Explanation                                                                |
|------|-----------------------|----------------------------------------------------------------------------|
| 500  | Internal Server Error | The receiving server encountered an error it couldn't handle               |
| 503  | Service Unavailable   | The service you are trying to reach is unavailable                         |

An exhausted list of HTTP status codes can be found [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status).

### Synchronicity (Not the Police album)
Also a popular concept in teaching, API's can be synchronous or asynchronous. More specifically, *endpoints* can be synchronous or asynchronous, an API can contain any combination of synchronous or asynchronous functions. So what does this mean exactly?

##### Synchronous
A synchronous function is one that completes over the course of the request and response sequence. Let's look at an example - take `www.baseurl.com/users`, with a synchronous function a `POST` request to this API would create a user resource. The API response would most likely return a `201 Created` status with information in the body that pertains to the newly created `user` resource, like an ID and timestamp of when that user was created.

##### Asynchronous
An asynchronous function is one that completes *after* the request and response cycle has completed. Looking at the above example, we could still make a `POST` request to `www.baseurl.com/users`, but the response we get back will differ slightly, and the receipt of that response does not indicate that our new user resource has been created. A response to an asynchronous function might return a `200 OK` status indicating that the request was received and that the back end code is processing the request. In the response you would still receive a unique identifier for the new resource, but you may also see a status indicating your request is in a received or processing state.

To check the status of the newly created resource, you would need to make a `GET` request to the new resource ID, which, in a good API, would be located at `www.baseurl.com/users/{userId}`. The `GET` request should return any information about the resource, as well as the status of that resources creation.

The majority of functions in the Bandwidth API's are asynchronous.

### Webhooks
"But mom, I don't *want* to make a GET request every time I create a resource"

With webhooks - you don't have to! Think of it like a push notification; A webhook is essentially an HTTP request that an API or service sends to your server, not to be confused with the HTTP response the API sends back to a user request. Like a request, a webhook is sent to a specified URL using an HTTP method and contains headers and a body. In relation to asynchronous functions, API's allow users to set input URL's to receive webhooks when resources requested have been created.

Bandwidth utilizes webhooks in all of its API's in the form of messaging callbacks, voice callbacks, and subscriptions.

### XML vs. JSON
The request and response bodies can be formatted in a number of different ways - they can even include things like files. The language of the request body must be noted in the `Content-Type` header of the request. The two formats relevant to Bandwidth are XML and JSON.

##### XML
XML, or Extensible Markup Language, is a standard used to format data in both a human and machine readable way. XML is comprised of nested tags used to store and identify data in a flexible way. XML tags can be nested inside of other XML tags in a parent/child relationship. Tags can also contain certain attributes to help classify data. Tags require an open and close tag to indicate where a certain bit of data starts and stops, which looks like this:
```XML
<Document>API 101</Document>
```

With an attribute, the XML would look like this:
```XML
<Document format='PDF'>API 101</Document>
```

XML allows the creation of custom tags, and there are even custom versions of XML that tailor to specific needs - an example being Bandwidth's BXML language used by the voice API. The IRIS API is entirely XML based. Here are some examples:

######  Ordering a Phone Number in IRIS
```XML
<Order>
    <CustomerOrderId>123456789</CustomerOrderId>
    <Name>Test order name</Name>
    <ExistingTelephoneNumberOrderType>
        <TelephoneNumberList>
            <TelephoneNumber>9492366947</TelephoneNumber>
        </TelephoneNumberList>
    </ExistingTelephoneNumberOrderType>
    <SiteId>34692</SiteId>
    <PeerId>627343</PeerId>
</Order>
```
When tags are nested inside of other tags, they are called elements. The parent tag in the above example is `<Order>`, which makes `<Name>` an element of the `<Order>` object.

######  Responding to a call event with BXML
```XML  
<Response>
   <Gather gatherUrl="https://gather.url/nextBXML" firstDigitTimeout="10" terminatingDigits="#">
      <SpeakSentence voice="kate">Please press a digit.</SpeakSentence>
   </Gather>
</Response>
```

##### JSON
JSON, or JavaScript Object Notation, is another data interchange format that is easily readable by both humans and machines, and consists of a collection of name/value pairs. JSON is widely popular because it is easy to parse (read), and most programming languages natively support it, meaning less time spent coding for developers and less troubleshooting when it comes to issues with the data itself. The Bandwidth Messaging API is exclusively JSON, and the voice API uses JSON in conjunction with BXML. Lets look at some examples of JSON request bodies:

###### Sending a group MMS in the Messaging API
```JSON
{
    "to"            : ["+19198675309", "19195554321ß"],
    "from"          : "+19195551234",
    "text"          : "This is a test text messsage",
    "media"         : "https://www.example.com/media/{mediaId}",
    "applicationId" : "1234ab56-789c-01de-2f34-56g7h8i901j2",
    "tag"           : "test mms message"
}
```

###### Creating an Outbound Phone Call
```JSON
{
    "to"            : "+19198675309",
    "from"          : "+19195551234",
    "answerUrl"     : "https://www.example.com/answerUrl",
    "applicationId" : "1234ab56-789c-01de-2f34-56g7h8i901j2"
}
```
Like XML, JSON can contain nested name value pairs. The Bandwidth callbacks for messaging and voice both contain nested information in name tags.

###### Bandwidth Inbound Messaging Webhook
```JSON
{
    "type"        : "message-received",
    "time"        : "2016-09-14T18:20:16Z",
    "description" : "Incoming message received",
    "to"          : "+12345678902",
    "message"     : {
      "id"            : "14762070468292kw2fuqty55yp2b2",
      "time"          : "2016-09-14T18:20:16Z",
      "to"            : ["+12345678902"],
      "from"          : "+12345678901",
      "text"          : "Hey, check this out!",
      "applicationId" : "93de2206-9669-4e07-948d-329f4b722ee2",
      "media"         : [
        "https://messaging.bandwidth.com/api/v2/users/{accountId}/media/14762070468292kw2fuqty55yp2b2/0/bw.png"
        ],
      "owner"         : "+12345678902",
      "direction"     : "in",
      "segmentCount"  : 1
    }
  }
```
Note the value for `"message"` is another JSON object nested within the parent.

## The Bandwidth API's
Bandwidth uses a separate API for each of its services. There are similarities between the services, like the username/password you use to make requests as well as account and application ID's, but those relationships are all handled on the back end - each API/product vertical is it's own separate entity and acts independently of the others with very few exceptions (ex. Web-RTC utilizes the Voice API).

The Bandwidth API's allow our customer to automate number provisioning, sending and receiving text messages, making and receiving phone calls - essentially facilitating communication across multiple channels with minimal human intervention required.

Bandwidth's API's follows basic REST practices with the exception of the DASH EVS API, which allows for both REST and SOAP requests. This guide will not cover the EVS SOAP API.

### Account Configuration
The first step to successful API usage at bandwidth is a correctly configured account. An improperly setup account can lead to several common failures that we will discuss in a later section - ensuring proper account setup can mitigate the need for several common customer support issues.

Bandwidth follows an account -> Sub-Account -> location hierarchy. A single IRIS account can have multiple sub-accounts, and each sub account can have multiple locations. Parent IRIS accounts can have multiple applications as well, and each application can be a voice or messaging application. To access the account, a user is needed with a set username, password, and roles.

##### Users
Users can be UI (user-interface) only, API only, or BOTH. API only users do not require password resets, whereas the other two user types do require password resets. A guide on creating API users can be found [here](https://dev.bandwidth.com/guides/accountCredentials.html#top). Because the username and password need to be sent in the HTTP API requests, it is more beneficial to create an API only user and provide those credentials in your API calls, because there wont be a risk of credentials expiring and your automated API calls suddenly failing with `401 Unauthorized` errors due to an expired or incorrect password. Users can have different roles toggled on or off as well, and these roles determine which parts of the API a user can access. If a certain user role is disabled and the user tries to make an API call that falls under that role, they will see a `403 Forbidden` response from the API, indicating that even though the username/password combination is a valid one, they are unable to perform the action they are attempting.

##### Sub-Accounts (Sites)
Sub-Accounts exist for organizational purposes, and are referred to as Sites in the API. At least one sub account is required per account with a valid address for billing purposes.

##### Locations (Sip-Peers)
A Location, referred to as a SipPeer in the API, can best be thought of as a logical grouping of phone numbers. Locations can control things like SMS and MMS enablement, and any telephone numbers added to the location will inherit its settings by default. One location is required to utilize Bandwidth's service, and one default location is required per sub-account. When provisioning telephone numbers, if a location is not specified in the API request, the number will provision to the default location of that sub-account.

##### Applications
Bandwidth Applications are where users set the callback URL for webhooks. An application can either be designated as a voice or messaging application, and users can associate one or multiple locations to each application. It is important to note that a location can only be associated to 1 messaging application and 1 voice application - but that an application can have many locations associated to it. When a messaging/voice event happens on a number, Bandwidth checks the IRIS database to determine which location the number lives in and what application it is associated to, and it is there we find the callback URL to send the webhook for the event.

### IRIS
##### Overview
Base URL: `https://dashboard.bandwidth.com/api/`
Language: `XML`
Team Slack Channel: #irischat
[Developer Documentation](https://dev.bandwidth.com/numbers/about.html)
[API Reference Guide](https://dev.bandwidth.com/numbers/apiReference.html)

The IRIS API handles all account and number management within Bandwidth. Users can create/modify/delete sub-accounts and locations, provision new TN's, port in in outside numbers, and modify settings account-wide using this API, among other things. IRIS is Bandwidth's largest service and is used by a multitude of internal and external customers. The IRIS API is what powers the [Bandwidth Dashboard](dashboard.bandwidth.com). Any actions taken here can be replicated with an API call, meaning that anything you do within the Bandwidth Dashboard could be automated and done programatically.

### Voice
##### Overview
Base URL: `https://voice.bandwidth.com/api/v2/accounts/{accountId}`
Language: `JSON` and `BXML`
Team Slack Channel: #voiceapi-chat
[Developer Documentation](https://dev.bandwidth.com/voice/about.html)
[API Reference Guide](https://dev.bandwidth.com/voice/methods/about.html)

The Voice API allows customers to automate and control the receipt and creation of phone calls. Powered by the Bandwidth Network, the API and callbacks allow customers to create and control telephone calls to live numbers in real time. When an incoming call is received, Bandwidth sends a JSON webhook to the callback URL declared in the application and expects instructions, in the form of BXML verbs, letting Bandwidth know what to do next.

The easiest way to visualize the voice API is like a game of call and response - Bandwidth sends a 'call' in the form of `JSON` information and expects a response containing BXML, telling us what to do next with the phone call. Users build these instructions using different BXML verbs, and send them in their HTTP response.

##### BXML
There are a multitude of [available BXML](https://dev.bandwidth.com/voice/bxml/about.html) verbs that users can respond to webhooks with. We suggest taking a look at each, along with their descriptions and available parameters. Verbs can be stacked and will be executed in the order they are stacked in. For example, stacking a play audio verb before a record verb would allow you to execute something like saying "please record your message," and then recording a voicemail from a live person on the other end of the call.

The below example shows a webhook sent by Bandwidth containing digits pressed by the end user, which can be requested using a `<Gather>` verb:

```JSON
{
    "eventType"        : "gather",
    "accountId"        : "55555555",
    "applicationId"    : "7fc9698a-b04a-468b-9e8f-91238c0d0086",
    "startTime"        : "2019-06-20T15:54:22.234Z",
    "answerTime"          : "2019-06-20T15:54:25.432Z",
    "from"             : "+15551112222",
    "to"               : "+15553334444",
    "direction"        : "outbound",
    "callId"           : "c-95ac8d6e-1a31c52e-b38f-4198-93c1-51633ec68f8d",
    "callUrl"          : "https://voice.bandwidth.com/api/v2/accounts/55555555/calls/c-95ac8d6e-1a31c52e-b38f-4198-93c1-51633ec68f8d",
    "digits"           : "2",
    "terminatingDigit" : ""
}
```

In the callback, you can see the value for `digits` is 2. Let's say you asked your user to press 1 to be transferred to sales or 2 to be transferred to customer service, and you can see that the user pressed 2. Now, you can return more BXML to let Bandwidth know which number to transfer the call to. Examples of what the callbacks look like for different types of events can be found [here](https://dev.bandwidth.com/voice/bxml/callbacks/about.html).

### Messaging
##### Overview
Base URL: `https://messaging.bandwidth.com/api/v2/users/{accountId}`
Language: `JSON`
Team Slack Channel: #messaging-chat
[Developer Documentation](https://dev.bandwidth.com/messaging/about.html)
[API Reference Guide](https://dev.bandwidth.com/messaging/methods/about.html)

The messaging API allows users to send and receive SMS and MMS messages from computer to handset and vice versa. The API is callback driven, and sends a webhook for every message sent and received by a telephone number. Users can create text message blasts, respond to incoming messages from other telephone numbers, or create text message events to send when users interact with a service through an existing portal, like an app or website. When someone sends a text to a Bandwidth telephone number, this is the `JSON` generated and sent to the callback URL:

```JSON
[
  {
    "type"        : "message-received",
    "time"        : "2016-09-14T18:20:16Z",
    "description" : "Incoming message received",
    "to"          : "+12345678902",
    "message"     : {
      "id"            : "14762070468292kw2fuqty55yp2b2",
      "time"          : "2016-09-14T18:20:16Z",
      "to"            : ["+12345678902"],
      "from"          : "+12345678901",
      "text"          : "Hey, check this out!",
      "applicationId" : "93de2206-9669-4e07-948d-329f4b722ee2",
      "media"         : [
        "https://messaging.bandwidth.com/api/v2/users/{accountId}/media/14762070468292kw2fuqty55yp2b2/0/bw.png"
        ],
      "owner"         : "+12345678902",
      "direction"     : "in",
      "segmentCount"  : 1
    }
  }
]
```

### e911
##### Overview
Base URL (DASH): `https://service.dashcs.com/dash-api/xml/emergencyprovisioning/v1`
Base URL (IRIS): `https://dashboard.bandwidth.com/api/`
Language: `XML`
Team Slack Channel: #evs-general
[DASH API Guide](https://support.bandwidth.com/hc/en-us/articles/115006226067-911-Dashboard-API-Guide)

There are 2 API's within Bandwidth that allow users to add address information that will be sent to a Public Safety Answering Point (PSAP) when a 911 call is made from that number; the DASH API, and the IRIS API.

##### DASH
DASH was an existing API acquired by Bandwidth, allowing us to offer 911 provisioning as a service.

##### IRIS e911

### Subscriptions

### Common User Mistakes

### Bandwidth API FAQ's
##### Rate Limits
##### Static IP's

## Resources and Tools
##### Postman
##### Ngrok
##### RequestBin
##### Dev.Bandwidth

## Glossary

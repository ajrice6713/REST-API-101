# REST API's 101
This repository intends to serve as a 101 overview of REST API's with a focus on the Bandwidth product API's to provide context and real-world examples.

## Table Of Contents
* [API 101](#API101)
  * [What is an API](#WhatIsAnAPI)
  * [Anatomy of an HTTP Request](#HTTPAnatomy)
    * [Methods](#HTTPMethods)
    * [URLs](#HTTPURLs)
    * [Headers](#HTTPHeaders)
    * [Body](#HTTPBody)
  * [Authentication](#Auth)
  * [RESTful Methods](#Methods)
  * [Request and Response](#R&R)
    * [2xx](#2xx)
    * [4xx](#4xx)
    * [5xx](#5xx)
  * [Synchronicity (Not the Police album)](#Synchronicity)
    * [Synchronous](#Synchronous)
    * [Asynchronous](#Asynchronous)
  * [Webhooks](#Webhooks)
  * [XML vs. JSON](#XMLvJSON)
    * [XML](#XML)
    * [JSON](#JSON)
* [The Bandwidth API's](#BandwidthAPIs)
  * [Account Configuration]()
  * [IRIS]()
  * [Messaging]()
  * [Voice]()
  * [e911]()
  * [Subscriptions]()
  * [Common User Mistakes]()
  * [Bandwidth API FAQ's]()
    * [Rate Limits]()
* [Resources and Tools]()
* [Glossary]()

## API 101 {#API101}

### So What is an API, Anyways? {#WhatIsAnAPI}
An API, or Application Programming Interface, can be thought of as a set of rules and regulations that define interactions between programs. The API determines what requests can be made to a service, what those requests need to look like, and what the expected response will be to each request. [Mulesoft](https://www.mulesoft.com/resources/api/what-is-an-api) provides a great real-world example of an API. Consider eating out at a restaurant - you're given a menu and the restaurant is equipped to prepare a certain subset of meals. How do you (the user) convey your order to the chef (backend code) that prepares the food? Thankfully, this restaurant comes equipped with a server (API)! The server, like an API, inspects your order (request) to make sure its something on the menu, and passes it off to the chef to process once they confirm that whats ordered is on the menu.

API's are everywhere, and we interact with them on a daily basis. Posting to social media, for example, flows through an API in the form of an HTTP request. You, the user, fill in text and media fields in a user interface (UI) and click 'submit.' Once submitted, the UI makes an HTTP API request which validates the content in the text and media fields, among other things, before your new content is actually posted to the service. The Bandwidth Dashboard UI sits on top of the IRIS API - meaning that requests made in the Bandwidth Dashboard pass through the IRIS API, and we can use built-in browser tools to inspect these requests and see them being made in real-time!

### Anatomy of an HTTP Request {#HTTPAnatomy}
To interact with an API, you would send an HTTP request containing a method, URL, headers, and body.

#### Methods {#HTTPMethods}
We will discuss HTTP methods in more detail in a [later section](#Methods), but, at a high level, the method lets the API know what kind of request you are making. Like the restaurant example, you would make a `GET` request to the server to get more information about a menu item, a `POST` request to place your order, a `PUT` request to modify it, and a `DELETE` request to cancel it.

#### URLs {#HTTPURLs}
The URL in the request determines where the headers and body information is sent. In an API, URL's are generally comprised of 2 parts; a base URL and an an endpoint. Lets look at an IRIS URL for example:

`https://dashboard.bandwidth.com/api/accounts/{accountId}/sites/{siteId}/sippeers/{sippeerId}/tns`  

The base URL for IRIS is `https://dashboard.bandwidth.com/api/`, and the endpoint in this example is `accounts/{accountId}/sites/{siteId}/sippeers/{sippeerId}/tns`. The base URL is the location of the service we are accessing, and the endpoint is the function of that service we are looking to utilize. The method determines how we use that function.

#### Headers {#HTTPHeaders}
Each HTTP request can include optional headers as well that can act in a plethora of ways and provide the API with information about the request. Standard format for headers is `headerName: value`.

Some of the most common headers include `Authorization`, `Content-Type`, and `Content-Length`. Many headers are set automatically by the service building the request, but in some instances they need to be added/modified for the request to work.

#### Body {#HTTPBody}
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

### Authentication {#Auth}
A major component of API requests is authentication. You wouldn't want someone using Facebook's API to make posts in your name, so to protect you, Facebook requires your username and password to be included in the request to change anything on your account. To verify the identity of the requester, some API's require some form of unique identification in the form of an `Authorization` header in the HTTP request.

Bandwidth uses basic authentication and requires both the username and password to be sent in the Authorization header in a specific format - `username:password` - which then needs to be encoded in base64. A complete HTTP Header would look like this: `Authorization: Basic dXNlcm5hbWU6cGFzc3N3b3Jk`.

Online encoding/decoding [tools](https://www.base64encode.org/) are available for free to help encode and decode credentials. Additionally, many programming languages can perform this encoding natively.

### RESTful Methods {#Methods}
As stated above, the method defined in the request determines the action taken at the endpoint. Bandwidth's API is what is known as a REST API (compared to a SOAP API, more information on the differences between the two can be found [here](https://smartbear.com/blog/test-and-monitor/soap-vs-rest-whats-the-difference/)).

There are four main methods utilized in Bandwidth's REST API: GET, PUT, POST, and DELETE.

| Method | Description                                   |
|--------|-----------------------------------------------|
| GET    | Get information about an existing resource(s) |
| POST   | Create a new resource                         |
| PUT    | Modify an existing resource                   |
| DELETE | Delete an existing resource                   |

There are other HTTP methods available in a REST API, but since they are not used within Bandwidth, we won't cover them in this guide. More detailed information can be found [here](https://assertible.com/blog/7-http-methods-every-web-developer-should-know-and-how-to-test-them).

### Request and Response in REST {#R&R}
Once your request has been constructed - it's time to send it! At a very high-level, this is done using the internet with some magic sprinkled in, and once the request has been received by the API you targeted, it will send back a response. The response, like your request, includes headers and a body, as well as a status code to easily distinguish between a successful and failed request. In general, you can expect to receive a body with information relevant to your request or the status code received.

It is important to note that just because your request was received, what you were requesting may not have been processed yet, and the successful response merely indicates that the API received your request and that it passed the necessary checks needed to be passed onto the backend code. This behavior is asynchronous and will be discussed in detail in a later section.

Here are some of the popular error codes that Bandwidth returns.

##### 2xx Responses {#2xx}
A 2xx response generally indicates a successful request.

| Code | Description           | Explanation                                                                |
|------|-----------------------|----------------------------------------------------------------------------|
| 200  | OK                    | Generic 'OK' response - the request completed successfully                 |
| 201  | Created               | New resource was created                                                   |
| 202  | Accepted              | Request has been received but not yet acted upon                           |
| 204  | No Content            | There is no content (body) in the response - but the headers may be useful |

##### 4xx Responses {#4xx}
A 4xx response generally indicates an error on the user side

| Code | Description           | Explanation                                                                |
|------|-----------------------|----------------------------------------------------------------------------|
| 400  | Bad Request           | Something in the request was malformed, or there was a syntax issue        |
| 401  | Unauthorized          | The server was unable to authenticate the credentials provided             |
| 403  | Forbidden             | The credentials were authenticated, but this user does not have permission |
| 404  | Not Found             | The resource you are trying to access does not exist                       |
| 429  | Too Many Requests     | You are making too many requests to the service at one time (rate limit)   |

##### 5xx Responses {#5xx}
A 5xx response generally indicates an error on the server side

| Code | Description           | Explanation                                                                |
|------|-----------------------|----------------------------------------------------------------------------|
| 500  | Internal Server Error | The receiving server encountered an error it couldn't handle               |
| 503  | Service Unavailable   | The service you are trying to reach is unavailable                         |

An exhausted list of HTTP status codes can be found [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status).

### Synchronicity (Not the Police album) {#Synchronicity}
Also a popular concept in teaching, API's can be synchronous or asynchronous. More specifically, *endpoints* can be synchronous or asynchronous, an API can contain any combination of synchronous or asynchronous functions. So what does this mean exactly?

##### Synchronous {#Synchronous}
A synchronous function is one that completes over the course of the request and response sequence. Let's look at an example - take `www.baseurl.com/users`, with a synchronous function a `POST` request to this API would create a user resource. The API response would most likely return a `201 Created` status with information in the body that pertains to the newly created `user` resource, like an ID and timestamp of when that user was created.

##### Asynchronous {#Asynchronous}
An asynchronous function is one that completes *after* the request and response cycle has completed. Looking at the above example, we could still make a `POST` request to `www.baseurl.com/users`, but the response we get back will differ slightly, and the receipt of that response does not indicate that our new user resource has been created. A response to an asynchronous function might return a `200 OK` status indicating that the request was received and that the back end code is processing the request. In the response you would still receive a unique identifier for the new resource, but you may also see a status indicating your request is in a received or processing state.

To check the status of the newly created resource, you would need to make a `GET` request to the new resource ID, which, in a good API, would be located at `www.baseurl.com/users/{userId}`. The `GET` request should return any information about the resource, as well as the status of that resources creation.

The majority of functions in the Bandwidth API's are asynchronous.

### Webhooks {#Webhooks}
"But mom, I don't *want* to make a GET request every time I create a resource"

With webhooks - you don't have to! A webhook is essentially an HTTP request that an API or service sends to your server, not to be confused with the HTTP response the API sends back to a user request. Like a request, a webhook is sent to a specified URL using an HTTP mehtod and contains headers and a body. In relation to asynchronous functions, API's allow users to set input URL's to receive webhooks when resources requested have been created.

Bandwidth utilizes webhooks in all of its API's in the form of messaging callbacks, voice callbacks, and subscriptions.

### XML vs. JSON {#XMLvJSON}
The request and response bodies can be formatted in a number of different ways - they can even include things like files. The language of the request body must be noted in the `Content-Type` header of the request. The two formats relevant to Bandwidth are XML and JSON.

##### XML {#XML}

##### JSON {#JSON}

## The Bandwidth API's {#BandwidthAPIs}

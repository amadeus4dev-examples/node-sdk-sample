# Introduction
This tutorial is to get you started with [Amadeus For Developers' Node SDK](https://github.com/amadeus4dev/amadeus-node) in __under 10 minutes__. We will create a node app from scratch, based on the basic architecture discussed in the first section. 

*Please note that this is NOT supposed to be a 'complete project' in any sense, rather just a boilerplate that can be quickly developed from scratch in a few minutes.*

We will start by [understanding the architecture](#Architecture), followed by building the app from scratch in just 3 simple steps:
- [Initialing a node app](#Initializing the Node App)
- [Connecting the node app with Amadeus For Developers Node SDK](#Connecting with Amadeus For Developers using Node SDK)
- [Creating a basic UI to view results](#Creating Basic UI)

# Pre-Requisites
- `API credentials`: In order to use Amadeus For Developers Node SDK, you will need a pair of their API credentials from the [Amadeus For Developers portal](https://developers.amadeus.com). It is as simple as [registering for an account](https://developers.amadeus.com/register), and [creating an app](https://developers.amadeus.com/my-apps) on the portal.
- `Docker`: For simplicity, we shall be using a node-based docker container to avoid conflicting with any node version installed on your machine. If you don't have docker, you can follow these steps [based on your operating system](https://docs.docker.com/get-docker/). Once docker is installed, running the following command will provide you with just the runtime-environment you need.
   ```bash
   docker run -it \
    -v $PWD/node-app:/app \
    -w /app \
    -e AMADEUS_CLIENT_ID=$AMADEUS_CLIENT_ID \
    -e AMADEUS_CLIENT_SECRET=$AMADEUS_CLIENT_SECRET \
    -p 8000:8000 \
    node:14.1.0-alpine3.10 /bin/sh
   ```
   __NOTE__ that you need to pre-set your `API credentials` as environment variables before running the above command:
   ```bash
   export AMADEUS_CLIENT_ID=YOUR_API_KEY_HERE
   export AMADEUS_CLIENT_SECRET=YOUR_API_SECRET_HERE
   ```
   Also, __NOTE__ that the application is located in the `node-app` folder inside current directory and the application is running on port `8000`.

# Architecture
![architecture-image](https://github.com/amadeus4dev/node-sdk-sample/blob/master/blog-images/architecture-image.jpg?raw=true)
As depicted by the diagram above, it's going to be a pretty simple architecture. Our `app` would contain:
- `server-side` that is responsible for the connection with Amadeus APIs using the node SDK; and
- `client-side` that is responsible for displaying the content from Amadeus APIs.

The following sections shall discuss the implementation of this architecture to translate this architecture into code.

# Initializing the Node App
![step-1-app-init](https://github.com/amadeus4dev/node-sdk-sample/blob/master/blog-images/step-1-app-init.jpeg?raw=true)
The goal of this section is to create a basic node app (equivalent to a `Hello World!`). This simply requires running the following command in your console:
```bash
npm init
```
__NOTE__ that this requires the `npm` CLI tool installed in your local machine. 
*For the sake of a simplicity, please feel free to press `enter` for each of the questions asked by the `npm` CLI tool.*

# Connecting with Amadeus For Developers
![step-2-sdk-connect](https://github.com/amadeus4dev/node-sdk-sample/blob/master/blog-images/step-2-sdk-connect.jpeg?raw=true)
This section is where we will install the [Amadeus For Developers Node SDK](https://github.com/amadeus4dev/amadeus-node). 
Firstly, install the node SDK to your app by running the following `bash` command:
```bash
npm install amadeus --save
```
Next, you need to load the `amadeus` dependency in your `index.js` file:
```js
var Amadeus = require('amadeus');
```
Finally, initialize the SDK as follows:
```js
var amadeus = new Amadeus({
  clientId: process.env["AMADEUS_CLIENT_ID"],
  clientSecret: process.env["AMADEUS_CLIENT_SECRET"]
});
```
__NOTE__ that this code expects the API credentials to be pre-set in your environment variables.

At this point, you can test your connection adding the following code-snippet in `index.js` file:
```js
amadeus.shopping.flightOffersSearch.get({
    originLocationCode: 'BOS',
    destinationLocationCode: 'LON',
    departureDate: '2020-08-01',
    adults: '2'
  }).then(function (data) {
    console.log(JSON.stringify(data));
  }).catch(function (responseError) {
    console.log(JSON.stringify(responseError));
  });
```
Hopefully, you should have the appropriate `JSON` response in your console on running the above file using the following command in your console where `docker` container is running:
```bash
node index.js
```

## Create HTTP-based interface(s) for `client-side`
The `index.js` file, which is (part of) the `server-side`, is now connected with the Amadeus APIs via the Node SDK. However, it needs an interface to communicate with the `client-side` application. For this, modify the same file (`index.js`) as follows:
```js
var http = require('http');
var fs = require('fs');

var Amadeus = require('amadeus');
var amadeus = new Amadeus({
  clientId: process.env["AMADEUS_CLIENT_ID"],
  clientSecret: process.env["AMADEUS_CLIENT_SECRET"]
});

http.createServer(function (request, response) {
  if (request.url === "/app") {
    fs.readFile("html/index.html", function (err, data) {
      response.writeHead(200, {
        'Content-Type': 'text/html'
      });
      response.write(data);
      response.end();
    });
  } else if (request.url === "/flights") {
    amadeus.shopping.flightOffersSearch.get({
      originLocationCode: 'BOS',
      destinationLocationCode: 'LON',
      departureDate: '2020-08-01',
      adults: '2'
    }).then(function (data) {
      response.writeHead(200, {
        'Content-Type': 'application/json'
      });
      response.write(JSON.stringify(data));
      response.end();
    }).catch(function (responseError) {
      response.writeHead(responseError.code, {
        'Content-Type': 'application/json'
      });
      response.write(JSON.stringify(responseError));
      response.end();
    });
  } else {
    response.writeHead(404, {
      'Content-Type': 'text/html'
    });
    response.write('404 error');
    response.end();
  }
}).listen(8000);
```
This enables the following interfaces to the `server-side` that are accessible over `HTTP`:
- `/app`: To access the `client-side` files from `html/` folder. 
  *This folder & its files will be created in next section.*
- `/flights`: To access the `HTTP` interfaces for the `server-side` app. 
  *For simplicity, this interface only wraps the `amadeus.shopping.flightOffersSearch.get` API (with hard-coded params) of the node SDK.*

Any request which does not match either of the two formats mentioned above, shall return with a `404` error.

# Creating Basic UI
![step-3-basic-ui](https://github.com/amadeus4dev/node-sdk-sample/blob/master/blog-images/step-3-basic-ui.jpeg?raw=true)
Without going into much detail, the UI of this boilerplate is set to have a `button` that triggers an `AJAX request` to communicate with the node-server (developed in previous section) at its `/flights` interface to fetch results from Amadeus For Developers in real-time. Once the `AJAX request` completes, the resulting JSON is placed inside the `<div id="flights>"`. 

Here is how the HTML `<body>` looks like:
```html
<div class="container">
    <h2>Flight Options</h2>
    <div id="flights" class="list-group">
      <button>Click here to fetch flight options!</button>
    </div>
  </div>
```
And, here is the javascript responsible for communicating with `server-side` for flight results and displaying them in the `HTML page`:
```js
$(document).ready(function () {
    $("button").click(function () {
      $.ajax({
        url: "/flights",
        success: function (response) {
          $("#flights").html(generateHTML(response.result.data));
        }
      });
    });
  });
```
__NOTE__ that we have abstracted the `generateHTML()` function to simplify the above snippet. If you want to check it out, please do so by opening the `html/index.html` file. Also, __NOTE__ that we're using `bootstrap` and `jQuery` to add a bit of styling in the UI.

# Conclusion
Finally, this app can be run using the following command in the docker container's console:
```bash
node index.js
```
To test, open the browser and browse to the following `URL`: `http://localhost:8000/app`. 
*You might need to change the port number, if the very first command (to start docker container) was modified.*

Clicking the `Click here to fetch flight options!` button:
![basic-ui-button](https://github.com/amadeus4dev/node-sdk-sample/blob/master/blog-images/basic-ui-button.png?raw=true)
should fetch & display the flight results as follows:
![basic-ui-flights](https://github.com/amadeus4dev/node-sdk-sample/blob/master/blog-images/basic-ui-flights.png?raw=true)
__NOTE__ that the button-click is not `instantaneous` and doesn't give a `loader` or `in-process` indication. So please give it a few seconds to complete the operation.

To build further upon this app, you will have to (at least) make the following changes:
- improve the UI to make it more user-friendly & engaging, 
- pass input-parameters from the `client-side` to the `server-side` for fetching desired results instead of the `hard-coded` one, and/or
- update `server-side` to have more interfaces for different [Amadeus For Developers APIs](https://developers.amadeus.com/self-service/apis-docs).
Hope this tutorial was useful!

# Useful Links
- [Amadeus For Developers](https://developers.amadeus.com/)
- [Node SDK GitHub Repository](https://github.com/amadeus4dev/amadeus-node)
- [Node SDK on NPM Repository](https://www.npmjs.com/package/amadeus)
- [Video version for this tutorial](Link TBD)
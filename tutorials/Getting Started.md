---
layout: page
categories: quickstarts-javascript
title: Getting Started
permalink: /quickstarts/javascript/uc/Getting%20Started
---

# Getting Started

In this quickstart, we will help you dip your toes into Kandy before you dive in. This guide will help you get started with the Kandy Javascript SDK.

## Using Kandy.js

To begin, you will need to include the Kandy javascript library in your application. The Kandy.js library can be found here: [Kandy.js](https://cdn.jsdelivr.net/gh/Kandy-IO/kandy-uc-js-sdk-3.x@619/dist/kandy.js).

Kandy.js will expose a factory function to your page called `create`. This function is used to create an instance of the SDK, as well as to configure that instance.

```javascript 
// Instantiate the SDK.
import { create } from kandy
const kandy = create(configs);

// Use the Kandy API.
kandy.on( ... );
```

After you've created your instance of Kandy, you can begin playing around with it to learn its functionality and see how it fits in your application. The Kandy API reference documentation will help to explain the details of the features available.

## Configurations

An important part of Kandy configurations is the server information. You will need to provide the server information that is part of your Kandy package. This can be done by providing a configuration object to the Kandy factory as shown below.

```javascript 
// Instantiate the SDK.
import { create } from kandy
const kandy = create({
    // Required: Server connection configs.
    authentication: {
        subscription: {
            // Specify the connection information for REST requests.
        },
        websocket: {
            // Specify the connection information for websockets.
        }
    }
    // Other feature configs.
    ...
});
```

To learn more about configuring Kandy, please see the [Configuration Quickstart](Configurations).

## Further Reading

The best way to learn is usually by example and the best way to learn Kandy is by going through quickstarts. At the end of each quickstart, there is a link to view working examples in Codepen. These examples are great resources with which to experiment and start your own code base. [Visit our quickstarts to learn more.](../)

## Browser Support

| Browser |        Versions         |
| :-----: | :---------------------: |
| Chrome  | Latest 3 Major Versions |
| Firefox | Latest 3 Major Versions |
|   IE    |           11            |
| Safari  |  Latest Major Version   |


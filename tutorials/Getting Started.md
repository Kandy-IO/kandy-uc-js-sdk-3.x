---
layout: page
categories: quickstarts-javascript
title: Getting Started
permalink: /quickstarts/javascript/uc/Getting%20Started
---

# Getting Started

In this quickstart, we will help you dip your toes into Kandy before you dive in. This guide will help you get started with the Kandy Javascript SDK.

## Using Kandy.js

To begin, you will need to include the Kandy javascript library in your application. The Kandy.js library can be found here: [Kandy.js](https://cdn.jsdelivr.net/npm/@kandy-io/uc-sdk@78782/dist/kandy.js).

Kandy.js will expose a factory function to your page called `create`. This function is used to create an instance of the SDK, as well as to configure that instance.

```  javascript
// Instantiate the SDK.
import { create } from kandy
const kandy = create(configs);

// Use the Kandy API.
kandy.on( ... );
```

To learn more about configuring Kandy, please see the [Configuration Quickstart](Configurations).

## Further Reading

The best way to learn is usually by example and the best way to learn Kandy is by going through quickstarts. At the end of each quickstart, there is a link to view working examples in Codepen. These examples are great resources with which to experiment and start your own code base. [Visit our quickstarts to learn more.](../)

## Browser Support

| Browser |        Versions         |
|:-------:|:-----------------------:|
| Chrome  | Latest 3 Major Versions |
| Firefox | Latest 3 Major Versions |
|   IE    |           11            |
| Safari  |  Latest Major Version   |




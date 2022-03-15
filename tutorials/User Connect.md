---
layout: page
categories: quickstarts-javascript
title: User Connect
permalink: /quickstarts/javascript/uc/User%20Connect
---

# Connecting and Disconnecting

In this quickstart we will cover how to connect and disconnect to the Kandy Platform using Kandy.js. We will provide snippets of code below, which together will form a working demo application.

The first step with Kandy.js is always to initialize it. You will need to know the server information for the Kandy platform that you are using for initialization. Depending on your platform, the only required configuration is the server address, as the others have generic defaults.
However, for convenience, we'll use the default configuration which can be used as a starting point.

```javascript 
import { create } from kandy
const kandy = create({
    // Required: Server connection configs.
    authentication: {
        subscription: {
            server: ...,
            ...
        },
        websocket: {
            server: ...,
            ...
        }
    }
})
```

To learn more about initializing Kandy, see our [Configuration Quickstart](Configurations). **This example does not provide data for `authentication` but it is required.**

Since we're going to be making a working demo, we also need some HTML. The HTML for this demo is quite simple.

```html
<div>
  <fieldset>
    <legend>Login using your account credentials</legend>
    User Email: <input type="text" id="username" /> Password:<input type="password" id="password" />
    <input type="submit" id="login" value="login" onclick="login();" />
    <input disabled type="submit" id="logout" value="logout" onclick="logout();" />
    Current user: <span id="current-user">None.</span>
  </fieldset>
  <div id="messages"></div>
</div>
```

What we have is a simple div containing the connected state of our app, two buttons, and an element for logging messages to.

## Step 1: Connecting

To connect using Kandy, you will need two things:

1. A username. This is the full username of a user on your domain. (Example: your-user@your-domain.kandy.io)
1. A password. Don't worry, its safe with us.

With these two things, you can call the connect function on Kandy.

```javascript
function login () {
  const username = document.getElementById('username').value
  const password = document.getElementById('password').value

  kandy.connect({
    username: username,
    password: password
  })
}
```

## Step 2: Connection Events

The `kandy.connect()` function does not return a value. Instead, Kandy.js uses events to tell you when something has changed. You can find a list of the authentication events in the API documentation.

To subscribe to these events, you use `kandy.on()`. Here is the example for our demo app:

```javascript
kandy.on('auth:change', function () {
  const user = kandy.getUserInfo()

  document.getElementById('current-user').innerHTML = user.username || 'None.'
  document.getElementById('username').disabled = Boolean(user.username)
  document.getElementById('password').disabled = Boolean(user.username)
  document.getElementById('login').disabled = Boolean(user.username)
  document.getElementById('logout').disabled = !Boolean(user.username)
  log('Connection state changed.')
})
```

If something goes wrong when we try to connect (invalid credentials maybe), we want to know. Kandy.js has an `auth:error` event to support this.

```javascript
// Listen for authentication errors.
kandy.on('auth:error', function (params) {
  log('Connect error: ' + params.error.message + ' (' + params.error.code + ')')
})
```

In the above piece of code we subscribe an anonymous function to the `auth:change` event. Now, whenever Kandy fires off an `auth:change` event, that function will be called. Inside this function we call `kandy.getConnection()`. This function returns an object that looks like so:

```javascript 
{ isConnected: true, isPending: false, error: undefined }
```

To learn more about the response from this API checkout the API documentation for `getConnection`.

## Step 3: Disconnecting

To disconnect, you simply call disconnect.

```javascript
function logout () {
  kandy.disconnect()
}
```

Calling this function will trigger a change in the connection state, which in turn will trigger any listeners to the `auth:change` event. You can therefore use your `auth:change` listener to detect if the disconnect was successful.

In situations where the application is going to be used by another user and you want to protect their data privacy, the destroy function will tear down the SDK and render it unusable. Afterwards the SDK can be created again so no data is passed between users who shouldn't have access.

```javascript
function logout () {
  kandy.on('auth:change', params => {
    const connection = kandy.getConnection()
    if (connection.isConnected === false && connection.isPending === false) {
      // If user is not connected and an operation is not pending, then the user disconnected.
      kandy.destroy()
      document.getElementById('username').disabled = true
      document.getElementById('password').disabled = true
      document.getElementById('login').disabled = true
      document.getElementById('logout').disabled = true
      log('Kandy SDK has been uninitialized, reload the page to reset tutorial.')
    }
  })
  kandy.disconnect()
}
```

### Live Demo

Want to play around with this example for yourself? Feel free to edit this code on Codepen.

<form action="https://codepen.io/pen/define" method="POST" target="_blank" class="codepen-form"><input type="hidden" name="data" value=' {&quot;js&quot;:&quot;/**\n * Kandy.io Authentication Demo\n */\n\nconst { create } = Kandy\nconst kandy = create({\n    // Required: Server connection configs.\n    authentication: {\n        subscription: {\n            server: ...,\n            ...\n        },\n        websocket: {\n            server: ...,\n            ...\n        }\n    }\n})\n\nfunction login () {\n  const username = document.getElementById(&apos;username&apos;).value\n  const password = document.getElementById(&apos;password&apos;).value\n\n  kandy.connect({\n    username: username,\n    password: password\n  })\n}\n\nkandy.on(&apos;auth:change&apos;, function () {\n  const user = kandy.getUserInfo()\n\n  document.getElementById(&apos;current-user&apos;).innerHTML = user.username || &apos;None.&apos;\n  document.getElementById(&apos;username&apos;).disabled = Boolean(user.username)\n  document.getElementById(&apos;password&apos;).disabled = Boolean(user.username)\n  document.getElementById(&apos;login&apos;).disabled = Boolean(user.username)\n  document.getElementById(&apos;logout&apos;).disabled = !Boolean(user.username)\n  log(&apos;Connection state changed.&apos;)\n})\n\n// Listen for authentication errors.\nkandy.on(&apos;auth:error&apos;, function (params) {\n  log(&apos;Connect error: &apos; + params.error.message + &apos; (&apos; + params.error.code + &apos;)&apos;)\n})\n\nfunction logout () {\n  kandy.disconnect()\n}\n\nfunction logout () {\n  kandy.on(&apos;auth:change&apos;, params => {\n    const connection = kandy.getConnection()\n    if (connection.isConnected === false && connection.isPending === false) {\n      // If user is not connected and an operation is not pending, then the user disconnected.\n      kandy.destroy()\n      document.getElementById(&apos;username&apos;).disabled = true\n      document.getElementById(&apos;password&apos;).disabled = true\n      document.getElementById(&apos;login&apos;).disabled = true\n      document.getElementById(&apos;logout&apos;).disabled = true\n      log(&apos;Kandy SDK has been uninitialized, reload the page to reset tutorial.&apos;)\n    }\n  })\n  kandy.disconnect()\n}\n\n// Utility function for appending messages to the message div.\nfunction log (message) {\n  document.getElementById(&apos;messages&apos;).innerHTML += &apos;<div>&apos; + message + &apos;</div>&apos;\n}\n\n&quot;,&quot;html&quot;:&quot;<script src=\&quot;https://cdn.jsdelivr.net/gh/Kandy-IO/kandy-uc-js-sdk-3.x@850/dist/kandy.js\&quot;></script>\n<script src=\&quot;$DEFAULTCONFIGURL$\&quot;></script>\n\n<div>\n  <fieldset>\n    <legend>Login using your account credentials</legend>\n    User Email: <input type=\&quot;text\&quot; id=\&quot;username\&quot; /> Password:<input type=\&quot;password\&quot; id=\&quot;password\&quot; />\n    <input type=\&quot;submit\&quot; id=\&quot;login\&quot; value=\&quot;login\&quot; onclick=\&quot;login();\&quot; />\n    <input disabled type=\&quot;submit\&quot; id=\&quot;logout\&quot; value=\&quot;logout\&quot; onclick=\&quot;logout();\&quot; />\n    Current user: <span id=\&quot;current-user\&quot;>None.</span>\n  </fieldset>\n  <div id=\&quot;messages\&quot;></div>\n</div>\n\n&quot;,&quot;css&quot;:&quot;&quot;,&quot;title&quot;:&quot;Kandy.io Authentication Demo&quot;,&quot;editors&quot;:&quot;101&quot;,&quot;js_external&quot;:&quot;https://cdn.jsdelivr.net/gh/Kandy-IO/kandy-uc-js-sdk-3.x@850/dist/kandy.js&quot;} '><input type="image" src="./TryItOn-CodePen.png"></form>


---
layout: page
categories: quickstarts-javascript
title: Rich Messaging
permalink: /quickstarts/javascript/uc/Rich%20Messaging
---

# Rich Messaging

In this quickstart we will cover how to send rich messages. We will provide snippets of code below, which together will form a working demo application.

This quickstart assumes that you are familiar with chat messages. For a refresher on chat messages, please read through our [Messaging Quickstart](Messaging).

The first step with Kandy.js is always to initialize it. In this quickstart, the default config values will work just fine, so this step is simple.

```javascript 
// Setup Kandy.
import { create } from kandy
const kandy = create({
    authentication: {
        // Required: Server connection configs.
        ...
    }
})
```

To learn more about initializing Kandy, see our [Configuration Quickstart](Configurations). **This example does not provide data for `authentication` but it is required.**

#### HTML

Since we're going to be making a working demo, we also need some HTML. The HTML for this demo is quite simple.

First we have a div to show if we are connected or not.

```html
<div id="auth-state">Connected: false</div>
```

Next, we have a fieldset that contains all the actions we will perform for messaging. We have a button and input field to create conversations, as well as a button and input field to create and send rich messages.

```html
<fieldset>
  <legend>Conversations</legend>
  <input type="button" value="Create" onclick="createConvo();" />
  conversation with:
  <input type="text" id="convo-participant" />
  <br /><br />

  <input type="file" id="file-input" />
  <br /><br />

  <input type="button" value="Send" onclick="sendMessage();" />
  <br /><br />

  <input type="button" value="Get File URL" onclick="getFileUrl();" />
</fieldset>
```

Finally, we have a div to print messages to.

```html
<div id="messages"></div>
```

#### Connection

To send messages, we first must be connected. For this section we can reuse the code from the [Connection Quickstart](User%20Connect).

### Step 1: Creating a Conversation

Creating a conversation is covered in depth in the [Messaging Tutorial](Messaging). The relevant code is below.

```javascript
/*
 *  Basic Chat functionality.
 */

// We will only track one conversation in this demo.
var currentConvo

// Create a new conversation with another user.
function createConvo() {
  var participant = document.getElementById('convo-participant').value

  // Pass in the full username of a user to create a conversation with them.
  currentConvo = kandy.conversation.get(participant)

  log('Conversation created with: ' + participant)
}
```

### Step 2: Creating a Rich Message

You create rich messages much like you create basic messages. The only difference is that you provide a `part.type` of `file` and a `file` attribute that contains the file.

```javascript
// Create and send a message to the current conversation.
function sendMessage() {
  if (currentConvo) {
    var file = document.getElementById('file-input').files[0]

    // Create the message object, passing in the file for the message.
    var message = currentConvo.createMessage({ type: 'file', file: file })

    // Send the message!
    message.send()
  } else {
    log('No current conversation to send message to.')
  }
}
```

Usually a file is selected using an input field of type `file` (as you can see in the HTML section above).

Once the message is created, it can be sent via `message.send()`.

### Step 3: Retrieving the file

The URL for a rich message is much like a text value, IE it is a key value pair on a `part` of a `message`. You can see how the demo app retrieves the URL below. The demo works under the assumption that a single message has been sent by using the demo's send function.

```javascript
// Print out the URL for a Rich Message
function getFileUrl() {
  // Ensure that the convo exists.
  if (currentConvo) {
    // Grab the first message from the conversation.
    message = currentConvo.getMessages()[0]

    // Check the first message for a part containing a url.
    if (message) {
      // Grab the url from the part, and grab an access token from Kandy.
      url = message.parts[0].url
      accessToken = kandy.getUserInfo().token

      // Append the access token to the end of the url.
      // This ensures our request for the file is authenticated.
      url = url + accessToken

      // Print.
      log('File URL: ' + url)
    } else {
      log('There are no messages in this conversation yet. Send a Rich Message first!')
    }
  } else {
    log('There is no conversation yet. Create a convo and send a Rich Message first!')
  }
}
```

### Live Demo

Want to play around with this example for yourself? Feel free to edit this code on Codepen.

<form action="https://codepen.io/pen/define" method="POST" target="_blank" class="codepen-form"><input type="hidden" name="data" value=' {&quot;js&quot;:&quot;/**\n * Kandy.io Rich Messaging Demo\n */\n\n// Variables for connecting.\nvar username = &apos;UsernameHere&apos;\nvar password = &apos;PasswordHere&apos;\n\n// Setup Kandy.\nconst { create } = Kandy\nconst kandy = create({\n    authentication: {\n        // Required: Server connection configs.\n        ...\n    }\n})\n\n/*\n * Authentication functionality.\n */\n\n// Listen for changes to the auth state.\nkandy.on(&apos;auth:change&apos;, function() {\n  var isConnected = kandy.getConnection().isConnected\n  document.getElementById(&apos;auth-state&apos;).innerHTML = &apos;Connected: &apos; + isConnected\n  log(&apos;Connection state changed.&apos;)\n})\n\n// Listen for authentication errors.\nkandy.on(&apos;auth:error&apos;, function(params) {\n  log(&apos;Connect error: &apos; + params.error.message + &apos; (&apos; + params.error.code + &apos;)&apos;)\n})\n\n// Login on page load.\nkandy.connect({\n  username: username,\n  password: password\n})\n\n// Utility function for appending messages to the message div.\nfunction log(message) {\n  document.getElementById(&apos;messages&apos;).innerHTML += &apos;<div>&apos; + message + &apos;</div>&apos;\n}\n\n/*\n *  Basic Chat functionality.\n */\n\n// We will only track one conversation in this demo.\nvar currentConvo\n\n// Create a new conversation with another user.\nfunction createConvo() {\n  var participant = document.getElementById(&apos;convo-participant&apos;).value\n\n  // Pass in the full username of a user to create a conversation with them.\n  currentConvo = kandy.conversation.get(participant)\n\n  log(&apos;Conversation created with: &apos; + participant)\n}\n\n// Create and send a message to the current conversation.\nfunction sendMessage() {\n  if (currentConvo) {\n    var file = document.getElementById(&apos;file-input&apos;).files[0]\n\n    // Create the message object, passing in the file for the message.\n    var message = currentConvo.createMessage({ type: &apos;file&apos;, file: file })\n\n    // Send the message!\n    message.send()\n  } else {\n    log(&apos;No current conversation to send message to.&apos;)\n  }\n}\n\n// Print out the URL for a Rich Message\nfunction getFileUrl() {\n  // Ensure that the convo exists.\n  if (currentConvo) {\n    // Grab the first message from the conversation.\n    message = currentConvo.getMessages()[0]\n\n    // Check the first message for a part containing a url.\n    if (message) {\n      // Grab the url from the part, and grab an access token from Kandy.\n      url = message.parts[0].url\n      accessToken = kandy.getUserInfo().token\n\n      // Append the access token to the end of the url.\n      // This ensures our request for the file is authenticated.\n      url = url + accessToken\n\n      // Print.\n      log(&apos;File URL: &apos; + url)\n    } else {\n      log(&apos;There are no messages in this conversation yet. Send a Rich Message first!&apos;)\n    }\n  } else {\n    log(&apos;There is no conversation yet. Create a convo and send a Rich Message first!&apos;)\n  }\n}\n\n/*\n * Listen for new messages sent or received.\n * This event occurs when a new message is added to a conversation.\n */\nkandy.on(&apos;messages:change&apos;, function(params) {\n  log(&apos;New message in conversation with &apos; + params.conversationId)\n})\n\n/*\n * Listen for a change in the list of conversations.\n * In our case, it will occur when we receive a message from a user that\n * we do not have a conversation created with.\n */\nkandy.on(&apos;conversations:change&apos;, function(params) {\n  log(&apos;New conversation with &apos; + params.conversationId)\n\n  if (!currentConvo) {\n    currentConvo = kandy.conversation.get(params.conversationId)\n  }\n})\n\n&quot;,&quot;html&quot;:&quot;<div id=\&quot;auth-state\&quot;>Connected: false</div>\n\n<fieldset>\n  <legend>Conversations</legend>\n  <input type=\&quot;button\&quot; value=\&quot;Create\&quot; onclick=\&quot;createConvo();\&quot; />\n  conversation with:\n  <input type=\&quot;text\&quot; id=\&quot;convo-participant\&quot; />\n  <br /><br />\n\n  <input type=\&quot;file\&quot; id=\&quot;file-input\&quot; />\n  <br /><br />\n\n  <input type=\&quot;button\&quot; value=\&quot;Send\&quot; onclick=\&quot;sendMessage();\&quot; />\n  <br /><br />\n\n  <input type=\&quot;button\&quot; value=\&quot;Get File URL\&quot; onclick=\&quot;getFileUrl();\&quot; />\n</fieldset>\n\n<div id=\&quot;messages\&quot;></div>\n\n&quot;,&quot;css&quot;:&quot;&quot;,&quot;title&quot;:&quot;Kandy.io Rich Messaging Demo&quot;,&quot;editors&quot;:&quot;101&quot;,&quot;js_external&quot;:&quot;https://cdn.jsdelivr.net/gh/Kandy-IO/kandy-uc-js-sdk-3.x@432/dist/kandy.js&quot;} '><input type="image" src="./TryItOn-CodePen.png"></form>


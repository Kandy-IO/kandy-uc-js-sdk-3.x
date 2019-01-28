---
layout: page
categories: quickstarts-javascript
title: Rich Messaging
permalink: /quickstarts/javascript/cpaas/Rich%20Messaging
---

# Rich Messaging

In this quickstart we will cover how to send rich messages. We will provide snippets of code below, which together will form a working demo application.

This quickstart assumes that you are familiar with chat messages. For a refresher on chat messages, please read through our [Messaging Quickstart](index.html#Messaging).

The first step with Kandy.js is always to initialize it. In this quickstart, the default config values will work just fine, so this step is simple.

``` hidden javascript
// Variables for connecting.
var username = "UsernameHere";
var password = "PasswordHere";
```

``` exclude javascript
// Setup Kandy.
import { create } from kandy
const kandy = create({
    authentication: {
        // Required: Server connection configs.
        ...
    }
})
```

``` hidden javascript
// Setup Kandy.
const { create } = Kandy
const kandy = create({
    authentication: {
        // Required: Server connection configs.
        ...
    }
})
```

To learn more about initializing Kandy, see our [Configuration Quickstart](index.html#Configurations). __This example does not provide data for `authentication` but it is required.__

#### HTML

Since we're going to be making a working demo, we also need some HTML. The HTML for this demo is quite simple.

First we have a div to show if we are connected or not.

``` html
<div id='auth-state'>Connected: false</div>
```

Next, we have a fieldset that contains all the actions we will perform for messaging. We have a button and input field to create conversations, as well as a button and input field to create and send rich messages.

``` html
<fieldset>
  <legend>Conversations</legend>
  <input type='button' value='Create' onclick='createConvo();' />
  conversation with:
  <input type='text' id='convo-participant' />
  <br/><br/>

  <input type="file" id="file-input"/>
  <br/><br/>

  <input type='button' value='Send' onclick='sendMessage();' />
  <br/><br/>

  <input type='button' value='Get File URL' onclick='getFileUrl();' />

</fieldset>
```

Finally, we have a div to print messages to.

``` html
<div id="messages"> </div>
```

#### Connection

To send messages, we first must be connected. For this section we can reuse the code from the [Connection Quickstart](index.html#User%20Connect).

``` hidden javascript
/*
 * Authentication functionality.
 */

// Listen for changes to the auth state.
kandy.on('auth:change', function() {
  var isConnected = kandy.getConnection().isConnected;
  document.getElementById('auth-state').innerHTML = 'Connected: ' + isConnected;
  log('Connection state changed.');
});

// Listen for authentication errors.
kandy.on('auth:error', function(params) {
  log('Connect error: ' + params.error.message + ' (' + params.error.code + ')');
});

// Login on page load.
kandy.connect({
  username: username,
  password: password
});

// Utility function for appending messages to the message div.
function log(message) {
  document.getElementById('messages').innerHTML += '<div>' + message + '</div>';
}
```

### Step 1: Creating a Conversation

Creating a conversation is covered in depth in the [Messaging Tutorial](index.html#Messaging). The relevant code is below.

``` javascript
/*
 *  Basic Chat functionality.
 */

// We will only track one conversation in this demo.
var currentConvo;

// Create a new conversation with another user.
function createConvo() {
  var participant = document.getElementById('convo-participant').value;

  // Pass in the full username of a user to create a conversation with them.
  currentConvo = kandy.conversation.get(participant);

  log('Conversation created with: ' + participant);
}
```

### Step 2: Creating a Rich Message

You create rich messages much like you create basic messages. The only difference is that you provide a `part.type` of `file` and a `file` attribute that contains the file.

``` javascript
// Create and send a message to the current conversation.
function sendMessage() {
  if(currentConvo) {
    var file = document.getElementById("file-input").files[0];

    // Create the message object, passing in the file for the message.
    var message = currentConvo.createMessage({type: 'file', file: file});

    // Send the message!
    message.send();
  } else {
    log('No current conversation to send message to.');
  }
}
```

Usually a file is selected using an input field of type `file` (as you can see in the HTML section above).

Once the message is created, it can be sent via `message.send()`.

### Step 3: Retrieving the file

The URL for a rich message is much like a text value, IE it is a key value pair on a `part` of a `message`. You can see how the demo app retrieves the URL below. The demo works under the assumption that a single message has been sent by using the demo's send function.

``` javascript
// Print out the URL for a Rich Message
function getFileUrl() {
  // Ensure that the convo exists.
  if(currentConvo){
    // Grab the first message from the conversation.
    message = currentConvo.getMessages()[0];

    // Check the first message for a part containing a url.
    if(message){
      // Grab the url from the part, and grab an access token from Kandy.
      url = message.parts[0].url;
      accessToken = kandy.getUserInfo().token;

      // Append the access token to the end of the url.
      // This ensures our request for the file is authenticated.
      url = url + accessToken;

      // Print.
      log('File URL: ' + url);
    } else {
      log('There are no messages in this conversation yet. Send a Rich Message first!');
    }
  } else {
    log('There is no conversation yet. Create a convo and send a Rich Message first!');
  }
}
```

``` hidden javascript
/*
 * Listen for new messages sent or received.
 * This event occurs when a new message is added to a conversation.
 */
kandy.on('messages:change', function(params) {
  log('New message in conversation with ' + params.conversationId);
});

/*
 * Listen for a change in the list of conversations.
 * In our case, it will occur when we receive a message from a user that
 * we do not have a conversation created with.
 */
kandy.on('conversations:change', function(params) {
  log('New conversation with ' + params.conversationId);

  if(!currentConvo) {
    currentConvo = kandy.conversation.get(params.conversationId);
  }
});
```

### Live Demo

Want to play around with this example for yourself? Feel free to edit this code on Codepen.

``` codepen
{
	"title": "Kandy.io Rich Messaging Demo",
	"editors": "101",
	"js_external": "https://localhost:3000/kandy/kandy.cpaas.js"
}
```

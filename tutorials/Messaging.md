---
layout: page
categories: quickstarts-javascript
title: Messaging
permalink: /quickstarts/javascript/cpaas/Messaging
---

# Messaging

In this quickstart we will cover how to send and receive text based messages using Kandy.js. We will provide snippets of code below, which together will form a working demo application.

The first step with Kandy.js is always to initialize it. In this quickstart, the default config values will work just fine, so this step is simple.

``` hidden javascript
// Variables for connecting.
var username = "UsernameHere";
var password = "PasswordHere";
```

``` javascript exclude
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

Next, we have a fieldset that contains all the actions we will perform for messaging. We have a button and input field to create conversations, as well as a button and input field to create and send messages. As well, we have a radio button to select what type of conversation we want to create, an IM conversation, or an SMS conversation. *Note:* not all backends support SMS messaging.

``` html
<fieldset>
  <legend>Conversations</legend>
  <p>Step 1: Select what type of conversation you want to create.</p>
  <div>
    <input type="radio" id="convoType1"
     name="convo" value="im" checked="checked">
    <label for="convoType1">IM</label>

    <input type="radio" id="convoType2"
     name="convo" value="sms">
    <label for="convoType2">SMS</label>
  </div>
  <br/>
  Step 2: Enter their contact information (full user ID or ten digit phone number):
  <input type='text' id='convo-participant' />
  <br/><br/>

  Step 3: Create!
  <input type='button' value='Create' onclick='createConvo();' />
  <br/><hr>

  <input type='button' value='Send' onclick='sendMessage();' />
  message to conversation:
  <input type='text' value='Test message' id='message-text' />

</fieldset>
```

Below that is a fieldset to hold the incoming and outgoing conversation messages.

``` html
<fieldset>
  <legend>Messages</legend>
  <div id='convo-messages'></div>
</fieldset>
```

Finally, we have a div to display general Kandy messages (such as any errors that may occur).

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

## Step 1: Creating a Conversation

In Kandy.js, there is the concept of a `Conversation` as an object. A `Conversation` object keeps track of messaging state between the participants of that conversation, as well as information and utilities for the conversation itself. When you send or receive a message with another user, it is sent or received through the conversation with that user. To start messaging with a user, you need to create a Conversation with them first.

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

  log('Conversation created with: ' + participant)
}
```

A `Conversation` has a few functions on it, such as `getMessages()`. You can learn more about these functions [here](../docs#conversation). An important thing to note is that `conversation.get()` will create a conversation in the state of Kandy if it doesn't already exist. If the conversation does already exist, Kandy will simply return that object.

## Step 2: Creating and Sending a Message

From that `Conversation` object, you can create a `Message` object. A Message object represents the message being sent/received, which, for this quickstart, will be a basic text message. To send the message, you simply call `send()` on the `Message` object.

``` javascript
// Create and send a message to the current conversation.
function sendMessage() {
  if(currentConvo) {
    var text = document.getElementById('message-text').value;

    // Create the message object, passing in the text for the message.
    var message = currentConvo.createMessage(text);

    // Send the message!
    message.send();
  } else {
    log('No current conversation to send message to.');
  }
}
```

## Step 3: Messaging Events

There are a few messaging events we care about. We will go over two such events below.

### `messages:change`

One such event is `messages:change`. This event is fired whenever a message is added to a conversation that is present in Kandy.js's state (including outgoing messages). Any subscribers to this event will receive the participant for which there is a new message. You can read more about this event [here](../docs#messaging).

``` javascript
/*
 * Listen for new messages sent or received.
 * This event occurs when a new message is added to a conversation.
 */
kandy.on('messages:change', function(params) {
  log('New message in conversation with ' + params.conversationId);

  // If the message is in the current conversation, render it.
  if(currentConvo.destination === params.conversationId) {
    renderLatestMessage(currentConvo);
  }
});
```

### `conversations:change`

This event is fired whenever a new conversation is added to the conversation list in the Kandy store. One such example of this occurring is when Kandy.js receives a message from a conversation it does not yet have a record for. In this instance, Kandy.js will create a representation of the new convo in the store, and emit this event. Any subscribers to this event will receive a conversation ID. You can read more about this event [here](../docs#messaging).

``` javascript
/*
 * Listen for a change in the list of conversations.
 * In our case, it will occur when we receive a message from a user that
 * we do not have a conversation created with.
 */
kandy.on('conversations:change', function(params) {
  log('New conversation with ' + params.conversationId);

  // If we don't have a current conversation, assign the new one and render it.
  if(!currentConvo) {
    currentConvo = kandy.conversation.get(params.conversationId);
    renderLatestMessage(currentConvo);
  }
});
```

When our event listeners receive an event, meaning our conversation has a new message, we want to display that message to the user. In the demo, our listeners do this by calling a `renderLatestMessage` function, which adds the message to our interface, as can be seen below.

``` javascript
// Display the latest message in the provided conversation.
function renderLatestMessage(convo) {
  // Retrieve the latest message from the conversation.
  var messages = convo.getMessages();
  var message = messages[messages.length - 1];

  // Construct the text of the message.
  var text = message.sender + ': ' + message.parts[0].text;

  // Display the message.
  var convoDiv = document.getElementById('convo-messages');
  convoDiv.innerHTML += '<div>' + text + '</div>';
}
```

This function receives the conversation object as input. It then grabs the messages via `convo.getMessages()`, then grabs the last message. From this message it grabs `message.parts[0]`. Messages can have multiple parts, such as a text part and an image part. Finally it formats and prints the message.

### Live Demo

Want to play around with this example for yourself? Feel free to edit this code on Codepen.

``` codepen
{
	"title": "Kandy.io Basic Chat Demo",
	"editors": "101",
	"js_external": "https://localhost:3000/kandy/kandy.cpaas.js"
}
```

---
layout: page
categories: quickstarts-javascript
title: Messaging
permalink: /quickstarts/javascript/uc/Messaging
---

# Messaging

In this quickstart we will cover how to send and receive text based messages using Kandy.js. We will provide snippets of code below, which together will form a working demo application.

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

Next, we have a fieldset that contains all the actions we will perform for messaging. We have a button and input field to create conversations, as well as a button and input field to create and send messages. As well, we have a radio button to select what type of conversation we want to create, an IM conversation, or an SMS conversation. _Note:_ not all backends support SMS messaging.

```html
<fieldset>
  <legend>Conversations</legend>
  <p>Step 1: Select what type of conversation you want to create.</p>
  <div>
    <input type="radio" id="convoType1" name="convo" value="im" checked="checked" />
    <label for="convoType1">IM</label>

    <input type="radio" id="convoType2" name="convo" value="sms" />
    <label for="convoType2">SMS</label>
  </div>
  <br />
  Step 2: Enter their contact information (full user ID or ten digit phone number):
  <input type="text" id="convo-participant" />
  <br /><br />

  Step 3: Create!
  <input type="button" value="Create" onclick="createConvo();" />
  <br />
  <hr />

  <input type="button" value="Send" onclick="sendMessage();" />
  message to conversation:
  <input type="text" value="Test message" id="message-text" />
</fieldset>
```

Below that is a fieldset to hold the incoming and outgoing conversation messages.

```html
<fieldset>
  <legend>Messages</legend>
  <div id="convo-messages"></div>
</fieldset>
```

Finally, we have a div to display general Kandy messages (such as any errors that may occur).

```html
<div id="messages"></div>
```

#### Connection

To send messages, we first must be connected. For this section we can reuse the code from the [Connection Quickstart](User%20Connect).

## Step 1: Creating a Conversation

In Kandy.js, there is the concept of a `Conversation` as an object. A `Conversation` object keeps track of messaging state between the participants of that conversation, as well as information and utilities for the conversation itself. When you send or receive a message with another user, it is sent or received through the conversation with that user. To start messaging with a user, you need to create a Conversation with them first.

```javascript
/*
 *  Basic Chat functionality.
 */

// We will only track one conversation in this demo.
var currentConvo

// Create a new conversation with another user.
function createConvo () {
  var participant = document.getElementById('convo-participant').value

  // Pass in the full username of a user to create a conversation with them.
  currentConvo = kandy.conversation.get(participant)

  log('Conversation created with: ' + participant)
}
```

A `Conversation` has a few functions on it, such as `getMessages()`. You can learn more about these functions in the documentation for `conversation` API. An important thing to note is that `conversation.get()` will create a conversation in the state of Kandy if it doesn't already exist. If the conversation does already exist, Kandy will simply return that object.

## Step 2: Creating and Sending a Message

From that `Conversation` object, you can create a `Message` object. A Message object represents the message being sent/received, which, for this quickstart, will be a basic text message. To send the message, you simply call `send()` on the `Message` object.

```javascript
// Create and send a message to the current conversation.
function sendMessage () {
  if (currentConvo) {
    var text = document.getElementById('message-text').value

    // Create the message object, passing in the text for the message.
    var message = currentConvo.createMessage(text)

    // Send the message!
    message.send()
  } else {
    log('No current conversation to send message to.')
  }
}
```

## Step 3: Messaging Events

There are a few messaging events we care about. We will go over two such events below.

### `messages:change`

One such event is `messages:change`. This event is fired whenever a message is added to a conversation that is present in Kandy.js's state (including outgoing messages). Any subscribers to this event will receive the participant for which there is a new message. You can read more about this event in the API documentation.

```javascript
/*
 * Listen for new messages sent or received.
 * This event occurs when a new message is added to a conversation.
 */
kandy.on('messages:change', function (params) {
  log('New message in conversation with ' + params.conversationId)

  // If the message is in the current conversation, render it.
  if (currentConvo.destination === params.conversationId) {
    renderLatestMessage(currentConvo)
  }
})
```

### `conversations:change`

This event is fired whenever a new conversation is added to the conversation list in the Kandy store. One such example of this occurring is when Kandy.js receives a message from a conversation it does not yet have a record for. In this instance, Kandy.js will create a representation of the new convo in the store, and emit this event. Any subscribers to this event will receive a conversation ID. You can read more about this event in the API documentation.

```javascript
/*
 * Listen for a change in the list of conversations.
 * In our case, it will occur when we receive a message from a user that
 * we do not have a conversation created with.
 */
kandy.on('conversations:change', function (params) {
  log('New conversation with ' + params.conversationId)

  // If we don't have a current conversation, assign the new one and render it.
  if (!currentConvo) {
    currentConvo = kandy.conversation.get(params.conversationId)
    renderLatestMessage(currentConvo)
  }
})
```

When our event listeners receive an event, meaning our conversation has a new message, we want to display that message to the user. In the demo, our listeners do this by calling a `renderLatestMessage` function, which adds the message to our interface, as can be seen below.

```javascript
// Display the latest message in the provided conversation.
function renderLatestMessage (convo) {
  // Retrieve the latest message from the conversation.
  var messages = convo.getMessages()
  var message = messages[messages.length - 1]

  // Construct the text of the message.
  var text = message.sender + ': ' + message.parts[0].text

  // Display the message.
  var convoDiv = document.getElementById('convo-messages')
  convoDiv.innerHTML += '<div>' + text + '</div>'
}
```

This function receives the conversation object as input. It then grabs the messages via `convo.getMessages()`, then grabs the last message. From this message it grabs `message.parts[0]`. Messages can have multiple parts, such as a text part and an image part. Finally it formats and prints the message.

### Live Demo

Want to play around with this example for yourself? Feel free to edit this code on Codepen.

<form action="https://codepen.io/pen/define" method="POST" target="_blank" class="codepen-form"><input type="hidden" name="data" value=' {&quot;js&quot;:&quot;/**\n * Kandy.io Basic Chat Demo\n */\n\n// Variables for connecting.\nvar username = &apos;UsernameHere&apos;\nvar password = &apos;PasswordHere&apos;\n\n// Setup Kandy.\nconst { create } = Kandy\nconst kandy = create({\n    authentication: {\n        // Required: Server connection configs.\n        ...\n    }\n})\n\n/*\n * Authentication functionality.\n */\n\n// Listen for changes to the auth state.\nkandy.on(&apos;auth:change&apos;, function () {\n  var isConnected = kandy.getConnection().isConnected\n  document.getElementById(&apos;auth-state&apos;).innerHTML = &apos;Connected: &apos; + isConnected\n  log(&apos;Connection state changed.&apos;)\n})\n\n// Listen for authentication errors.\nkandy.on(&apos;auth:error&apos;, function (params) {\n  log(&apos;Connect error: &apos; + params.error.message + &apos; (&apos; + params.error.code + &apos;)&apos;)\n})\n\n// Login on page load.\nkandy.connect({\n  username: username,\n  password: password\n})\n\n// Utility function for appending messages to the message div.\nfunction log (message) {\n  document.getElementById(&apos;messages&apos;).innerHTML += &apos;<div>&apos; + message + &apos;</div>&apos;\n}\n\n/*\n *  Basic Chat functionality.\n */\n\n// We will only track one conversation in this demo.\nvar currentConvo\n\n// Create a new conversation with another user.\nfunction createConvo () {\n  var participant = document.getElementById(&apos;convo-participant&apos;).value\n\n  // Pass in the full username of a user to create a conversation with them.\n  currentConvo = kandy.conversation.get(participant)\n\n  log(&apos;Conversation created with: &apos; + participant)\n}\n\n// Create and send a message to the current conversation.\nfunction sendMessage () {\n  if (currentConvo) {\n    var text = document.getElementById(&apos;message-text&apos;).value\n\n    // Create the message object, passing in the text for the message.\n    var message = currentConvo.createMessage(text)\n\n    // Send the message!\n    message.send()\n  } else {\n    log(&apos;No current conversation to send message to.&apos;)\n  }\n}\n\n/*\n * Listen for new messages sent or received.\n * This event occurs when a new message is added to a conversation.\n */\nkandy.on(&apos;messages:change&apos;, function (params) {\n  log(&apos;New message in conversation with &apos; + params.conversationId)\n\n  // If the message is in the current conversation, render it.\n  if (currentConvo.destination === params.conversationId) {\n    renderLatestMessage(currentConvo)\n  }\n})\n\n/*\n * Listen for a change in the list of conversations.\n * In our case, it will occur when we receive a message from a user that\n * we do not have a conversation created with.\n */\nkandy.on(&apos;conversations:change&apos;, function (params) {\n  log(&apos;New conversation with &apos; + params.conversationId)\n\n  // If we don&apos;t have a current conversation, assign the new one and render it.\n  if (!currentConvo) {\n    currentConvo = kandy.conversation.get(params.conversationId)\n    renderLatestMessage(currentConvo)\n  }\n})\n\n// Display the latest message in the provided conversation.\nfunction renderLatestMessage (convo) {\n  // Retrieve the latest message from the conversation.\n  var messages = convo.getMessages()\n  var message = messages[messages.length - 1]\n\n  // Construct the text of the message.\n  var text = message.sender + &apos;: &apos; + message.parts[0].text\n\n  // Display the message.\n  var convoDiv = document.getElementById(&apos;convo-messages&apos;)\n  convoDiv.innerHTML += &apos;<div>&apos; + text + &apos;</div>&apos;\n}\n\n&quot;,&quot;html&quot;:&quot;<div id=\&quot;auth-state\&quot;>Connected: false</div>\n\n<fieldset>\n  <legend>Conversations</legend>\n  <p>Step 1: Select what type of conversation you want to create.</p>\n  <div>\n    <input type=\&quot;radio\&quot; id=\&quot;convoType1\&quot; name=\&quot;convo\&quot; value=\&quot;im\&quot; checked=\&quot;checked\&quot; />\n    <label for=\&quot;convoType1\&quot;>IM</label>\n\n    <input type=\&quot;radio\&quot; id=\&quot;convoType2\&quot; name=\&quot;convo\&quot; value=\&quot;sms\&quot; />\n    <label for=\&quot;convoType2\&quot;>SMS</label>\n  </div>\n  <br />\n  Step 2: Enter their contact information (full user ID or ten digit phone number):\n  <input type=\&quot;text\&quot; id=\&quot;convo-participant\&quot; />\n  <br /><br />\n\n  Step 3: Create!\n  <input type=\&quot;button\&quot; value=\&quot;Create\&quot; onclick=\&quot;createConvo();\&quot; />\n  <br />\n  <hr />\n\n  <input type=\&quot;button\&quot; value=\&quot;Send\&quot; onclick=\&quot;sendMessage();\&quot; />\n  message to conversation:\n  <input type=\&quot;text\&quot; value=\&quot;Test message\&quot; id=\&quot;message-text\&quot; />\n</fieldset>\n\n<fieldset>\n  <legend>Messages</legend>\n  <div id=\&quot;convo-messages\&quot;></div>\n</fieldset>\n\n<div id=\&quot;messages\&quot;></div>\n\n&quot;,&quot;css&quot;:&quot;&quot;,&quot;title&quot;:&quot;Kandy.io Basic Chat Demo&quot;,&quot;editors&quot;:&quot;101&quot;,&quot;js_external&quot;:&quot;https://cdn.jsdelivr.net/gh/Kandy-IO/kandy-uc-js-sdk-3.x@595/dist/kandy.js&quot;} '><input type="image" src="./TryItOn-CodePen.png"></form>


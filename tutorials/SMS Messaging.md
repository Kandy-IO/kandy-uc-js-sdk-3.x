---
layout: page
categories: quickstarts-javascript
title: SMS Messaging
permalink: /quickstarts/javascript/uc/SMS%20Messaging
---

# SMS Messaging

SMS Messaging in Kandy.js is almost identical to Messaging. There is only one difference, and that is when creating the conversation object.

## Creating a SMS Conversation

When creating or retrieving a SMS Conversation, you have to specify that it is of `type: 'sms'`, like so:

``` javascript
/*
 *  SMS Chat functionality.
 */

// We will only track one conversation in this demo.
var currentConvo;

// Create a new conversation with another cell device.
function createConvo() {
  var participant = document.getElementById('cell-device').value;

  // Pass in a cell number to create a conversation with that number.
  currentConvo = kandy.conversation.get(participant, {type: 'sms'});

  log('Conversation created with: ' + participant)
}
```

If this is specified, then the conversation and all messages created from it will have a type of `sms`.

The rest of SMS messaging is the same as [Messaging](Messaging). All API functions and events will work the same.


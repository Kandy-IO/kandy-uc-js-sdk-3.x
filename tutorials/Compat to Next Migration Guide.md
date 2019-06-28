---
layout: page
categories: quickstarts-javascript
title: Compat to Next Migration Guide
permalink: /quickstarts/javascript/uc/Compat%20to%20Next%20Migration%20Guide
---

# Migrating to Kandy v3.0

This document will go over some of the differences you will encounter when migrating an existing Kandy app using Kandy SDK v2.X.X to use Kandy v3.0. We use the word migrate, because this isn't just a simple update from v2 to v3 of Kandy. The API's specifically are very different. The flow to complete a task, such as send a message or make a call, is also very different.

With all these differences, we believe we have created an SDK that is easier to use and more expandable with future additions. We hope that after migrating, you agree.

## Plugins

Kandy v3.0 moved to a plugin based architecture. This means that different sections of the SDK (think messaging or calls) are separated into plugins within the SDK. Some of these plugins are part of the "base" SDK (such as configuration) meaning they are required to be included no matter what. Other plugins are optional, such as calls or messaging.

## Instantiating Kandy v3.0

When using Kandy v3.0, you have to create your own instance of Kandy to use. It is at this instantiation point that you would configure any parts of Kandy that need configuration.

```  javascript
import { create } from kandy
var kandy = create(configuration)
```

This functionality gives you a lot of power to do things that Kandy v2 was not capable of, such as have multiple instances of Kandy on the same page. Since these instances are totally separate (with there own states), they can even interact with each other, via things like messaging or calls.
``` exclude javascript
import { create } from kandy

// Create two individual instances of kandy with different configurations
var config1 = {...}
var config2 = {...}
kandyInstance1 = create(config1)
kandyInstance2 = create(config2)
```

## Configuration

In Kandy v3.0, configuration options are passed into Kandy when you are creating an instance. These config values will only apply to the returned instance of Kandy. If config values are provided, they are separated by the plugin that you are configuring, such as the `call` plugin or the `connectivity` plugin.

```  javascript
import { create } from kandy
var kandy = create({
    logs: {
        logLevel: 'error'
        // Other logs configuration
    },
    call: {
        callDefaults: {...},
        webrtcdtls: true,
        // Other call configuration
    },
})
```

You can learn more about the possible config values for different plugins [here](Configurations).

## State

One of the biggest changes from Kandy v2 to Kandy v3.0 is the introduction of a state (or 'store') inside the Kandy SDK. This store keeps track of information that the user may need (think messages from an ongoing conversation, or contacts in the addressbook) as well as information that the SDK needs to keep track of (think config values). Many functions performed through the SDK will interact with the store in some way.

``` javascript
// First create a conversation with a user.
var conversation = kandy.conversation.get(userId)

// Fetch functions retrieve information from the server.
// Grab the most recent 5 messages for this convo, placing them in the store.
conversation.fetchMessages(5)

// Retrieve the messages associated with this conversation from the store.
var messages = conversation.getMessages()

```

## Events

In Kandy v3.0, most information will be conveyed via events. There are many advantages to events over simple function returns or callbacks. The biggest plus being that it allows the application to subscribe to a stream of potential changes, as opposed to the one time use of promises or callbacks.

Because of this event and subscription model, users of the SDK can also leverage popular reactive libraries (such as RxJS) and integrate observables into their application.

A good example of the event flow is `kandy.connect()`.

``` javascript
// Listen for any changes to our authentication state
kandy.on('auth:change', function(){
  console.log(kandy.getConnection())
})
// Listen for any authentication errors
kandy.on('auth:error', function(error){
  console.log(error)
})
// Attempt to connect
kandy.connect({username: 'bob', password: 'supersecret', domainApiKey: '...'})
```

The above code registers functions that will be triggered whenever Kandy emits an `auth:change` or `auth:error` event. Calling `kandy.connect()` will result in an immediate `auth:change` event, which signifies that it is attempting to connect (connection is pending). Upon completing the attempted connect Kandy will then emit either another `auth:change` event, or an `auth:error` event if something goes wrong.

In most cases, as is shown above for `connect()`, function calls to Kandy v3.0 will have no return value.

The reference documentation lists all of the events that can be emitted by different actions. Each feature section includes the list of events related to that feature.

## Calls

#### Config

In Kandy v3, call configuration defaults are set at the instantiation of the Kandy object. It is then possible to override these defaults on any future call for that instance of Kandy. To learn more about this configuration check out the [Voice and Video Calls Quickstart](Voice%20%26%20Video%20Calls) for a simple setup, and the [Configuration Docs](index.html#Configurations) for more in depth information.

As well, Events for calls have changed. Please see the [Call Events Docs](../../references/uc#calls) to learn more about what events you may encounter.

#### Call Object

When a call begins, you will get a call ID either from the return value of `kandy.call.make` or from a `call:receive` event. This ID can then be used to retrieve a call object, if so desired.

``` javascript
// Set listener for incoming calls.
kandy.on('call:receive', function(newCallId) {
    // Keep track of the callId.
    callId = newCallId;

    // Retrieve call information.
    call = kandy.call.getById(callId);
    log('Received incoming call from ' + call.from);
});

// Make a call, save the Id
callId = kandy.call.make(callee, {
    sendInitialVideo: withVideo,
    remoteVideoContainer: remoteContainer,
    localVideoContainer: localContainer
});

// Grab the call object associated with this Id
let call = kandy.call.getById(callId);

```

It is important to note that the `callId` returned by `kandy.call.getById()` is not the same as previous versions of the SDK. In v3.0 this `callId` is a unique `id` generated by the SDK and is only used to facilitate the tracking of a call by the application.

#### Events

You may have noticed above that the events look a little different for Kandy v3.0 calls. All events are entirely new, so events you may have expected to be emitted in Kandy v2.0 will no longer be emitted. In most cases, Kandy v2.0 events will have a corresponding v3.0 event, however not always.

Take a look at the [Calls Events doc](../../references/uc#calls) to check out what events exist.

#### State

Like most things in Kandy v3, calls are tracked by the internal store or state of the Kandy SDK. This offsets some of the responsibility of tracking states in an app onto the SDK, hopefully making it a little easier to develop using the SDK.

## Messaging

In Kandy v2, sending a message was treated as a single action, with either a success or failure callback being triggered. Different types of messages were totally separate functions, such as `sendImWithVideo()` or `sendImWithFile()`.

``` javascript
kandy.sendImWithFile(userId, file, success, failure, options)
// or
kandy.sendImWithImage(userId, image, success, failure, options)
```

In Kandy v3.0, messaging is handled in a much more organized way. Messages are associated with a conversation, and have `parts` associated with them to make sending rich messages of various types far easier. As well, there are no callback functions. Instead, there are a few events to listen for that will allow you to easily update your app when something occurs. Messaging related events can be found [here](../../references/uc#messaging).

``` javascript
// Create a conversation with a user.
let conversation = kandy.conversation.get(userId)
// Create a message to build upon and send to the conversation.
let message = conversation.createMessage({type: 'text', text: 'This is message text.'})
// Add some more parts to the message, in this case a file.
message.addPart({type: 'file', file: fileBlob})
// Send the message!
message.send()

kandy.on('conversations:change', ({destination}) => {
    // Grab the messages from the conversation.
    let messages = conversation.getMessages()
    // Render the messages.
    renderMessages(messages)
})
```

This new system also allows an application to leverage the store to hold messages on its behalf. This means at any point in an application's progress you can call `getMessages()`, get back all the messages that are in the store, and not have to track those messages in your application.

## Sessions / Co-Browsing

Sessions and Co-Browsing are not currently in Kandy v3.0. We may revisit these features at one point but for now there is no plan to add them.

## Address-book

In Kandy v2, address book has a few isolated functions, such as `retrieveDirectory()`, or `searchDirectory`. Both of these examples call the server and then return information to the user. This means no information was stored by Kandy and the end developer is responsible for storing any information retrieved.

``` javascript
// Kandy v2.0 Example
kandy.retrieveDirectory((users) => {
    // Store the users somewhere locally for the app to use...
})
```

In Kandy v3.0, contacts are first retrieved via a `refreshContacts()` function. Once this has been performed, the local state will be up to date with the servers list of contacts. From here, you can request individual user information using `getUser(<id>)`. This flow offsets the responsibility to store the contacts from the developer, to the SDK. This tends to be a much better developer experience.

``` javascript
// Kandy v3.0 Example

// Fetch and store contact information
kandy.contacts.refresh()

// Get information for a specific user from the local store.
kandy.user.get(userId)
```


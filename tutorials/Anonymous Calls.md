---
layout: page
categories: quickstarts-javascript
title: Anonymous Calls
permalink: /quickstarts/javascript/cpaas/Anonymous%20Calls
---

# Anonymous Calls

Anonymous Calls is a Kandy feature that enables a user to make a call without needing their own user account. It allows for scenarios such as a visitor to a website making a call to a support line.

This quickstart will outline the differences between a Voice and Video Call scenario and a Anonymous Call scenario. Since Anonymous calls are an extension of Voice and Video calls, it is expected that the reader understands [Voice and Video calls](Voice%20%26%20Video%20Calls) fully.

## Prerequisites

1. Anonymous call requires that the Kandy domain has the CallMe service enabled for their users as part of their configurations.
1. Anonymous call requires the Anonymous call version of the Kandy SDK.
1. Anonymous call requires a server component for the generation of secure tokens.

## Types of Anonymous Calls

There are two types of Anonymous Calls: Time-limited and Unrestricted.

#### Time-Limited Anonymous Call

A "time-limited" Anonymous call is the proper name for what "Anonymous Call" generally refers to. Anonymous call does not require the caller to have a user account, so instead of authentication, the user requires time-limited tokens to verify the call. It is recommended that these tokens are generated with a server component, which allows you to moderate the Anonymous calls being made. More about the tokens are explained below.

#### Unrestricted Anonymous Call

An "Unrestricted" Anonymous Call does not require any verification for the call. This allows for a much simpler flow, but is unsecure, since it is more difficult to moderate the calls being made.

## Making a Call

The only difference in making a call between regular Calls and Anonymous calls is the "make call" API itself. Other mid-call operation APIs, events, etc. are the same as regular Calls.

```
// regular call API.
kandy.call.make(callee, callOptions);

// anonymous call API.
kandy.call.makeAnonymous(callee, credentials, callOptions);
```

For a time-limited call, you provide the tokens and the "realm" (see below) as part of the API call. For an unrestricted call, you simply omit the credentials by providing a null parameter.

Both types of calls can also include an optional `from` property in the `callOptions`, to indicate the caller.

```
let callOptions = {
    // Same call options as a regular call.
    isVideoEnabled: true,
    remoteVideoContainer: ...,
    ...
    // A from property.
    from: 'debbie@anon.callMe'
};

// Time-limited call.
let credentials = {
    accountToken: 'abc123...',
    fromToken: 'def456...',
    toToken: 'ghi789...',
    realm: 'kandyRealm123'
};
kandy.call.makeAnonymous(callee, credentials, callOptions)

// Unrestricted call.
kandy.call.makeAnonymous(callee, {}, callOptions);
```

## Receiving a Call

It should be noted that Anonymous calls are meant as outgoing-only calls. The Anonymous Call version of the Kandy SDK does not support receiving calls.

## Generating Tokens

The tokens used for a Anonymous call act as verification that the call is valid, and that it should be allowed. There are two security features encoded in the tokens for this: a timestamp and a secret key.

Token generation is done in two steps:
1. Creating the token, then
2. Encrypting the token.

### Token Creation

A token itself is simply a string of constructed in a certain format. The formats for creating the tokens are as shown below:

##### Account Token

`userID;timestamp`, eg. `user1@kandy.io;1234567890`

The user used for this token is the user that is configured as the "anonymous" user; the account is pre-setup for the Anonymous service. This user is considered the callee of the call.

##### From & To Tokens

`sip:userID;timestamp`, eg. `sip:user1@kandy.io;1234567890`

The from and to tokens are used to identify the users that the call is between. For the from token, the userID should be the same user as was used for the account token. For the to token, the userID should be the user receiving the call.

All three tokens are time-limited, meaning they will expire after a set amount of time. The expiry period is configured as a setting on the Kandy domain that the users belong to.

### Token Encryption

The tokens then need to be encrypted using a secret key.

#### Key / Realm

As a pre-requisite of using time-limited calls, Kandy server configuration will be needed to create a "realm" that corresponds to the Kandy domain that the users belong to. This realm defines the expiry period for the tokens and the key that should be used to encrypt and decrypt the tokens. When the application provides the tokens as part of the `call.makeAnonymous` API, it will also need to provide the name of this realm (see the `credentials` object in the 'Making a Call' section).

#### Algorithm

The algorithm that should be used to encrypt the token is the AES/ECB/PKCS5Padding cipher. The key used for it, which is configured as part of the realm, should be 128 bits (16 characters).




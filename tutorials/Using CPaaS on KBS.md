---
layout: page
categories: quickstarts-javascript
title: Using CPaaS on KBS
permalink: /quickstarts/javascript/cpaas/Using%20CPaaS%20on%20KBS
---

# Using CPaaS on KBS

To use the CPaaS on Kandy Business Solutions, you'll need to ensure you do the following.

## Use the CPaaS SDK

Note that there are different SDKs for different kinds of solutions. The SDK you'll need is the Kandy CPaaS SDK. You can find it in the [Releases page](../../releases).

## Configuration

You need to configure the SDK to point to the proper backend. Specifically, you need to put this information in the `authentication` configuration. Here's an example that contains the required configuration for KBS.

```  javascript
// Initialize an instance of Kandy.js.
import { create } from kandy
const kandy = create({
    authentication: {
        subscription: {
            server: 'kbs-na-kl.kandy.io'
        },
        websocket: {
            server: 'kbs-na-kl.kandy.io'
        }
    }
});
```

You can learn more about other configuration options in [Configuration Quickstart](Configurations)

## Login with your subscribers

The next step is to authenticate on the SDK. You can use your KBS subscribers's credentials to do that. You can learn more about authenticating on the SDK in the [User Connect Quickstart](User%20Connect)




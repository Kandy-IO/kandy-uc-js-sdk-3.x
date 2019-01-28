---
layout: page
categories: quickstarts-javascript
title: Codec Removal
permalink: /quickstarts/javascript/cpaas/Codec%20Removal
---

# Codec Removal

In some scenarios it's necessary to remove certain codecs being offered by the SDK to the remote party. While creating an SDP handler would allow a user to perform this type of manipulation, it is a non-trivial task that requires in-depth knowledge of WebRTC SDP.

To facilitate this common task, the SDK provides a codec removal handler that can be used for this purpose.

The SDP handlers are exposed on the entry point of the SDK. They need to be added to the list of SDP handlers via configuration on creation of a Kandy instance.

``` exclude javascript
import { create, sdpHandlers } from 'kandy';

const codecRemover = sdpHandlers.createCodecRemover(['VP8', 'VP9'])

const kandy = create({
    call: {
        sdpHandlers: [codecRemover]
    }
})
```

``` hidden javascript
const { create } = Kandy;

const codecRemover = create.sdpHandlers.createCodecRemover(['VP8', 'VP9'])

const kandy = create({
    call: {
        sdpHandlers: [codecRemover]
    }
})
```

In the above example we create a codec remover that will remove the VP8 and VP9 codecs. Then we create an instance of Kandy by adding the new SDP handler to the configuration.

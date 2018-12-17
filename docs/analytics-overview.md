# Ultra Analytics SDK Documentation

## Overview

This part of SDK helps developers send user generated events to Ultra Platform.

### Add the dependency

First step is to import sdk from fk-platform-sdk/analytics. A sample for using it is shown below.
##### For react native
```js
import {Travel} from "fk-platform-sdk/analytics";
```
##### For webview
Include the following script inside a `<script>` tag:
```js
https://img1a.flixcart.com/linchpin-web/fk-platform-sdk/analytics-min@1.0.3-analytics-sdk.js
```

### Create Event Object
After importing SDK, you can create event object of any type described in [this page](analytics-travel.md), passing in all the required arguments for that event.

### Sending the event
Finally push the event passing in the event object created as the argument. If the event is invalid then an error level log will be printed on console describing the issue with the event.
```js
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(event);
```
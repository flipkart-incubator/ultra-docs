# Ultra Analytics SDK Documentation

## Overview

This part of SDK helps developers send user generated events to Ultra Platform.

### Add the dependency

First step is to import sdk from fk-platform-sdk/analytics. To use analytics module you need to use the latest aplha version of SDK```"fk-platform-sdk": "1.0.3-analytics-sdk-alpha"```.

A sample for using it is shown below.
##### For react native
```js
import {<category_name>} from "fk-platform-sdk/analytics"; // for e.g <category_name> = Travel
```
##### For webview
Include the following script inside a `<script>` tag:
```js
https://img1a.flixcart.com/linchpin-web/fk-platform-sdk/fk-analytics-min@1.0.3-analytics-sdk-alpha.js
```

### Create Event Object
After importing SDK, you can create event object of any type described in [this page](analytics-travel.md), passing in all the required arguments for that event.

### Sending the event
Finally push the event passing in the event object created as the argument. 
```js
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(event);
```

!!!note
    If the event is invalid then an error log will be printed on console describing the issue with the event. This error will include the list of parameter which are invalid. 
    
    Sample error log message :

    ```
    Invalid event, ensure you resolve following issues: fromLocation is empty.
    ```

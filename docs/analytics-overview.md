# Ultra Analytics SDK Documentation

## Overview

This part of SDK helps developers send user generated events to Ultra Platform.

### 1. Add the dependency

First step is to add the dependency for the analytics library.

##### For react native or node
Add the [latest alpha version of SDK](https://www.npmjs.com/package/fk-platform-sdk/v/1.0.3-analytics-sdk-alpha) to your `package.json` :
```js
"fk-platform-sdk": "1.0.3-analytics-sdk-alpha"
```
Then import the analytics module wherever you want to push an event.
```js
import {<category_name>} from "fk-platform-sdk/analytics"; // for e.g <category_name> = Travel
```
##### For webview
Include the following script inside a `<script>` tag:
```js
https://img1a.flixcart.com/linchpin-web/fk-platform-sdk/fk-analytics-min@1.0.3-analytics-sdk-alpha.js
```

### 2. Create Event Object
After importing SDK, you can create event object of any type described in [this page](analytics-travel.md), passing in all the required arguments for that event.

### 3. Sending the event
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

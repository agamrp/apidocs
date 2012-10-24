Overview: API Version 1
========
The Crittercism API provides machine-readable, read/write access to Crittercism's Data platform. The API enables a variety of use cases by several distinct groups of users; this document records the requirements for the API, and serves as a specification for how it operates.

Users
=====
The API serves several groups of users:

* **App Developers** send data to the Crittercism API using the Crittercism client libraries. Developers want to understand app performance and may optionally use some of the advanced features of the API, including breadcrumbs, metadata, and usernames. Developers also sometimes upload symbol files to Crittercism.
* **Partners** register apps and/or developers. Representative partner users:
  * Moai: register new developers, create new applications
  * Appcelerator: register new developers, create new applications, real-time data lookup to Appcelerator for display on their site
  * LinkedIn: data retrieval

Design Principles
=================
* Mainline use cases should have client libraries (currently have good coverage for data ingestion, but poor data egress capabilities)
* API should use REST principles, specifically:
  * URLs as "resources", identifying things, not actions
  * Actions specified by HTTP verb (PUT, POST, GET, DELETE)
  * Universal, widely-used object IDs
  * Meaningful HTTP return codes (200, 201, 400, 401, 404, etc.)
* API should be versioned with thougtful end-of-life/deprecation contracts
* API shouldn't expose underlying storage mechanisms (e.g. memcached, redis, mongo)
* Authentication?
  * Out-of-band (HTTP Basic, OAuth); should reserve request/response for data
* API should use Rails/C-style underscored names ("app_id", "application", "my_long_variable_name") rather than camelCasedJavascriptVariableNames or PythonPascalCasedNames
* We'll need some kind of throttling/rate-limiting; get this right later.

Data Types
==========
The API uses the following data types:

Platform
--------
Platform represents a combination of operating system and physical hardware that runs an application instrumented with Crittercism. Platform has the following keys:

  * "os_name", a string representing the name of the operating system, one of
    * "wp" for Windows Phone
    * "android" for Android Java
    * "android-ndk" for Android Native Development Kit runtime
    * "ios"
    * "html5"
  * "os_version" is a numerical version identifier for the operating system, with type string, e.g. "8.0"

Stack Traces
------------
For Windows, the "stack_trace" field should be an array with one line of the trace per line of the array. Example:

`"stack_trace": [ "at CrittercismWP8TestApplication.MainPage.Button_Click_1(Object sender, RoutedEventArgs e)", " at System.Windows.Controls.Primitives.ButtonBase.OnClick()", ... ]`

Stack traces for other platforms TBD

Endpoints
=========
All endpoints accessible via SSL-encrypted HTTP at **api.crittercism.com** (port 443). All endpoints use URL-identified resources; POST and PUT requests use JSON-encoded entity bodies with the HTTP headers set to specify "application/json" MIME types. Clients should implement full Unicode handling, with support for common encoding types including UTF-8 and UTF-16.

Loads
-----
  * Show: Looks up an app load by id (todo)
    * URL: GET /v1/loads/{load\_id}
    * Returns: HTTP 200 with load on found, 404 if not found
  * Create: Record that an app load has occurred
    * URL: POST /v1/loads
    * Input: JSON message, example: `{
	"app_id": "4fdac56a456798fe",
	"app_state": {
          "battery_level": "60",
          "app_version": "2.0"
        },
	"platform": {
          "client": "wp8v1.0",
          "device_id": "34324-fdece-34tesf-343ce",
          "device_model": "Nokia Lumia 800",
          "os_name": "wp",
          "os_version": "4.0"
        }}`
      * Clients responsible for generating globally-unique device_id (use a guid). Clients should attempt to reuse these IDs across all applications installed on the same device, if possible. 
    * Returns: HTTP 201 on success, HTTP 400 on error. 
    * Side effects: Writes an app load event into persistent storage.

Crashes
-------
  * Show: Looks up a crash by id (todo)
    * URL: GET /v1/crashes/{crash\_id}
    * Returns: HTTP 200 with crash on found, 404 if not found
  * Create: Record that a crash has occurred
    * URL: POST /v1/crashes
    * Input: JSON message, example: `{
      "app_id": "4fdac56a456798fe",
      "app_state": {
        "battery_level": "60",
        "app_version": "2.0"
      },
      "breadcrumbs": {
        "current_session": [
          { "message": "User started playing the game", "timestamp": "2012-09-05T17:39:25+17:00" },
          { "message": "User continued playing the game", "timestamp": "2012-09-05T17:39:25+17:00" }
        ],
        "previous_session": [
          { "message": "User started playing the game", "timestamp": "2012-09-04T17:39:25+17:00" }
        ]
    },
    "crash": {
      "name": "System.NullArgumentException",
      "reason": "The runtime tried to dereference a null pointer",
      "stack_trace": "platform dependent stacktrace (see "stack trace" section)"
    },
    "platform": {
      "client": "iosv1.0",
      "device_id": "34324-fdece-34tesf-343ce",
      "device_model": "iPhone 3GS",
      "os_name": "ios",
      "os_version": "4.5"
    }}`
    * Returns: HTTP 201 with created object on success, HTTP 400 on error
    * Side effects: Writes a crash to persistent storage

Errors (Handled Exceptions)
---------------------------
  * Show: Looks up an error by id (todo)
    * URL: GET /v1/errors/{error\_id}
    * Returns: HTTP 200 with error on found, 404 if not found
  * Create: Record that an error has occurred
    * URL: POST /v1/errors
    * Input: JSON message, example: `{
      "app_id": "4e1c13d0ddf52049b4000012",
      "app_state": {
        "battery_level": "50",
        "app_version": "1.4"
      },
      "error": {
        "name": "Application.ReallyHideousError",
        "reason": "The user smashed the device in two",
        "stack_trace": "platform dependent stacktrace"
      },
      "platform": {
        "client": "iosv1.0",
        "device_id": "34324-fdece-34tesf-343ce",
        "device_model": "Samsumg Galaxy SIII",
        "os_name": "android",
        "os_version": "4.5"
      }}`
    * Returns: HTTP 201 with created object on success, HTTP 400 on error
    * Side effects: Writes the error to persistent storage

Data Types
==========
Documents valid values for each part of the API.

  * All times should be ISO-8601 formatted strings.


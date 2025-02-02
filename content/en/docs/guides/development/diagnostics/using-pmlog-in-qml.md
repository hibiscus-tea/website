---
title: "Using PmLog in QML"
date: 2019-03-15
weight: 60
toc: true
---

You can use the APIs described below to log information, including performance, in components or QML applications written using QML.

## APIs for a Component

The PmLogLib API is provided as a QML plugin on PmLog of the qml-webos-component. Any QML that imports PmLog will be able to call PmLogLib APIs as defined below. The PmLogLib API supports singleton version as well as instance version.

### PmLogLib API

<div class="table-container">
<table class="table is-bordered is-fullwidth">
<colgroup>
<col style="width: auto" />
<col style="width: auto" />
</colgroup>
<thead>
<tr class="header">
<th>API</th>
<th>Description and Syntax</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><strong>critical</strong></p></td>
<td><p>Logs critical errors.</p>
<p><strong>Syntax:</strong></p>
<pre>critical(msgid, kv_pairs, msg);</pre></td>
</tr>
<tr class="even">
<td><p><strong>error</strong></p></td>
<td><p>Logs errors.</p>
<p><strong>Syntax:</strong></p>
<pre>error(msgid, kv_pairs, msg);</pre></td>
</tr>
<tr class="odd">
<td><p><strong>warning</strong></p></td>
<td><p>Logs warning messages.</p>
<p><strong>Syntax:</strong></p>
<pre>warning(msgid, kv_pairs, msg);</pre></td>
</tr>
<tr class="even">
<td><p><strong>info</strong></p></td>
<td><p>Logs usage metrics.</p>
<p><strong>Syntax:</strong></p>
<pre>info(msgid, kv_pairs, msg);</pre></td>
</tr>
<tr class="odd">
<td><p><strong>debug</strong></p></td>
<td><p>Logs debug messages.</p>
<p><strong>Syntax:</strong></p>
<pre>debug(msg);</pre></td>
</tr>
</tbody>
</table>
</div>

### Parameters

The table below provides the description of the message components. The `msgid` is mandatory, and `kv_pairs` and `msg` are optional.

<div class="table-container">
<table class="table is-bordered is-fullwidth">
<colgroup>
<col style="width: auto" />
<col style="width: auto" />
<col style="width: auto" />
</colgroup>
<thead>
<tr class="header">
<th>Parameter</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><strong>msgid</strong></p></td>
<td><p>const char *</p></td>
<td><p>The msgid is an arbitrary short string (in most cases between 5 and 16 characters long) that uniquely identifies a log message within a component. The msgid cannot be a NULL or an empty string. It cannot contain a blank space (" ") or curly brackets ("{}") in it. </p>
<p>Every log statement, except for the debug and trace log statement, is expected to have a unique msgid to clearly differentiate the messages by function and use them for metrics analysis. For example, the number of native app launches per session, can be determined by counting the messages with msgid. The selection of the message IDs is left up to the developer. Typically it would be a short string in all capitals, for example, APPSTRT. It is not necessary that the meaning of the message is apparent from the msgid alone, but it is best to standardize on the names.</p>
<p>The msgids must be defined in a separate header file with comments to indicate their purpose. This will be used in data crunching for metrics. For example: </p>
<pre>#define MSGID_APP_START “APPSTRT” /** App started successfully */</pre>
<p><br />
It may be an additional burden on the developers to add message IDs to log statements. However, considering that our logs at or above INFO should only include significant information, we expect that programs will not need a large number of message IDs and the effort required will be kept minimal. </p>
{{< note >}}
The msgid is not required if the level parameter is set to debug. Since debug level logs are used by developers for code debugging and not used for metrics analysis, it is kept simpler for developers to log as a free-text at the debug level.
{{< /note >}}</td>
</tr>
<tr class="even">
<td><p><strong>kv_pairs</strong></p></td>
<td><p>const char *</p></td>
<td><p>This is an optional parameter which consists of a set of key-value pairs followed by a free text to provide additional information in the logs which can be used for analytics purpose.</p>
{{< note >}}
This parameter is optional, as long as the message ID itself is descriptive. For example, a message with an ID of `NETWORK_DOWN` or `READ_CONF_FAIL` is self-descriptive and does not need additional information passed as a parameter.
{{< /note >}}
</div>
<ul>
<li><p>Info and the higher level of messages should use key-value pairs to provide useful information about the log.</p></li>
<li><p>Keys should <strong>NOT</strong> contain a colon (":") in it.</p></li>
<li><p>To log a boolean message, use <code>PMLOGKFV("BOOLKEY","%s","true")</code>.</p>
<p>If double quotes are to be used in a value string, it should be stringified. For example:</p>
<pre>PMLOGKS("KEY", g_strescape("my \"quoted\" string value"));</pre></li>
<li><p>A free text - is the message part of the log. Since this is a format string, it can contain format specifiers, such as <code>%d</code> or <code>%s</code>. These format specifiers will be replaced with respective parameters in the variable parameter list. If a message does not have a free text to log, then it is specified as a blank space (" "). This text is to benefit from a human reading the logs. It will be discarded for any metrics gathering. Hence you must not use this field alone to log critical information. Here is an example of a free text: </p>
<pre>
PmLogMsg (my_context, Info, "APPLAUNCH", 3, PMLOGKFV("ID", "%d", app_info->id), PMLOGKS("NAME", app_info->name), PMLOGKS("STATUS", "launched"), "App launched successfully in %s", _func_);
</pre></li>
</ul></td>
</tr>
<tr class="odd">
<td><p><strong>msg</strong></p></td>
<td><p>const char *</p></td>
<td><p>The text to be logged.</p></td>
</tr>
</tbody>
</table>
</div>

## Example of PmLogLib API

The following is the sample code for how components written in QML can log information.

### Instance Version

``` javascript
import PmLog 1.0
PmLog {
     id: pmLog
     context: "com.webos.mycomponent"
}
pmLog.error("APPCRASH", {"APP_NAME":  "Facebook", "APP_ID": appId}, "Facebook app crashed, restart application");
pmLog.info("URLLOAD", {"URL": "http://webosose.org", "TIME": 200, "UNIT": "ms"});
pmLog.debug("BRWSRCLOSE");
```

Output:

``` plaintext
1970-01-01T00:01:05.714335Z [65] user.error DBGFRWK[1550]: com.webos.mycomponent APPCRASH {"APP_ID": "com.webos.app.facebook", "APP_NAME": "Facebook"} Facebook app crashed, restart application
1970-01-01T00:01:05.714572Z [65] user.info DBGFRWK[1550]: com.webos.mycomponent URLLOAD {"TIME": 200, "UNIT": "ms", "URL": "http://webosose.org"}
1970-01-01T00:01:05.714667Z [65] user.debug DBGFRWK[1550]: com.webos.mycomponent BRWSRCLOSE
```

### Singleton Version

``` javascript
import PmLog 1.0
PmLogger.context = "qml";

PmLogger.error("APPCRASH", {"APP_NAME": "Facebook", "APP_ID": appId}, "Facebook app crashed, restart application");
PmLogger.info("URLLOAD", {"URL": "http://webosose.org", "TIME": 200, "UNIT": "ms"});
PmLogger.debug("BRWSRCLOSE");
```

Output:

``` plaintext
1970-01-01T00:01:05.713362Z [65] user.error DBGFRWK[1550]: qml APPCRASH {"APP_ID": "com.webos.app.facebook", "APP_NAME": "Facebook"} Facebook app crashed, restart application
1970-01-01T00:01:05.713800Z [65] user.info DBGFRWK[1550]: qml URLLOAD {"TIME": 200, "UNIT": "ms", "URL": "http://webosose.org"}
1970-01-01T00:01:05.713971Z [65] user.debug DBGFRWK[1550]: qml BRWSRCLOSE
```

## PmLogLib Performance APIs for a Component

The following two types of APIs are provided for logging performance from a QML component.
Any QML that imports PerformanceLog will be able to call pmloglib performance APIs as defined below. The pmloglib performance API supports singleton version as well as instance version.

### PmLogLib Performace API for Measuring Time

These APIs measure performance time. They can measure time across processes. Both `time` and `timeEnd` log the time (in milliseconds) that was spent between the calls. Both take `msgid` and `kv_pairs` parameters that identify the measurement point. The `kv_pairs` and `msg` parameters are optional.

<div class="table-container">
<table class="table is-bordered is-fullwidth">
<colgroup>
<col style="width: auto" />
<col style="width: auto" />
</colgroup>
<thead>
<tr class="header">
<th>API</th>
<th>Description and Syntax</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><strong>time</strong></p></td>
<td><p>Starts a new timer.</p>
<p><strong>Syntax:</strong></p>
<pre>time(const char* msgid, const char* kv_pairs);</pre></td>
</tr>
<tr class="even">
<td><p><strong>timeEnd</strong></p></td>
<td><p>Stops a timer and prints the elapsed time.</p>
<p><strong>Syntax:</strong></p>
<pre>timeEnd(const char* msgid, const char* kv_pairs, const char* msg);</pre></td>
</tr>
</tbody>
</table>
</div>

### PmLogLib Performace API for Measuring FPS

These APIs measure frame per second. Both `fps` and `fpsEnd` log the frames per second (FPS) that was swapped between the calls. Both take `msgid` and `kv_pairs` parameters that identify the measurement point. The `kv_pairs` and `msg` parameters are optional.

<div class="table-container">
<table class="table is-bordered is-fullwidth">
<colgroup>
<col style="width: auto" />
<col style="width: auto" />
</colgroup>
<thead>
<tr class="header">
<th>API</th>
<th>Description and Syntax</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><strong>fps</strong></p></td>
<td><p>Starts a new FPS timer.</p>
<p><strong>Syntax:</strong></p>
<pre>fps(const char* msgid, const char* kv_pairs);</pre></td>
</tr>
<tr class="even">
<td><p><strong>fpsEnd</strong></p></td>
<td><p>Stops a FPS timer and prints the elapsed time.</p>
<p><strong>Syntax:</strong></p>
<pre>fpsEnd(const char* msgid, const char* kv_pairs, const char* msg);</pre></td>
</tr>
</tbody>
</table>
</div>

A measure point which describes what to measure consists of `msgid` and `kv_pairs`. The `msgid` and `kv_pairs` of two calls should be the same.

## Example of PmLogLib Performance API

The following is the sample code for how components written in QML can log information.

### Instance Version for Time Measure

``` javascript
import PerformanceLog 1.0
PerformanceLog {
     id: perfLog
     context: "com.webos.mycomponent"
}
perfLog.time("APP_LAUNCH", {"APP_ID": appId});
perfLog.timeEnd("APP_LAUNCH", {"APP_ID": appId}, "Web application is launched from Showcase");
perfLog.time("APP_SWITCH", {"APP_FROM": "Angry Birds", "APP_TO": "Chess"});
perfLog.timeEnd("APP_SWITCH", {"APP_FROM": "Angry Birds", "APP_TO": "Chess"});
perfLog.time("BROWSER_LOAD");
perfLog.timeEnd("BROWSER_LOAD");
```

Output:

``` plaintext
1970-01-01T00:01:09.303503Z [69] 192 local0.info DBGFRWK[1552]: com.webos.mycomponent APP_LAUNCH {"APP_ID": "com.webos.app.facebook", "UNIT": "ms", "TIME": 2} Web application is launched from Showcase
1970-01-01T00:01:09.304773Z [69] 192 local0.info DBGFRWK[1552]: com.webos.mycomponent APP_SWITCH {"APP_FROM": "Angry Birds", "APP_TO": "Chess", "UNIT": "ms", "TIME": 1}
1970-01-01T00:01:09.305467Z [69] 192 local0.info DBGFRWK[1552]: com.webos.mycomponent BROWSER_LOAD {"UNIT": "ms", "TIME": 1}
```

### Instance Version for FPS Measure

``` javascript
import PerformanceLog1.0
PerformanceLog {
     id: perfLog
     context: "com.webos.mycomponent"
}
perfLog.fps("SPLASH_ANIMATION", {"APP_ID": appId});
perfLog.fpsEnd("SPLASH_ANIMATION", {"APP_ID": appId}, "Web application is launching");
perfLog.fps("CARD_TRANSITION", {"STATE_FROM": "HIDE", "STATE_TO": "TWOANDHALF"});
perfLog.fpsEnd("CARD_TRANSITION", {STATE_FROM": "HIDE", "STATE_TO": "TWOANDHALF"});
perfLog.fps("LAUNCHER_READY");
perfLog.fpsEnd("LAUNCHER_READY");
```

Output:

``` plaintext
1970-01-01T00:01:09.306372Z [69] 192 local0.info DBGFRWK[1552]: com.webos.mycomponent SPLASH_ANIMATION {"APP_ID": "com.webos.app.facebook", "UNIT": "fps", "FPS": 0} Web application is launching
1970-01-01T00:01:09.307689Z [69] 192 local0.info DBGFRWK[1552]: com.webos.mycomponent CARD_TRASITION {"STATE_FROM": "HIDE", "STATE_TO": "TWOANDHALF", "UNIT": "fps", "FPS": 0}
1970-01-01T00:01:09.308302Z [69] 192 local0.info DBGFRWK[1552]: com.webos.mycomponent LAUNCHER_READY {"UNIT": "fps", "FPS": 0}
```

### Singleton Version for Time Measure

``` javascript
import PerformanceLog1.0
PerformanceLogger.context = "qmlPerformance";

PerformanceLogger.time("APP_LAUNCH", {"APP_ID": appId});
PerformanceLogger.timeEnd("APP_LAUNCH", {"APP_ID": appId}, "Web application launched from Showcase);
PerformanceLogger.time("APP_SWITCH", {"APP_FROM": "Angry Birds", "APP_TO": "Chess"});
PerformanceLogger.timeEnd("APP_SWITCH", {"APP_FROM": "Angry Birds", "APP_TO": "Chess"});
PerformanceLogger.time("BROWSER_LOAD");
PerformanceLogger.timeEnd("BROWSER_LOAD");
```

Output:

``` plaintext
1970-01-01T00:01:09.303503Z [69] 192 local0.info DBGFRWK[1552]: qmlPerformance APP_LAUNCH {"APP_ID": "com.webos.app.facebook", "UNIT": "ms", "TIME": 2} Web application is launched from Showcase
1970-01-01T00:01:09.304773Z [69] 192 local0.info DBGFRWK[1552]: qmlPerformance APP_SWITCH {"APP_FROM": "Angry Birds", "APP_TO": "Chess", "UNIT": "ms", "TIME": 1}
1970-01-01T00:01:09.305467Z [69] 192 local0.info DBGFRWK[1552]: qmlPerformance BROWSER_LOAD {"UNIT": "ms", "TIME": 1}
```

### Singleton Version for FPS Measure

``` javascript
import PerformanceLog 1.0
PerformanceLogger.context = "qmlPerformance";

PerformanceLogger.fps("APP_LAUNCH", {"APP_ID": appId});
PerformanceLogger.fpsEnd("APP_LAUNCH", {"APP_ID": appId}, Web application is launching);
PerformanceLogger.fps("CARD_TRANSITION", {"STATE_FROM": "HIDE", "STATE_TO": "TWOANDHALF"});
PerformanceLogger.fpsEnd("CARD_TRANSITION", {"STATE_FROM": "HIDE", "STATE_TO": "TOWANDHALF"});
PerformanceLogger.fps("LAUNCHER_READY");
PerformanceLogger.fpsEnd("LAUNCHER_READY");
```

Output:

``` plaintext
1970-01-01T00:01:09.306372Z [69] 192 local0.info DBGFRWK[1552]: qmlPerformance SPLASH_ANIMATION APPCRASH {"APP_ID": "com.webos.app.facebook", "UNIT": "fps", "FPS": 0} Web application is launching
1970-01-01T00:01:09.307689Z [69] 192 local0.info DBGFRWK[1550]: qmlPerformance CARD_TRANSITION {"STATE_FROM": "HIDE", "STATE_TO": "TWOANDHALF", "UNIT": "fps", "FPS": 0}
1970-01-01T00:01:09.308302Z [69] 192 local0.info DBGFRWK[1552]: qmlPerformance LAUNCHER_READY { "UNIT": "fps", "FPS": 0}
```

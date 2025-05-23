# Overview
This is a library to interact with the Windows Event Logging system. The focus is to interact directly with the Windows API, rather than parsing evt files. This will allow you to use python to parse events as well as subscribe to providers.

It is a fork of [schorschii/winevt_ng](https://github.com/schorschii/winevt_ng) which is a fork of the abandoned [bannsec/winevt](https://github.com/bannsec/winevt) with some fixes and improvements.

# Install
`winevt_ng` can be installed directly as a package from pypi. I recommend you install it into a python virtual environment.

```bash
$ mkvirtualenv --python=$(which python3) winevt_ng # Optional
(winevt_ng)$ pip install winevt_ng
```

# Current Features
Currently, this library supports querying and subscribing to event logs or parsing of event log files. Because this library uses the Windows API directly, you can query for any of the reigstered event providers.

# Example
## Query
Let's say you want to review the error report alerts that are in your Application event log. To print out all the times you dropped a dump file, you could do the following:

```python
In [1]: from winevt_ng import EventLog

In [2]: query = EventLog.Query("Application","Event/System/Provider[@Name='Windows Error Reporting']")

In [3]: for event in query:
   ...:     for item in event.EventData.Data:
   ...:         if "dmp" in item.cdata:
   ...:             print(item.cdata)
```

If you were interested in seeing every time you had an error or critical event from the System, you could do:

```python
In [1]: from winevt_ng import EventLog

In [2]: query = EventLog.Query("System","Event/System[Level<=2]")

In [3]: for event in query:
   ...:     print(event.System.Provider['Name'])
```

## Subscription
Let's say you want to watch for new Errors and Critial events from the System log, and want to be able to take some form of immediate action. You can acomplish that through a subcription using a python function as your callback.

```python
In [1]: from winevt_ng import EventLog

In [2]: def handle_event(action, pContext, event):
   ...:     print("Got event: " + str(event))
   ...:

In [3]: cb = EventLog.Subscribe("System","Event/System[Level<=2]",handle_event)

In [4]: Got event: <Event EventID=10016 Level=Error>
Got event: <Event EventID=10016 Level=Error>
Got event: <Event EventID=10016 Level=Error>
```

If you want to cancel your subscription, simply use the unsubscribe method:

```python
In [5]: cb.unsubscribe()
```

# EventLog.Event
The `EventLog.Event` class abstracts the concept of a Windows Event Log. There are likely two primary ways you would use this:

## Event.xml
Every `Event` object has an `xml` property to it. That property is the same XML you would find looking through the Windows Event Viewer. It is returned as a string and you can parse it however you wish.

## Event structure
Every `Event` object also has a structure to it. The structure is effectively the output of `untangle`. That said, it starts at the Event level so it makes Windows Events easier to traverse. Here's an example of XML:

```xml
<Event xmlns="http://schemas.microsoft.com/win/2004/08/events/event">
  <System>
    <Provider EventSourceName="Software Protection Platform Service" Guid="{E23B33B0-C8C9-472C-A5F9-F2BDFEA0F156}" Name="Microsoft-Windows-Security-SPP"/>
    <EventID Qualifiers="16384">903</EventID>
    <Version>0</Version>
    <Level>0</Level>
    <Task>0</Task>
    <Opcode>0</Opcode>
    <Keywords>0x80000000000000</Keywords>
    <TimeCreated SystemTime="2017-05-05T16:11:37.282412200Z"/>
    <EventRecordID>12126</EventRecordID>
    <Correlation/>
    <Execution ProcessID="0" ThreadID="0"/>
    <Channel>Application</Channel>
    <Computer>Phoenix</Computer>
    <Security/>
  </System>
  <EventData/>
</Event>
```

If this were the XML for our `Event` object, and we wanted to find out the TimeCreated, we could do the following:

```python
event.System.TimeCreated['SystemTime']
```

# Authenticate Local and Remote
You can authenticate locally and remotely. If you provide no extra details, you will by default authenticate locally as your current user. However, for both `Query` and `Subscribe`, you can provide the following optional arguments:

 - username
 - password
 - domain
 - server
 - auth (default, negotiate, kerberos, ntlm)

For example, if you wished to connect to a server using username "administrator", it would be:

```python
query = EventLog.Query("Security","*",username="administrator", server="myserver", domain="mydomain")
```

You would then be prompted for the password interactively.

# Bookmarks
If you want to ensure you're not losing your place, you can use bookmarks. The Bookmark class abstracts the fundamental Windows construct of a bookmark. Use of bookmarks can be done by:

1. Instantiate a new bookmark

```python
bookmark = EventLog.Bookmark()
```

1b. If you already have a bookmark, just feed in the xml

```python
bookmark = EventLog.Bookmark(xml)
```

2. Give the bookmark parameter to `Query` or `Subscribe`

```python
cb = EventLog.Subscribe("System","*",handle_event,bookmark=bookmark)
```

3. Save your bookmark by saving your xml however you wish

```python
bookmark.xml
```

The updating of the bookmark will occur behind the scenes for you.

# Multiple Subscription Support
This library supports subscribing to as many channels as you want. The caveat is that you should not allow your `Subscription` objects to be garbage collected. In practice, this just means don't overwrite your class variables. Even if you're not using them, keep them around so that python doesn't try to garbage collect them on you.

# Tested On
I have only tested this on my Windows 10 x64 system with python 3.6 x64. It should work across most Windows systems given a Python x64 version >=3.2 (cffi changes).

It will very likely NOT work on python 2.

It might work on python 3.2+ x86. Let me know your experience.

# Development
## Publishing
```
python setup.py sdist
twine upload -r pypi dist/*
```

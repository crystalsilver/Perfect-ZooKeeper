# Perfect-ZooKeeper[简体中文](README.zh_CN.md)

<p align="center">
    <a href="http://perfect.org/get-involved.html" target="_blank">
        <img src="http://perfect.org/assets/github/perfect_github_2_0_0.jpg" alt="Get Involed with Perfect!" width="854" />
    </a>
</p>

<p align="center">
    <a href="https://github.com/PerfectlySoft/Perfect" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_1_Star.jpg" alt="Star Perfect On Github" />
    </a>  
    <a href="http://stackoverflow.com/questions/tagged/perfect" target="_blank">
        <img src="http://www.perfect.org/github/perfect_gh_button_2_SO.jpg" alt="Stack Overflow" />
    </a>  
    <a href="https://twitter.com/perfectlysoft" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_3_twit.jpg" alt="Follow Perfect on Twitter" />
    </a>  
    <a href="http://perfect.ly" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_4_slack.jpg" alt="Join the Perfect Slack" />
    </a>
</p>

<p align="center">
    <a href="https://developer.apple.com/swift/" target="_blank">
        <img src="https://img.shields.io/badge/Swift-3.0-orange.svg?style=flat" alt="Swift 3.0">
    </a>
    <a href="https://developer.apple.com/swift/" target="_blank">
        <img src="https://img.shields.io/badge/Platforms-OS%20X%20%7C%20Linux%20-lightgray.svg?style=flat" alt="Platforms OS X | Linux">
    </a>
    <a href="http://perfect.org/licensing.html" target="_blank">
        <img src="https://img.shields.io/badge/License-Apache-lightgrey.svg?style=flat" alt="License Apache">
    </a>
    <a href="http://twitter.com/PerfectlySoft" target="_blank">
        <img src="https://img.shields.io/badge/Twitter-@PerfectlySoft-blue.svg?style=flat" alt="PerfectlySoft Twitter">
    </a>
    <a href="http://perfect.ly" target="_blank">
        <img src="http://perfect.ly/badge.svg" alt="Slack Status">
    </a>
</p>



This project implements an express Swift library of ZooKeeper

This package builds with Swift Package Manager and is part of the [Perfect](https://github.com/PerfectlySoft/Perfect) project.


## Release Note

This project can only be built on ⚠️ Ubuntu 16.04 ⚠️ . Mac OS X doesn't support it.

## Quick Start

### Swift Package Manager

Add Perfect-ZooKeeper to your project's Package.swift:

``` swift
.Package(url: "https://github.com/PerfectlySoft/Perfect-ZooKeeper.git", majorVersion: 1)
```

### Import Library

Import Perfect-ZooKeeper library to your source code:

``` swift
import PerfectZooKeeper
```

### Debug

To debug your ZooKeeper application, Perfect-ZooKeeper provides a static method called `debug` to set the debug level, for example:

``` swift
// this will set debug level to the whole application
ZooKeeper.debug()
```

You can also adjust the debug level to method `debug(_ level: LogLevel = .DEBUG)` by a parameter:

- level: LogLevel, the debug level, could be .ERROR, .WARN, .INFO, or .DEBUG by default

### Log

To trace and log your ZooKeeper application, Perfect-ZooKeeper provides a static method called `log`, for example:

``` swift

// this will redirect the debug information to standard error stream
ZooKeeper.log()

```

The only parameter of `log(_ to: UnsafeMutablePointer<FILE> = stderr)` is `to`, a `FILE` pointer as in C stream, it is `stderr` by default but you can redirect it to any available `FILE` streams.

### Using ZooKeeper Object

Before performing any actual connections, it is necessary to construct a `ZooKeeper` object:

``` swift
let z = ZooKeeper()
```

Or alternatively, you can also add a timeout setting to such an object, which defines the maximal milliseconds to wait for connection:

``` swift
// indicates that the connection attempt will be treated as broken in eight seconds.
let z = ZooKeeper(8192)
```

### Connect to ZooKeeper Hosts

Use `ZooKeeper.connect()` to connect to specified hosts. Take example, the demo below shows how to connect to a ZooKeeper host, and how the program invokes your callback once connected:

``` swift
try z.connect("servername:2181") { connect in
  switch(connect) {
  case .CONNECTED:
    // connection is made
  case .EXPIRED:
    // connection is expired
  default:
    // connection is broken
  }
}
```

⚠️ NOTE ⚠️ , you may also connect to a cluster of host by replacing the above connection string into a string of multiple hosts in such an expression: "server1:2181,server2:2181,server3:2181", but this may be subject to the ZooKeeper version that you are connecting with. For more information of ZooKeeper connection string, see [ZooKeeper Programmer's Guide](https://zookeeper.apache.org/doc/trunk/zookeeperProgrammers.html)


### Existence of a ZNode

Once connected, you may check a specific ZNode by calling `exists()`, for example:

``` swift
let a = try z.exists("/path/to")
print(a)
```

This function will return a `Stat()` structure if nothing wrong, for example:
``` swift
// this is a sample result of calling print(try z.exists("/path/to"))
Stat(czxid: 0, mzxid: 0, ctime: 0, mtime: 0, version: 0, cversion: -1, aversion: 0, ephemeralOwner: 0, dataLength: 0, numChildren: 1, pzxid: 0)
```

### List Children of a ZNode

Method `children()` may list all available direct sub nodes under the objective and put them into an array of string.

``` swift
let kids = try z.children("/path/to")
// if success, it will list all sub nodes under /path/to in an array.
// for example, if there is /path/to/a and /path/to/b, then the result is probably ["a", "b"]
print(kids)
```

### Save Data to a ZNode

As a key-value directory, each ZNode may contain a small amount of data in form of a string, usually not exceed to 10k. You can save your own configuration data into a ZNode, synchronously or asynchronously, as demanded.

#### Save Data Synchronously

Synchronous version of `save()` will return a `Stat()` structure if success:

``` swift
let stat = try z.save("/path/to/key", data: "my configuration value of key")
print(stat)
```

Parameters of `func save(_ path: String, data: String, version: Int = -1) throws -> Stat`:
- path: String, the absolute full path of the node to access
- data: String, the data to save
- version: Int, version of data, default is -1 which indicates ignoring the version info

#### Save Data Asynchronously

Asynchronous version of `save()` has all the same parameters with an extra `StatusCallback` but without returning value:

``` swift
try z.save("/path/to/key", data: "my configuration value of key") { err, stat in
  guard err == .ZOK else {
    // something wrong
  }
  guard let st = stat else {
    // async save() returns a null status
  }
  // print the status after saving
  print(st)
}
```

### Load Data from a ZNode

Similar to `save()`, the ZooKeeper `load()` also has both synchronous version and asynchronous version as well:

#### Load Data Synchronously

To load data from a ZNode synchronously, simply call `load("/path/to")`, and it will return a tuple of (Value: String, Stat):

``` swift
let (value, stat) = try z.load("/path/to")
```

#### Load Data Asynchronously

The data loading from a ZNode asynchronously will require an extra callback with a parameter of (error: Exception, value: String, Stat) as demo below:

``` swift
try z.load(path) { err, value, stat in
  guard err == .ZOK else {
    // something wrong
  }//end guard
  guard let st = stat else {
    // there is no status information of node
  }//end guard
  print(st)
  // this is the actual data value as a String
  print(value)
}//end load
```

### Make a Node

Function `func make(_ path: String, value: String = "", type: NodeType = .PERSISTENT, acl: ACLTemplate = .OPEN) throws -> String` can build different type of nodes, with writing data value and set ACL (Access Control List) info for this node in the same moment. Here are the parameters:

- path: String, the absolute full path of the node to make
- value: String, the value to store into node
- type: NodeType, i.e., .PERSISTENT, .EPHEMERAL, .SEQUENTIAL, or .LEADERSHIP, which means ephemeral + sequential. Default type is .PERSISTENT
- acl: ACLTemplate, basic ACL template to apply in this incoming node, i.e., .OPEN, .READ or .CREATOR. Default is .OPEN, which means nothing to restrict

⚠️ Note ⚠️ The return value will be a string formed serial number only if the node type is .SEQUENTIAL or .LEADERSHIP, otherwise it should be empty and ignored.

#### Make a Persistent Node

The following code demonstrates how to create a persistent node with data:

``` swift
let _ = try z.make("/path/to/key", value: "my config data value for this key")
```

#### Make a Temporary Node

A temporary ZNode means it will automatically disappear once the session was over (usually after a few seconds of disconnection). To create such a node, simply add a node type parameter to call:

``` swift
let _ = try z.make("/path/to/tempKey", value: "data for this temporary key", type: .EPHEMERAL)
```

#### Make a Sequential Node

A Sequential ZNode means if you want to create a `/path/to/key` node, it will return a `/path/to/key0123456789`, i.e., a 10 digit number will be added to the node you named, and `make()` method will return this newly & automatically generated incremental serial number, in a form of string:

``` swift
let serialNum = try z.make("/path/to/myApplication", type: .SEQUENTIAL)
print(serialNum)
// if success, the serialNum will be something like `0000000123` and the node will be `/path/to/myApplication0000000123`
```

⚠️ Note ⚠️ Sequential node is persistent and can be removed only by calling `remove()` method explicitly.

#### Make a Leadership Node

The purpose of leadership node is to select a leadership server among all candidates. Similar to .SEQUENTIAL, the leadership node is also a temporary node, which help all clustered backups to determine who shall be the leader among the cluster by checking whose serial number is the minimal one, which means who is the first available.

``` swift
let serialNum = try z.make("/path/to/myApplication", type: .LEADERSHIP)
print(serialNum)
// if success, the serialNum will be something like `0000000123` and the node will be `/path/to/myApplication0000000123`
// then you could probably use `children()` to check if there are any engaging candidate instances in competition.
// If the current instance get the minimal serial number, then it shall be the leader of cluster;
// otherwise the candidates shall wait for opportunities to win the leadership.
```

### Remove a Node

Method `func remove(_ path: String, version: Int32 = -1)` enables the function to delete an existing node:

``` swift
// this action will result in a removal regardless node versions.
try z.remove("/path/to/uselessNode")
```

### Watch for Changes

Perfect-ZooKeeper provides a useful function `watch()` to monitor other instance operations against a specific node.
The full API of `watch()` method is `func watch(_ path: String, eventType: EventType = .BOTH, renew: Bool = true, onChange: @escaping WatchCallback)` with parameter explained below:

- path: String, the absolute full path of the node to watch
- eventType: watch for .DATA or .CHILDREN, or .BOTH
- renew: watch the event for once or for ever, false for once and true for ever.
- onChange: WatchCallback, callback once something changed

For example:

``` swift
try z.watch("/path/to/myCheese") { event in
  switch(event) {
  case CONNECTED:
    // me myself just connected to this node???
  case DISCONNECTED:
    // connection is broken
  case EXPIRED:
    // connection is expired
  case CREATED:
    // this shall never happen - just created for the node me watch?
  case DELETED:
    // the node has been deleted by someone else
  case DATA_CHANGED:
    // someone just touched my cheese
  case CHILD_CHANGED:
    // children were changed
  default:
    // unexpected here
  }
}//end watch
```


## Issues

We are transitioning to using JIRA for all bugs and support related issues, therefore the GitHub issues has been disabled.

If you find a mistake, bug, or any other helpful suggestion you'd like to make on the docs please head over to [http://jira.perfect.org:8080/servicedesk/customer/portal/1](http://jira.perfect.org:8080/servicedesk/customer/portal/1) and raise it.

A comprehensive list of open issues can be found at [http://jira.perfect.org:8080/projects/ISS/issues](http://jira.perfect.org:8080/projects/ISS/issues)

## Further Information
For more information on the Perfect project, please visit [perfect.org](http://perfect.org).

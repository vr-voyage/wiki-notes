---
title: Basics of the OSC protocol in VRChat
description: The basics of the OSC protocol, and how to use this protocol on VRChat.
published: true
date: 2022-02-17T22:05:55.686Z
tags: vrchat, osc
editor: markdown
dateCreated: 2022-02-17T22:05:55.686Z
---

# OSC Protocol basics and usage with VRChat

## OSC Protocol

### Quick explanation

#### Messages

The protocol is pretty basic, notably with VRChat.

An OSC message is formatted like this :

* (String) Endpoint address, starting with `/`
* (String) Format of the following data elements, starting with `,`
* (Binary encoding) Encoded data

Strings are ASCII encoded and null-terminated, and are padded with null characters to align them on 4 bytes boundaries.

For a list of default useable endpoints on VRChat, see :
https://docs.vrchat.com/v2022.1.1/docs/osc-as-input-controller

#### Usage

On VRChat, the OSC packets are sent/received through UDP.
By default, VRChat receive data on port 9000 and send them on port 9001.
You can [change these defaults settings through the command line](https://docs.vrchat.com/v2022.1.1/docs/osc-overview) :

* `--osc=InputPortN:AddressWhereToSend:SendPortN` where :
  * `InputPortN` is the port number where you will send data to VRChat (**input**)
  * `AddressWhereToSend` is the address where VRChat should send data to (**output**)
  * `SendPortN` is the port number where VRChat will send data to (**output**)

#### Example

Moving the character forward by :
* Using the endpoint `/input/Vertical`
* Sending the **floating-point** (`f`) number **1.0**

Using the Ruby programming language (any will do) :

```ruby
require 'socket'
s = UDPSocket.new

s.send(
	mesg="/input/Vertical\0,f\0\0\x3F\x80\x00\x00",
  flags=0,
  host=127.0.0.1,
  port=9000)
```

#### Explanation

##### Endpoint

The endpoint `/input/Vertical` is encoded as a String so it needs to be NULL-terminated and aligned on 4 bytes :
* `/input/Vertical\0` : 16 bytes

##### Format

Only one **floating-point number** will be sent, so the format will be :
* `,f`

The format is a encoded as a String, so it needs to be NULL-terminated and aligned on 4 bytes :
* `,f\0\0` : 4 bytes

##### Data

Then the floating-point number **1.0** needs to be binary-encoded (as a 32-bits floating-point number, little endian) :
* 1.0 → `3F 80 00 00` (`\x3F\x80\x00\x00` in Python/Ruby strings)


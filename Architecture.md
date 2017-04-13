mqtt-smarthome
==============

_Note that this project is not associated with or endorsed by http://mqtt.org_

Overview
--------
This document describes an architectural concept for a Smart Home Automation system
which ties together heterogeneous hardware using a centralized message bus: MQTT.

_Effectively, it just defines a convention for exchanging messages, and a message format._

It's goal is to provide an interoperability layer close to the hardware level.
Logic and Visualization are to be implemented at a higher level, on top of the MQTT bus.

```
+---------------+                                                          
|               |                        +----------------------+          
| Logic Engine  +-------+          +-----+ Hardware Interface A |          
|               |       |          |     | (e.g. KNX)           |          
+---------------+    +--+----------+-+   +----------------------+          
                     |               |                                     
                     |  MQTT-Broker  |                                     
                     |               |                                     
+---------------+    +--+----------+-+   +----------------------+          
|               |       |          |     | Hardware Interface B |          
| Visualization +-------+          +-----+ (e.g. Homematic)     |          
|               |                        +----------------------+          
+---------------+                                                          
```                                                                           

This convention does make no attempt at abstracting hardware messages into
higher level concepts like "Lights" or "Switches". Current home automation 
hardware and their hardware interfaces have a wide range of different design 
concepts, ranging from simply addressing devices directly (like many basic RF
protocols) over abstracting devices into channels and datapoints
(e.g. Homematic) to the link-orientied Group Address concept used by KNX.
This abstraction, if required, needs to happen at a higher level, e.g.
inside a logic engine. 


Message Bus
-----------
A MQTT broker (http://en.wikipedia.org/wiki/MQTT) serves as the central message bus.
In short, MQTT allows applications to exchange messages under so-called "topics".
Applications which are interested in certain topics _subscribe_ to them,
and other applications _publish_ messages to them.

                                                        
## Topic Hierarchy ##
                                                                           
MQTT topics are hierarchically structured, much like files and directories in a
file system. Example:

    toplevelname/function/item...
    
By convention, in mqtt-smarthome the *toplevelname* will always refer to a specific
instance of a hardware interface or other participent in the system. 

The second level *function* defines a function like _status_, _set_ etc.

The subsequent levels are  defined by the specific interface.

For example, a KNX gateway might reflect the KNX Group Address hierarchy, which
might look like

    knxgateway1/status/Kitchen/Lights/Front Left

whereas a Homematic interface may represent device/channel names and datapoints:

    hm/status/Light Kitchen Front Left/LEVEL

The *toplevelname* should be configurable so that multiple instances of interfaces
can coexist on a single message bus.

    
### Status reports ###
                                                                           
Interfaces report states (values from sensors, feedback from actuators)
by publishing into the topic

    toplevelname/status/itemname

The *retain* flag should be set depending on whether the status is a one-shot
event (e.g. a keypress) or a measurement or other state (e.g. a temperature).
One-shot events should not be retained, whereas states should have *retain*
set.

### Change/Action requests ###

Visualization UIs and Logic Engines request state changes by publishing 
the requested new value into the topic

    toplevelname/set/itemname
 
The _itemname_ hierarchy for _set_ should be the same as the one used for the _status_
function tree.

Messages published to this hierarchy should never have the MQTT *retain* flag set.
          
                                                                           
### Connected status ###
                                                                           
Each interface should maintain a topic 

    toplevelname/connected
    
which is an integer enum:

  * 0 - disconnected from MQTT broker
  * 1 - connected to MQTT, but disconnected from hardware
  * 2 - connected to MQTT and hardware, i.e. fully operational

It should ensure that this is set to _0_ using MQTT's
_Last Will and Testament_ functionality upon a disconnect.
                                                                           

#### Active get ####

Interfaces which support active value requests from hardware
devices (like KNX or Homematic) may support a hierarchy

    toplevelname/get/itemname
    
A write of any value to that hierarchy should trigger an active read
operation of the given _itemname_. The result of the read should be
published by the interface as a _status_ update.

Messages published to this hierarchy should never have the MQTT *retain* 
flag set.


#### Command channel ####

Interfaces may support a command scheme which may not be representable
by the normal item hierarchy. In this case, the interface may listen to

    toplevelname/command

to receive such commands.

Messages published to this hierarchy should never have the MQTT *retain* 
flag set.

### QoS recommendations ###

MQTT supports various QoS (Quality of Service) levels for messages and
connections:

  * 0 - no guaranteed delivery, no duplicates possible
  * 1 - guaranteed delivery, *duplicates possible*
  * 2 - guaranteed delivery, no duplicates possible

In the context of this proposal, it is recommended that QoS 1 is best avoided
due to the chance that messages may be delivered multiple times.

It should be assumed that duplicated messages in the _set_,  _get_ and
_command_ hierarchies may cause unwanted repeated hardware interaction, and 
that duplicated messages in the _status_ hierarchy may trigger unwanted
repeated events in a logic layer.


Message format
--------------
The message format for _status_ reports may either be a simple
value or a JSON encoded object which contains the simple value
and possible additional information.

The message format for _set_ operations is always a simple value.

The actual type of *simple value* depends on the underlying hardware.
It may be a boolean, a integer or floating point number, or a string.

If the _status_ message is a JSON encoded object, the object will
always have at least the field _val_, which holds aforementioned
simple value.

Additional possible fields:

  * ts - the timestamp when the value was obtained. Milliseconds since Epoch.
  * lc - the timestamp when the value last changed. Milliseconds since Epoch.

Example:

```
{
	"val": 17.9,
	"ts": 1421792677000,
	"lc": 1421792677000
}
```
                                                                          
Interfaces may include additional fields in the object. Those fields
always should be prefixed with a mnemonic identifier denoting the
interface. For example, a KNX interface may include a field
"knx_src_addr" which holds the originating bus address of a KNX write
message.


Feedback
--------
Feedback is welcome. Feel free to open an issue or a pull request
on GitHub.


Logo
----
The project logo is based in part on work by http://brsev.deviantart.com/
licensed as Creative Commons Attribution Non-commercial (by-nc)


History
-------
* V0.1 - 2015-01-05 - owagner
  - first draft
* V0.2 - 2015-01-11 - owagner
  - interfaces may define additional topics
  - recommendation that interfaces should have a configurable topic prefix
  - expanded architecture rationale
* V0.3 - 2015-01-12 - owagner
  - clarify that the "connected" topic is also supposed to be a JSON
    object
* V0.4 - 2015-01-20 - owagner
  - major revamp:
    - split topic hierarchies into status/set subhierarchies, thereby
      getting rid of the "ack" flag
    - messages may now be simple values as well as JSON encoded objects
    - defined "ts" and "lc" timestamps
    - extended "connected" into an enum to differentiate between
      a running interface with and without active hardware connection
* V0.5 - 2015-01-21 - owagner
  - clarified retain and QoS recommendations (thanks to bartbes)

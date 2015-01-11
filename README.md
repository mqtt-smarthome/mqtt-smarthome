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

    toplevelname/midlevel/item
    
By convention, in mqtt-smarthome the *toplevelname* will always refer to a specific
instance of a hardware interface or other participent in the system, whereas
the subsequent levels are defined by the specific interface.

For example, a KNX gateway might reflect the KNX Group Address hierarchy, which
might look like

    knxgateway1/Kitchen/Lights/Front Left

whereas a Homematic interface may represent device/channel names and datapoints:

    hm/Light Kitchen Front Left/LEVEL

The *toplevelname* should be configurable so that multiple instances of interfaces
can coexist on a single message bus.
    
### Status report ###
                                                                           
Interfaces report states (values from sensors, feedback from actuators)
by publishing into a specific topic. 

### Change/Action requests ###

Visualization UIs and Logic Engines request state changes by publishing 
the requested new value into the given topic 
                                                                           
### Additional topics ###
                                                                           
Each interface should maintain a topic 

    toplevelname/connected
    
which is a boolean showing whether the interface is currently connected to the
MQTT broker. It should ensure that this is set to _false_ using MQTT's
_Last Will and Testament_ functionality upon a disconnect.
                                                                           
Interfaces may specify further topics below *toplevelname* to provide
e.g. command channels or special status values. 

Message format
--------------                                                                           
The message format is always a text string which carries a JSON encoded
object, with two mandatory fields:

* val: the actual value
* ack: a boolean denoting whether the value is a _confirmed_ value (e.g.
from the hardware) or a _requested_ value (e.g. requested by user input
from a Visualization UI, or from a logic engine).

```
    {
      "val": 23.1,
      "ack": true
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
  
  

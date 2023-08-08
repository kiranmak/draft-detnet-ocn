---
abbrev: ocn-in-detnets
docname: draft-km-detnet-for-ocn-latest
title: Using Deterministic Networks for Industry Operations and Control
date:  false
category: info
stream: independent

ipr: trust200902
area: "Internet"
workgroup: "Detnet Group"
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
-

    ins: K. Makhijani
    name: Kiran Makhijani
    organization: Futurewei
    email: kiran.ietf@gmail.com

-
    ins: T. Faisal
    name:  Tooba Faisal
    organization: King's College London
    email: tooba.hashmi@gmail.com
-
    ins: R. Li
    name: Richard Li
    org: Futurewei
    email: richard.li@futurewei.com
-
    ins: C. Westphal
    name: Cedric Westphal
    org: Futurewei
    email: cedric.westphal@futurewei.com

informative:
  FACTORY:   I-D.wmdf-ocn-use-cases
  VIRT-PLC:  I-D.km-iotops-iiot-frwk
  PTP-GRID: DOI.10.1109/IEEESTD.2016.7479438
  NIST-OT:  DOI.10.6028/NIST.SP.800-37r2

--- abstract

Remote industrial processes enable control & operations from the
software-defined application logic. In order to support process automation
remotely, not only Deterministic Networks (DetNet) are needed but an interface
between the application endpoints to the devices over a DetNet infrastructure
is also required. This document describes an interface to deterministic
networks from the view of endpoints to support process control and operations.

--- middle

# Introduction {#intro}

Process automation systems involve operating a piece of equipment (such as
actuating and/or sensing field devices). The communication between the
controllers and field devices exhibits a well-defined set of behaviors and
has specific characteristics: the delivery of a control-command to a
machine must be executed within the time frame specified by a controller or
by an application to provide reliable and secure operation.  A low or zero
tolerance to latency and packet losses (among other things) is implied.

The endpoints (controllers and field devices) embody machine-to-machine
communications to facilitate both remote and local process automation. In this
document, networks that support all the characteristics of remote process
automation are referred to as Operation and Control Networks (OCNs) for
convenience. This document describes using DetNet to enable OCN applications
 since they provide mechanisms for guaranteed delay aware packet delivery,
reliability, and for packet loss mitigation.

This document defines the interface between an OCN application and DetNet
framework. i.e., using DetNet services for communication between the
controllers and the field devices. This interface is used by an application to
express its network-specific requirements. This document presents the
perspective of an end system. Because IP network stack is widely used by
general-purpose applications and provides more connection flexibility to end
systems, the scope our discussion is specific to the IP-enabled
DetNet data planes {{!DETNET-DP=RFC8655}}. For the other type of field devices,
service level proxy is assumed (section  4.1 in RFC8655).

Mapping OCNs to DetNet helps with developing a better understanding of how
DetNets can be used in such scenarios. To this end, the document provides a
background on {{background}} the type of traffic patterns in OCN applications.
It proposes an interface between an application and DetNet, and a potential
solution direction to support OCN traffic patterns over DetNet, as covered in
{{approaches}}.

# Terminology

- Operational Technology (OT):
 : Programmable systems or devices that interact
with the physical environment (or manage devices that interact with the physical
environment). These systems/devices detect or cause a direct change through the
monitoring and/or control of devices, processes, and events. Examples include
industrial control systems, building management systems, fire control systems,
and physical access control mechanisms. Source: {{NIST-OT}}

- Industral Automation:
  : Mechanisms that enable machine-to-machine communication by use of
technologies that enable automatic control and operation of industrial devices
and processes leading to minimizing human intervention.

- Control Loop:
 : Control loops are part of process control systems in which
desired process response is provided as input to the controller, which
performs the corresponding action (using actuators) and reads the output values.
Since no error correction is performed, these are called open control loops.

- Feedback Control Loop:
  : A feedback loop is part of a system in which some portion (or all) of
the system's output is used as input for future operations.

- Industrial Control Networks:
 : Industrial control networks are the
interconnection of equipment used for the operation, control or monitoring of
machines in the industral environment. It involves a different level of
communication - between fieldbus devices, digital controllers and software
applications

- Human Machine Interface (HMI):
: An interface between the operator and the machine.
The communication interface relays I/O data back and forth between an operator's
terminal and HMI software to control and monitor equipment.

## Acronyms

- HMI: Human Machine Interface
- OCN: Operations and Control Networks
- PLC: Programmable Logic Control
- OT: Operational Technology
- OC: Operation and Control
- OCN: Operation and Control Networks

# Background on Industrial Control Systems {#background}

An industrial control network interconnects devices used to operate, control and
monitor physical equipment in industrial environments. {{icn-arch}} below shows
such systems' reference model and functional components. Closest to the
physical equipment are field devices (actuators and sensors) that connect to
the Programmable Logic Controllers (PLCs) or other types of controllers using
serial bus technologies (and now Ethernet).  Above those controllers are Human
Machine Interface (HMI) connecting different PLCs and performing several
controller functions along with exchanging data with the applications.

A factory floor is divided into cell-sites. The PLCs or other types of
controllers are physically located close to the equipment in the cell-sites.
The collection of monitoring, status and sensing data is first done on the site
and then transmitted over secure channels to the data applications for
aggregation and further processing. These applications can be hosted
in remote cloud infrastructure, but are also often hosted within a
limited domain environment, controlled by a single operator, like
on-premise, at the edge or in a private cloud. Both options gain
from infrastructure that scales out, has elastic compute and storage
resources, so they will both be referred to as cloud in the following sections.

~~~~~drawing

        +-+-+-+-+-+-+
     ^  | Data Apps |....            External business-logic
     :  +-+-+-+-+-+-+   :                Network
     :        |         :
     v  +-+-+-+-+-+-+  +-+-+-+-+--+
        | vendor A  |  |vendor B  |  Interconnection of
        | controller|  |controller|  controllers
     ^  +-+-+-+-+-+-+  +-+-+-+-+-+   (system integrators)
     :       |         |
     :   +-+-+-+-+  +-+-++-+
     :   | Net X |  | Net Y|
     v   | PLCs  |  | PLCs |--+    device-controllers
     ^   +-+-+-+-+  +-+-+--+  |
     :      |        |        |
     :   +-+-+    +-+-+    +-+-+
     v   |   |    |   |    |   |   Field devices
         +-+-+    +-+-+    +-+-+
~~~~~
{: #icn-arch title="Functions in Industrial Control Networks"}

What is changing now is that data applications are integrating process control
functions to improve automation and to make real-time decisions,
programmatically. The equipment control and  collection of data generated by
the sensors should be possible over small or large-scale deterministic networks
as illustrated in {{new-arch}}.

~~~~~drawing

               +-+-+-+-+-+-+-+-+
               |     Data Apps |      Integrated Apps with
               | c1 | c2  | c3 |      Remote process control
               +-+-+-+-+-+-+-+-+
                \   ,-----.   /
                 +-[  Det- ]-+
                   [Network]
                    `-----'
               +-+-+-|  |-+-+-+-+
               |        |       |
             +-+-+    +-+-+   +-+-+
             |   |    |   |   |   |   Field devices
             +-+-+    +-+-+   +-+-+
~~~~~
{: #new-arch title="Converged Cloud based Industrial Control Networks"}

One particular motivation is to provide the behavior of a field bus between
the cloud and the actuators/sensors with the same assurance of reliability and
latency, albeit over wide area networks (WAN). This is evident from many
industrial control applications, such as factory automation {{FACTORY}}, PLC
virtualization {{VIRT-PLC}}, power grid operations {{PTP-GRID}}, etc. that are now
expected to operate in the cloud by leveraging virtualization and shared
infrastructure wherever possible.

## Connected Controllers, Sensors and Actuators

{::comment}
## Reference Points for Connecting Controllers, Sensors and Actuators
{:/comment}

Control systems comprise Controllers, Sensors and Actuators. The data
traffic essentially carries instructions that cause machines or equipment
to move and do things within or at a specific time. The connectivity exists in
the following manner:

- A controller interfaces with the sensors and actuators. The controller knows an
application's performance parameters which are expressed in terms of network
specific requests or resources such as tolerance to packet loss, latency limits,
jitter variance, bandwidth, and specification for safety.  The controller knows
all the packet delivery constraints.

- An actuator receives specific commands from the controllers. The DetNet should
be able to enable control of actuating devices remotely from the controller
while meeting all the requirements (or key performance indicators - KPIs)
necessary for successful command
execution. The actuator participates in a closed control loop as needed.

- A sensor emit periodic data from the sensors. It may intermittently provide
asynchronous readings upon request from the controller. Sensors may report
urgent messages regarding malfunctioning in certain equipment, cell-sites, or
zones.

In many control systems there is at least one controller (or server) entity on
one end and two other entities - the sensors and actuators on the other end.
The communication with sensors and actuators is through the controller entity;
as such data applications do not directly interact with the field devices.
Neither actuators nor sensors perform decision-making tasks. This
responsibility belongs to the controller.

## Generalized Communication Model

To describe networked process control behavior a conceptual communication model
is used so that the data applications do not concern with the details of the
networks realizing operations and control. We refer to this model as operation
and control network (OCN). The scope of the model is

- Logical reference points: identify an endpoint's role or function as
  sensor-point, actuation-point, or operation & control point (oc-point for
short). Note: term 'oc-point' is used to avoid confusion with network
controllers and term 'fd-point' is used when both type of field devices are
refered to.

- Interface specification: in terms of associated traffic patterns between the
  endpoints as described below in {{ocn-pattern}}. The interface may be any
type of network (Ethernet, IP, wireless, etc. The model assumes that the
network is capable of providing network services and resources necessary of the
application specific operations and control.

Depending on the design of the usecase the process controller functionality
(oc-point) may reside as a software module in the data application or as a
separate module. When deployed as a separate module, another connectivity
interface between the data application and oc-point will be needed and is out
of the scope of this document.

The applications will use an communication interface between oc-point and
sensor-point to receive sensory data and similarly interface between oc-point
to actuation-point to execute a single or a sequence of control instructions.

This abstraction provides an additional layer of  protection in the sense that
the traffic patterns between the reference points are well defined so any
exceptions can be easily caught.

## Traffic Patterns {#ocn-pattern}

For either local or wide area, the process automation activities over the
network can generate a variety of traffic patterns between the oc-point and
field devices such as:

### Control Loops {#c-loop}

The equipment being operated upon is sensitive to when a command request
actually executes. An actuator upon receiving a command (function code) will
immediately perform the corresponding action. It is the responsibility of network
and controller to ensure that behavior of the sensor and actuator follows the
expectations of applications.

For several such applications, the knowledge of a successful operation is equally
critical to advance to the next steps; therefore, getting the response back in
a specified time is required, leading to a knowledge of timing. These types of
bounded-time request and response mechanisms are called control loops.

Unlike general purpose applications, commands cannot be batched, the
parameters of the command that will follow depends on the result of the previous one.
Each request in control loop takes up a minimal payload size (function code,
value, device or bus address) and will often fit in a single short packet.

In Detnet-enabled network, it can be imagined as a small series of packets with
the same flow identifier, but with different latency constraints.

It is required to support control loops where each request presents its own
latency constraints to the network and where commands are small sized packets.

###  Periodicity {#ocn-intervals}

Sensors emit data at regular intervals, but this information may not always be
time-constrained. Usually, controllers are programmed to tolerate and record
intermittent losses.  Automation software can make a more informed decision by
monitoring a lot of sensor data.  Thus, the traffic volume generated by sensors is
expected to high.  The periodicity of each sensor can also vary based on the
equipment.

It is required that network capacity is planned appropriately for the periodic
traffic generated from the different sensors. The periodic interval should also
be preserved in the network because any variations could provide false
indications that the equipment is misbehaving.

### Ordering

In real-time process control communications, out of order message processing
will lead to costly failures of operations.  Messages such as request and
reply, or a sequence of commands may be correlated therefore, both time
constraints and order must be preserved. The traffic is generated when software
triggers control-commands to field devices. This may not always map into
asynchronous DetNet flows if observation interval is not known.

The network should be capable of supporting sporadic on-demand short-term flows.
This does not imply instantaneous resource provisioning, instead it would be
more efficient if the provisioned resources could be shared for such
asynchronous traffic patterns.

Another consideration with ordering is that both actuators and sensors are
low-resource devices.  They can not buffer multiple packets and execute them in
order while maintaining the latency bounds of each command execution.  This
means the network must pace packets that may arrive early.


### Urgency

Besides latency constrained and periodic messages, sensors also report failures
as fault notifications, such as pressure valve failure, abnormally high
humidity, etc. These messages must be delivered with utmost urgency and
immediately.


## Communication Patterns

Control systems follow a specific communication discipline. The field devices
(sensors and actuators) are always controlled, i.e., interact with the system
through controllers in the following manner:-

- Sensor to controller: data emitted at periodic interval providing
  status/health of the environment or equipment. The  traffic volume for this
communication is determined by the payload size of each  sensor data and the
interval. These are a kind of synchronous Detnet flows but with much higher time intervals; still the inter-packet gap should be minimum.

- Controller to/from actuator: the commands/instructions to write or read.
  Actuators generally do not initiate a command unless requested by the
controller. Actuators will often execute a command, read the corresponding
result, and send that in response to the original write command.  The traffic
profile will be balanced in both directions due to requests/ response behavior. These are like asynchronous flows but without the observation interval constraint.

{::comment}
However, the actual volume (or rate) will depend on the capabilities of the
field device (whether it needs to be operated instruction by instruction or can
be pre-programmed).
{:/comment}


# Gap Analysis {#gaps}

Today, most of the operations and control solutions are split approaches. This
means that the controller is on-premises close to the equipment, sensor data is
first collected on-site, and then bulk transmitted to the enterprise cloud for
further processing.

To support delivering remote instructions to the machines over wide area
networks using Deterministic Network data plane architecture
{{!DETNET-DP=RFC8655}} and corresponding data plane DetNet over IP
{{!DETNET-IP=RFC8939}} mechanisms apply as discussed in {{detnet-rel}}. Later in
{{depend}} additional asks from DetNet are covered.

## Deterministic Networks Relevance {#detnet-rel}

> Note: This section's text and explanation on DetNet can be removed.

~~~~drawing

 DetNet IP       Relay                        Relay       DetNet IP
 End System      Node                         Node        End System

+----------+                                             +----------+
|   Appl.  |<------------ End-to-End Service ----------->|   Appl.  |
+----------+  ............                 ...........   +----------+
| Service  |<-: Service  :-- DetNet flow --: Service  :->| Service  |
+----------+  +----------+                 +----------+  +----------+
|Forwarding|  |Forwarding|                 |Forwarding|  |Forwarding|
+--------.-+  +-.------.-+                 +-.---.----+  +-------.--+
         : Link :       \      ,-----.      /     \   ,-----.   /
         +......+        +----[  Sub- ]----+       +-[  Sub- ]-+
                              [Network]              [Network]
                               `-----'                `-----'

         |<--------------------- DetNet IP --------------------->|
~~~~~
{: #detnet-arch title="A Simple DetNet-Enabled IP Network, Ref. RFC8939"}

{{detnet-arch}} illustrates a DetNet-IP dataplane divided into two sublayers
and is specified in {{!RFC8939}} . The forwarding sub-layer is responsible for
resource allocation and explicit path functions, whereas the service sublayer
provides packet replications, sequence numbering, and other functions. Within
the Detnet nodes, resources are allocated a priori for a flow. The
DetNet-enabled end systems originate traffic encapsulated with Detnet
forwarding and service sub-layers; otherwise relay node will create those
based on information received from the end system (via traffic classification).

The DetNets support both asynchronous (by allocating resources for the
observation interval) and synchronous (with repeating schedules) flow behaviors
(Section 4.3.2 in {{!DETNET-DP=RFC8655}}). The granularity of traffic treatment
is at the flow level specified by 6-tuple information, including DSCP.

Realistically, leveraging DetNets for Operations and Control (OCN) traffic
patterns {{ocn-pattern}} directly depends on how DetNets will provide network
knowledge to the applications.

## DetNet related Considerations and Dependencies {#depend}

A DetNet-aware node should express the network requirements as part of either
forwarding sublayer or service sublayer. {{!DETNET-IP=RFC8939}} doesnot specify
the interface to how those sublayers are mapped. This can be a non-trivial
task, as an OCN-application, will first need some way to request its DetNet
service provider (DN-SP). The DN-SP is expected to allocate resources and
return a mapping  - possibly a DSCP (DetNet Qos) for each pair. This could be
become a tedius scaling problem as the number of controller-device pairs start
to grow or keep changing.

Given that only DSCP is available, field-device pair can pose issues such as:

  - How can application request the proper network-resource for each command?
  - How can an application  receive periodic data from sensors and with what interval?
  - What are the ways to differentiate a less sensitive (periodic) updates from
    urgent alarms.
  - Or  how to differentiate data received from a sensor vs an actuator (with
    stringent latency requirements) and process them accordingly?

These issues are described below in more detail.

### Operator vs Application view {#app}

The DetNet data plane is designed with a network-operator-centric approach. The
operator's view on dealing with large-scale networks is discussed as newer
requirement in {{!I-D.ietf-detnet-scaling-requirements}}. In order to use
resources efficiently, flow aggregation is done. The operators in industrial
control networks are not necessarily network experts; they simply need an
application side tool to dispatch their request to the Deterministic networks'
entry point. Especially, an OCN application may need many
controller-field-device (ctrl-flddev) pairs to the be mapped to different
aggregates.

As the number of ctrl-flddev pairs grow, their variable traffic profiles can
become hard to manage.

The Detnet service provisioning internals should be transparent to an OCN
application. This can be achieved with a common signaling or user-to-network
interface between the applications and DetNet.

### Flow reservation and classification {#class}

Inside the DetNet, flow identification is done using IP header and DSCP
information. These flow identifiers are then used by DetNet nodes to provide
the corresponding traffic treatment. For a variety of commands and sensor data, latency or interval
parameters will vary and DSCP maybe limited in expressing all the possible requirements.

{::comment}
Accordingly, resources are provisioned over
longer timescales, i.e., the model works for relatively predictable scenarios.
The problem is that the control loops in {{c-loop}} may be  short messages so that
one command is sent per packet, expecting a response from  the actuator in
another return packet. The transmission of the next set of commands is driven
programmably by the applications. This is how the softwarization of industrial
processes is happening now.

Perhaps, it can be stated that the provisioning resources for flows does not necessarily
guarantee that the Detnet-specific resource contention at the instant will not occur.

Moreover, for any cloud-based solution, controller may as well send commands to
the devices from different locations (different IP addresses), thus the scale of
provisioned flows can grow very fast.

The applications in on-demand production pipelines could modify the pace and plan
for the production programmatically based on various other factors. Therefore,
it is not efficient to provision such networks for long periods. his should be
understood that flow-specific resource reservation planning is not useful.
While there is a benefit to a coarse granularity of flow classifications, there
is also a requirement for a finer granularity of DetNet traffic treatment.
{:/comment}

Embedding requirements explicitly can provide deterministic scheduling even in
an otherwise link that can be congested  when used with non-deterministic flows.


### Split Traffic flows {#split}

One of the most constrained design elements in today's industrial control
systems is that data from the sensors is collected on-site and often aggregated
before transporting
to the cloud.  Historical reasons for this approach do not apply anymore.  Due
to growth in sensor data, it now requires a much larger on-site storage
infrastructure which is expensive.  Applications also expect real-time
streaming telemetry data.  Although latency constraints are not as strict as
for control loops, sensor data need to preserve periodicity ({{ocn-intervals}}),
thus could use DetNet service support.

Leveraging DetNet could eliminate split traffic flows by collecting the
sensor data by the applications. This also allows  controllers to be run and
operated from the cloud platforms where much more powerful compute capabilities
and available.

### Provisioning for variety of Traffic flows {#prov}

Different operational scenarios have different constraints; even commands
within the same application will have different time requirements.

  - Different types of latency bounds will be required between a controller and
    an actuator pair based on the type of end-equipment and precision
    requirements. Out-of-order message processing may lead to failures and shutdown
    of operations.  Messages may also be correlated. Therefore, time constraints
    may be applied on a single message or on a group of messages.


  - Similarly, each sensor-controller pair may come with its own interval
    requirement. Sensors emit data at regular interval but this type of
    information may not always be time-constrained. The gaps between the period
    can provide an indication to the controller about communication or other
                                                                       problems.

- Additionally, some faults and alarm messages are urgent reports and must be marked and
  transmitted accordingly.

It is not clear if all these variations can be predictably resolved without any
additional information offered to the DetNet forwarding plane. For example, if
two independent OCN flowlets (that is, ordered group of packets that are related at
process control logic) with variable bounded latency are classified to the same
DetNet flow, they will receive the same treatment, regardless if one has the
shorter latency than the other and may end up behind a flowlet with longer
latency value. On the other hand, if an OCN flowlet have packets with different
latency values, they could end up in different DetNet flow and may not reach
destination in a specific order.


### Security {#sec}

Industrial control networks also have split security boundaries. They have been
designed to be air-gapped or secure by separation.  This is not ideal for
remote operations and control. Current systems deploy strict admission control
policies on both ingress and egress directions.
 
With the growing volume of traffic in control networks, the border gateways and
firewalls will need to incorporate a large number of flow rules; this can be more
prone to errors related to provisioning churns, especially if the system is
dynamic or continuously changing.
 
Application flows can be protected at the network layer as described in the
[RFC9055] Section 10. In case applications provide additional data (metadata)
to the network layer, the integrity of metadata has to be protected from  the
application endpoint to the DetNet edge

## Summary of Gaps

 - Application view ({{app}}: An OCN application is unaware of how DetNet services are
   provisioned. A common UNI between the applications and DetNet-enabled
network needs to be added to the current framework to better map the
expectations better.

 - Security ({{sec}}): of process control related metadata to be used by network
   must be secured.
 - Traffic behavior ({{prov}} and {{class}}): Within the same DetNet flow, classified via
   6-tuple, additional information/metadata must be supported so that dynamic
   traffic patterns can be scheduled deterministically.
 - Split traffic ({{split}}): Leveraging DetNet should eliminate split traffic
   flows by direct collection of sensor data by the applications. This also
   allows  controllers to be run and operated from the cloud platforms where much
   more powerful compute capabilities are available.

# DetNet Potential Approach {#approaches}

Remote process automation presents different types of traffic profiles and to
deal with them within the DetNet framework, we discuss few possibilities.

The DetNet UNI will enable applications to convey specific requirements to
DetNet-aware Network. Note that it is just an interface and is blind to the
internal implementation of such networks.

The DetNet architecture does not describe how DetNet-aware nodes can design DetNet sub-layers.
 But even from the view of an end system, the separation between
forwarding and service sublayer functions should be maintained. This means, the DSCP should not be
overloaded and DetNet-IP forwarding layer should be extended.

## Application association to Forwarding sublayer

Applications should convey specific resource requirements to the DetNets they
connect to. There are two potential options: (a) The DetNet Relay-node performs
translation and binding to one of the DetNet services in the DetNet; or (b) or carry
the application-defined data over DetNet as is and enables processing on transit nodes.

## Encapsulation

OCN applications are expected to be IP based end stations. (MPLS DetNet will
not apply). It is also reasonable to assume that the applications are IPv6 capable, therefore, Ipv6 extension headers can be used to 
request network services inband. With an IPv4 base data plane, the encapsulations could potentially be over UDP, however, that is not the focus of this document. This document specifically deals with HBH IP6 extension headers mechanisms to interface with a Deterministic Network.

The end system network requirement is expressed as 'OCN flow QoS'.
Each packet carries its own unique OCN-QoS. The metadata to be transmitted to DetNet are:

      - Async traffic with latency information.
      - Sync, periodic traffic
      - urgency of messages
      - Flowlet identification (for related packets).

This can be implemented using the HBH extension header option.

## Operation and Control Network Option (OCNO)

   The OCN Option (OCNO) is a hop-by-hop option that can
   be included in IPv6 for OCN traffic control extensions.

~~~~drawing

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
                                   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                                   |  Option Type  |  Opt Data Len |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | OCNF flags     |   OCN-TC-Flowlet nonce       |  sequence     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                (bounded latency spec)                         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                (Delay variation spec)                         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~
{: #ocn-detnet title="Explicit Traffic Control HBH Options"}

{: vspace="0"}
  Option Type:
  :   8-bit identifier of the type of option.  The option identifier
   for the OCN Option (0x??) to be allocated by the IANA. First two bits
  will be 00 (skip over this option and continue processing the header.)

  Option Length:
  :  8-bit unsigned integer. Multiple of 8-octets.

  OCN Function Flags:
  : Some flags require metadata, while others don't.  So process flags in order, if the
flag is off,  the following metadata will not be present.

     | Flag | Description                      |
     |------:+-------------------------------- |
     |   U  | send the message immediately. its an alarm  |
     |   P  | periodic packet (intervals in ~ms)      |
     |   F  | part of flowlet. see Nonce and seq  |
     |   L  | bounded latency spec provided    |
     |   R  | Reliability with no packet loss tolerance |
     |   V  | Delay variation with no packet loss tolerance |

{: #ocn-flags title="OCN Flags to indicate DetNet Functions"}

Flowlet nonce:
:    16-bit. identifies that a packet is associated to a group of packets and shares fate.
      as an example, an application can set the same nonce for a set of an actuator and sensors.
     when set to 0, flow id is set to the same value in related flows. when flow id is also 0, no
relationship exists.

Flowlet sequence:
:    8-bit. sequence to be used for ordering within flowlets.

Bound Latency Spec:
:    32-bit. Encodings, to be defined.\\
     16-bit (upper bound), 16-bit (lower-bound). This field will provide upper and lower
     latency bounds describing the the latency bounds in milliseconds corresponding
     to the packet.

Delay Variation Spec:
:     16-bit. for synchronous stream, delay variation tolerance in ms.

## OCNO Operation

~~~~drawing

   OCN
 Controller         Ingress Relay        Egress Relay      OCN
+----------+             Node                Node        fld-device
|   Appl.  |          <----------DetNet-Service ------>   +-----+
+----------+        ............           ...........    |Appl.|
| OCNO-EH  :--UNI-->: Service  :-- DetNet -: Service  :   +-----+
+----------+        +----------+           +----------+   |IPv6 |
| Ipv6     |        |Forwarding|           |Forwarding|   +-----+
+--------.-+        +---.------+           +----------+
    :   : OCN scope    :                         :
    :   +..............+                         :
    :--------------------------------------------:
              extended scope
~~~~~
{: #ocn-interface title="An interface from controller application to DetNet"}


## OCNO Extension Header Signaling

The current definition of OCNO only covers applications to DetNet unidirectional interface. It is possible to extend OCNO EH for conveying errors from DetNet to the controller as well - for example, when a service violation happened in the DetNet, the Relay node will set an error flag in OCNO EH. Field devices are considered resource constrained and are not expected to insert or process extension headers.
Due to flow aggregation, it is upto the DetNet operator to design mapping between an application and the DetNet.

Two different options of carrying hop-by-hop options are considered.

 1. EH is inserted by the application and the Relay node performs mapping to DetNet flow.
 2. if the DetNet data plane is IPv6, then EH can be carried all the way to the last Relay node, which
    acts as a gateway for the fld device and performs EH-specific functions.

# IANA Considerations

To request an option code.

# Security Considerations

See the section on security above.

# Acknowledgements

--- back

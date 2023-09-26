---
abbrev: ocn-in-detnets
docname: draft-km-detnet-for-ocn-latest
title: Using Deterministic Networks for Industrial Operations and Control
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
    ins: R. Li
    name: Richard Li
    org: Futurewei
    email: richard.li@futurewei.com
-
    ins: C. Westphal
    name: Cedric Westphal
    org: Futurewei
    email: cedric.westphal@futurewei.com
-
    ins: L. Contreras
    name: Luis M. Contreras
    org: Telefonica
    email: luismiguel.contrerasmurillo@telefonica.com
-
    ins: T. Faisal
    name:  Tooba Faisal
    organization: King's College London
    email: tooba.hashmi@gmail.com

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

Process automation systems involve operating equipment (such as actuating
and/or sensing field devices). The communication between the 'process
controllers' and field devices exhibit a well-defined set of behaviors and has
specific characteristics: delivering a control-command to a machine must be
executed within the time frame specified by a controller or an application to
provide reliable and secure operation. A low or zero tolerance to latency and
packet losses (among other things) is implied.

The endpoints ('process controllers' and field devices) embody
machine-to-machine communications to facilitate remote and local process
automation. In this document, networks that support all the characteristics of
remote process automation are referred to as Operation and Control Networks
(OCNs) for convenience. This document describes using DetNet to enable OCN
applications since they provide mechanisms for guaranteed delay-aware packet
delivery, reliability, and packet loss mitigation.

This document defines the interface between an OCN application and the DetNet
framework. i.e., using DetNet services for communication between the
controllers and the field devices. This interface is used by an application to
express its network-specific requirements. This document presents the
perspective of an end system. Because general-purpose applications widely use
IP network stack and provide more connection flexibility to end systems, the
scope of our discussion is specific to the IP-enabled DetNet data planes
{{!DETNET-DP=RFC8655}}. A proxy function is assumed for the other type of field
devices and service levels (section 4.1 in RFC8655).

Mapping OCNs to DetNet helps better understand how DetNets can be used in such
scenarios. The document provides a background on the type of traffic patterns
in OCN applications. It proposes an interface between an application and DetNet
and a potential solution direction to support OCN traffic patterns over DetNet.

# Terminology

- Operational Technology (OT):
 : Programmable systems or devices that interact
with the physical environment (or manage devices that interact with the physical
environment). These systems/devices detect or cause a direct change through the
monitoring and/or control of devices, processes, and events. Examples include
industrial control systems, building management systems, fire control systems,
and physical access control mechanisms. Source: {{NIST-OT}}

- Industrial controller or process controller:
  : Is a logic control function used in process automation and control systems.
A process controller maintains the operational requirement of a process and
performs functions similar to programmable logic controllers (PLCs) but it can
be either a hardware or software component. The term process controller is used
through out to avoid confusion with 'network controllers' used in network
infrastructures.

- Industrial Automation:
  : Mechanisms that enable machine-to-machine communication by use of
technologies that enable automatic control and operation of industrial devices
and processes leading to minimizing human intervention.

- Control Loop:
 : Control loops are part of process control systems in which
desired process response is provided as input to the 'process controller', which
performs the corresponding action (using actuators) and reads the output values.
Since no error correction is performed, these are called open control loops.

- Feedback Control Loop:
  : A feedback loop is part of a system in which some portion (or all) of
the system's output is used as input for future operations.

- Industrial Control Networks:
 : Industrial control networks are the
interconnection of equipment used for the operation, control, or monitoring of
machines in the industrial environment. It involves a different level of
communication - between fieldbus devices, digital controllers, and software
applications.

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
the Programmable Logic Controllers (PLCs) or other types of controllers (Note:
in this memo term 'process controller' will be used to differentiate
from other meanings of controller) using serial bus technologies (and now
Ethernet).  Above those 'process controllers' are Human Machine Interface (HMI)
connecting different PLCs and performing several controller functions along
with exchanging data with the applications.

A factory floor is divided into cell sites. The PLCs or other types of
controllers are physically located close to the equipment in the cell sites.
Monitoring, status, and sensing data are collected on the site
and then transmitted over secure channels to the data applications for
aggregation and further processing. These applications can be hosted
in remote cloud infrastructure but are often hosted within a
limited domain environment, controlled by a single operator, like
on-premise, at the edge, or in a private cloud. Both options gain
from infrastructure that scales out and has elastic computing and storage
resources so they will be referred to as cloud in the following sections.

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

Data applications can integrate softwarized process control
functions to improve automation and make programmatic real-time decisions. The
equipment control and collection of data generated by the sensors should be
possible over small or large-scale deterministic networks as illustrated in
{{new-arch}}.

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

One particular motivation is to provide the behavior of a field bus between the
cloud and the actuators/sensors. i.e., with the same assurance of reliability
and latency, albeit over wide area networks (WAN). Many industrial control
applications, such as factory automation {{FACTORY}}, PLC virtualization
{{VIRT-PLC}}, power grid operations {{PTP-GRID}}, etc.,  are now expected to
operate in the cloud by leveraging virtualization and shared infrastructure
wherever possible.

## Connected Process-Controllers, Sensors and Actuators

{::comment}
## Reference Points for Connecting Controllers, Sensors and Actuators
{:/comment}

Control systems comprise 'process controllers', Sensors and Actuators. The data
traffic essentially carries instructions that cause machines or equipment
to move and do things within or at a specific time. The connectivity exists in
the following manner:

- A 'process controller' interfaces with the sensors and actuators. It knows an
application's performance parameters which are expressed in terms of network
specific requests or resources such as tolerance to packet loss, latency limits,
jitter variance, bandwidth, and specification for safety.  The 'process controller' knows
all the packet delivery constraints.

- An actuator receives specific commands from the 'process controllers'. The
  Deterministic network between them should support control of actuating
devices remotely from the 'process controller' while meeting all the
requirements (or key performance indicators - KPIs) necessary for successful
command execution. The actuator participates in a closed control loop as needed.

- A sensor emit periodic sensor data. It may intermittently provide
asynchronous readings upon request from the 'process controller'. Sensors may report
urgent messages regarding malfunctioning in certain equipment, cell sites, or
zones.

In many control systems, there is at least one 'process controller' (or server) entity on
one end and two other entities - the sensors and actuators on the other end.
The communication with sensors and actuators is through a 'process controller' application;
as such data applications do not directly interact with the field devices.
Neither actuators nor sensors perform decision-making tasks. This
responsibility belongs to the 'process controller'.

## Generalized Communication Model

To describe networked process control behavior, a conceptual communication model
is used so that the data applications do not concern with the details of the
networks realizing operations and control. We refer to this model as an operation
and control network (OCN) model, with the following components:

- Logical reference points: identify an endpoint's role or function as
  sensor-point, actuation-point, or operation & control point (oc-point for
short). Note: the term 'oc-point' is used to avoid confusion with the network
controllers and the term 'fd-point' is used when both types of field devices are
referred to.

- Interface specification: in terms of associated traffic patterns between the
  endpoints as described below in {{ocn-pattern}}. The interface may be any
type of network (Ethernet, IP, wireless, etc. The model assumes that the
network is capable of providing network services and resources necessary of the
application specific operations and control.

Depending on the design of the usecase, the 'process controller' functionality
(oc-point) may reside as a software module in the data application or as a
separate module. When deployed as a separate module, another connectivity
the interface between the data application and oc-point will be needed and is out
of the scope of this document.

The applications will use a communication interface between oc-point and
sensor-point to receive sensory data and similarly interface between oc-point
to actuation point to execute a single or a sequence of control instructions.

This abstraction provides an additional layer of  protection in the sense that
the traffic patterns between the reference points are well defined so any
exceptions can be easily caught.

## Traffic Patterns {#ocn-pattern}

For either local or wide areas, the process automation activities over the
network can generate a variety of traffic patterns between the oc-point and
field devices such as:

### Control Loops {#c-loop}

The equipment being operated upon is sensitive to when a command request
actually executes. An actuator, upon receiving a command (say a function code) will
immediately perform the corresponding action.
{::comment}
It is the responsibility of network
and 'process controller' to ensure that behavior of the sensor and actuator follows the
expectations of applications.
{:/comment}

For several such applications, the knowledge of a successful operation is equally
critical to advance to the next steps; therefore, getting the response back in
a specified time is required, leading to a knowledge of timing. These types of
bounded-time request and response mechanisms are called control loops.

Unlike general-purpose applications, commands cannot be batched; the
parameters of the command that will follow depends on the result of the previous one.
Each request in the control loop takes up a minimal payload size (function code,
value, device or bus address) and will often fit in a single short packet.

In Detnet-enabled network, it can be imagined as a small series of packets with
the same flow identifier, but with different latency constraints.

It is required to support control loops where each request presents its own
latency constraints to the network and where commands are small sized packets.

###  Periodicity {#ocn-intervals}

Sensors emit data at regular intervals; i.e., there may be more tolerance to
variations in jitter between the measurement intervals. Usually, 'process
controllers' or applications listening to sensor data are programmed to
tolerate and record intermittent losses or delay variations upto certain number
of times. Therefore, time criticality is not always high.

Notably, industrial software now increasingly rely on sensor data collection to
monitor the state and behavior of the entire shop floor.  Thus, the number of
sensors are growing and the combined traffic volume generated by sensors is
expected to be very high. In fact will contribute to a large percentage of ocn traffic.
Moreover, the periodicity of each sensor will also vary based
on the equipment.

It is required that network capacity is planned appropriately for the periodic
traffic generated from the different sensors. The periodic interval should also
be preserved in the network because any variations could provide false
indications that the equipment is misbehaving.

### Ordering

In real-time process control communications, out-of-order processing of related
messages will lead to costly operations failures.  For example, messages
such as request and reply, or a sequence of commands to different endpoints may
be related in the application work flow, therefore, both time constraints and order must be preserved.

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
humidity, etc. These messages must be delivered immediately and with the utmost urgency.


## Communication Patterns

Control systems follow a specific communication discipline. The field devices
(sensors and actuators) are always controlled, i.e., interact with the system
through 'process controllers' in the following manner:-

- Sensor to 'process controller': data emitted at periodic intervals providing
  status/health of the environment or equipment. The  traffic volume for this
communication is determined by the payload size of each  sensor data and the
interval. These are a kind of synchronous Detnet flows but with much higher time intervals; still the inter-packet gap should be minimal.

- Process controller to/from actuator: the commands/instructions to write or read.
  Actuators generally do not initiate a command unless requested by the
'process controller'. Actuators will often execute a command, read the corresponding
result, and send that in response to the original write command.  The traffic
profile will be balanced in both directions due to requests/ response behavior. These are like asynchronous flows but without the observation interval constraint.

{::comment}
However, the actual volume (or rate) will depend on the capabilities of the
field device (whether it needs to be operated instruction by instruction or can
be pre-programmed).
{:/comment}


# Industrial Control Application Interfaces to DetNets {#gaps}

Note: use which term? process-controller or industrial-controller?

Current industrial automation solutions utilize a split approach.
industrial-controllers are placed close to the equipment to achieve operational
accuracy, whereas actual process instructions are received through other means
possibly involving human interface. Similarly, sensor data is first acquired
on-site then transmitted in bulk to the enterprise cloud or remote site for
further processing. Such approaches lead to increase in IT infrastructure costs
on the shop floors.

This document is developed with the assumption that the deterministic networks
are deployed between enterprise sites and shop floors. They have resources
available to provide latency guarantees, reliability, and link capacity over
known physical distances. Thus, they can be used to deliver process control and
sensor data collection remotely from an application to shop floor machinery
over larger distances or the Wide Area Networks (WAN) thereby reducing the need
for IT infrastructure on shop floors.

## Deterministic Networks Relevance {#detnet-rel}

> Note: This section's text and explanation on DetNet can be removed.

DetNet data plane framework {{!RFC8939}} describes the DetNet IP encapsulation
into two sublayers as shown in {{fig:detnet-arch}}. The forwarding sub-layer
allocates resources to ensure low loss, latency, and in-order delivery. In
contrast, the service sub-layer manages packet replication, sequence numbering,
and related functions. Together, these sublayers are described as DetNet flows,
which serve as the aggregators for multiple application flows (app-flows).

App-flows and DetNet flows are two different constructs. App-flows describe an
end system's traffic; they initiate requests for network resources under an OT
management application. The request for resources by app-flows and their mapping to
DetNet flows are separate functions from the network resource reservations of
DetNet flows. Their specifications are covered by the flow information model
{{!RFC9016}}. Because resource requests by app-flows and allocations by DetNet
systems are provisioned before actual traffic transmission, a high level of
predictability is ensured in DetNets.

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
{: #fig:detnet-arch title="A Simple DetNet-Enabled IP Network, Ref. RFC8939"}


The traffic originating from end systems (the app-flows) is encapsulated within the DetNet flows. This encapsulation occurs at the reference point where the association or mapping between app-flows and DetNet flows is established.
Specifically, in a DetNet unaware end system, the relay node will do the
mapping (also shown in {{fig:detnet-arch}}).

Various other deterministic network technologies exist at lower layers such
as TSN, 5G, and optical. This document only leverages a specific case using
IP as a direct interface between an application and the DetNet since most
enterprise applications use IP stack.  Other options are out of the scope of
this work. The scope is further narrowed for DetNet unaware end systems to
minimize changes to the existing IP-based industrial-controller applications.

Referring to {{fig:detnet-arch}}, an 'industrial-controller' will be one
DetNet endpoint of the application, while field devices are the remote
endpoints. Note the asymmetry between the compute and memory capabilities of
the two types of endpoints, viz. industrial-controller and field-devices.

The legacy field devices are not expected to be DetNet aware. Therefore, will
require their adjacent gateways to take up the DetNet relay node role and
continue to provide associated translation capabilities. Whereas
software-based PLC applications can be DetNet aware nodes but require greater
flexibility than what is currently offered by the flow information model to
support dynamic changes in the process control operations.

## DetNet Considerations {#depend}

The industrial control model has to support different types of traffic profiles
for a substantial number of field devices. Configuration of each app-flow using
{{!RFC9016}} could become a tedious scaling problem as the number of
industrial-controller-to-field-device pairs grow or keep changing.

The current provisioning model poses issues such as:

  * How can an application request the proper network resource for each
    command?
  * How can an application receive periodic sensor data, and with what
    interval?
  * What are the ways to differentiate less sensitive (periodic) updates from
    urgent alarms?
  * Or how to differentiate data received from a sensor vs. an actuator (with
    stringent latency requirements) and process them accordingly?

These issues and considerations are described below in more detail.

### Operator vs Application view {#app}

The DetNet is primarily designed with a network-operator-centric approach. The
operator's view on dealing with large-scale networks is being discussed in
{{!I-D.ietf-detnet-scaling-requirements}}. DetNet relies on flow aggregation to
use resources efficiently. The integrated OT and IT networks will require
simpler network provisioning at least from an application's perspective;
preferably, a toolset or an Application Programming Interface (API) to dispatch
their requests to the edge of the Deterministic networks.

### Flow reservation and classification {#class}

A single OCN application may require different resource requirements for each
controller-field-device (ctrl-flddev) pairs, and will potentially interface
with multiple field devices.

These variations are easier to achieve with a signaling or user-to-network
interface between the applications and DetNet. Embedding requirements
explicitly can also help DetNet edges to make more dynamic decisions as against
static mappings between app-flows ro DetNet-flows.
an otherwise link that can be congested when used with non-deterministic
flows.

### Split Traffic flows {#split}

A natural consequence of deploying with ICA-95 security architecture in
industrial control systems is that data from the sensors is collected on-site
and often aggregated before being transported to the cloud. For remote process
control, this approach does not apply anymore.  Due to growth in sensor data, it
now requires a much larger on-site storage infrastructure which is expensive.
Applications also expect real-time streaming telemetry data. Although latency
constraints are not as strict as for control loops, sensor data need to
preserve periodicity ({{ocn-intervals}}), thus could use DetNet service
support.

Leveraging DetNet could eliminate split traffic flows by collecting
the sensor data by the applications.  This also allows industrial
controllers to run and operate from cloud platforms with
much more powerful computing capabilities.


### Provisioning for a variety of Traffic flows {#prov}

Different operational scenarios have other constraints; even commands
within the same application will have different time requirements.

  - Different types of latency bounds will be required between a 'process controller' and
    an actuator pair based on the type of end-equipment and precision
    requirements. Out-of-order message processing may lead to failures and shutdown
    of operations.  Messages may also be correlated. Therefore, time constraints
    may be applied to a single message or on a group of messages.

  - Similarly, each sensor-controller pair may come with its own interval
    requirement. Sensors emit data at regular intervals but this type of
    information may not always be time-constrained. The gaps between the period
    can provide an indication to the controller about communication or other problems.

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
the destination in a specific order.


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
{{!RFC9055}} Section 10. In case applications provide additional data (metadata)
to the network layer, the integrity of metadata has to be protected from  the
application endpoint to the DetNet edge

## Summary of Gaps

 - Application view ({{app}}): An OCN application is unaware of how DetNet services are
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

# Operation & Control Header Option {#approaches}

An interface from application to network using IPv6 operation and control
Extension header (EH) option is proposed as means for app-flow to express
network resources with a fine granularity. Other options as YANG based
provisioning do not scale, nor are easy to change dnamically. Since
applications generating app-flows use IP, an IPv6 EH option provide are a more
natural fit than other encapsulations and is specifically suitable for DetNet
unaware end systems.

## System Behavior

Executing remote process automation within the DetNet framework, requires a
management application to interface with the DetNet controller for initial
resource-pool provisioning shown as 'MGMT' in {{fig:detnet-ind}}.

This management application understands the capabilities of endsystems
(industrial-controllers, field-device gateways) under it's control. It requests
aggregated resource requests to the DetNet-controller. These reservations could
be per source and destination address pairs and many app-flows between them.

The out-of-band flow of provisioning happens in the following steps:

{:req: counter="bar" style="format (%d)"}

{:req}
* A management application or centralized user controller ('MGMT') is
  responsible for the initial network resource setup with network service
  provider entities (e.g. with the controller as explained in
  {{!I-D.ietf-detnet-controller-plane-framework}} Section 3.2). It identifies
  the amount and types of resources needed by the applications. This can
  potentially follow existing DetNet YANG models or proprietary approaches.
* A network controller allocates/provisions and maps those requests to DetNet
  flows. It is sufficient to return the results of success or
  failure of reservations to the MGMT function (no explicit mappings).
* All the endsystems from then onwards should operate with in the bounds
  of resources allocated.
* Applications and relay nodes could employ additional monitoring mechanisms to
  keep overall system within the bounds and prevent failures in deterministic
  operations. MGMT function also mangages updates to network-provider about any
  changes to the resource between source/destination leads to updates.
* An application such as software-based industrial controller can now send
  traffic with more specific resource requests using {{ocno}} format.



As shown in {{fig:detnet-ind}}, this management interface is bidirectional to
receive success and failure of the reservations.

~~~~~
DetNet
End System
   _
 / PC\     +-----+      +-----------+            DetNet
| App |<-->|MGMT |<====>|DETNET-CTRL|          End System
/-----\    +-----+      +---+-------+          +------+
| NIC |                /   |       \           |FD-GW |
+--+--+ De|tNet       /    |        \          +----+-+
   |       UN|I+----+    +----+       +----+ DetNet |
   |      v    |    |    |    |-+     | PE |  UNI(U)|
   +-----------U PE +----+ P  | |     |    U--------+
               |    |    |    | |-----|    |
               +----+    +--+-+ |     +----+
                            +---+
             |<------DetNet ----------->|

    PC APP: Process Controller Application
    FD-GW:  Field device gateway
    NSP entity: Network service provider controller
                e,g, DetNet Controller
~~~~~
{: #fig:detnet-ind title="A Realistic DetNet Based Industrial Application Network"}

## Scope and Limits (goals and non goals)

The proposed OCN-EH solution is a generic interface to the DetNets from OT
applications with a programmable and dynamic process automation capabilities.
Once the high-level reservation of resources is done, DetNet should process
the incoming traffic with OCN-EH with in its capabilities.

The following are the non-goals:

   -   To provide support for stringent periodic traffic schedules:
       DetNets support both asynchronous (by allocating resources for the
       observation interval) and synchronous flow behaviors (Section 4.3.2 in
       {{!DETNET-DP=RFC8655}}). OCN- EH option for extremely sensitive
       periodicity are not explicitly explored, a control plane provisioning
       may be sufficient. Intervals are supported for sensors, emitting
       periodic data.

   -   To change field device behavior: OCN-EH solution does not expect
       changes to field-devices. It depends on their gateways to
       terminate DetNet flows and perform fieldbus protocol translations.

   -   To provide mapping procedures: Explicit procedures for mappings and how
       they are
       performed, updated  on edge nodes are not discussed since they are
       proprietory or specific to NSP domain.

 Main goals:

   -   To provide a programmable and extensible interface:
       OCN applications are IP end stations. (MPLS DetNet will
       not apply). It is reasonable to assume that the applications are IPv6
       capable; therefore, Ipv6 extension headers can be used to request network
       services inband. With an IPv4 data plane, the encapsulations could
       be over UDP; however, that is not the focus.

   -   Application to receive errors or feedback from the network:
       A signaling from the relay node to the end system can help
       measure application performance.

## Types of App-flow Requests

The end system network requirement is expressed as 'OCN flow QoS'.
Each packet carries its own unique OCN-QoS. The metadata to be transmitted to
DetNet are:

    -  Async traffic with latency information.
    -  Sync, periodic traffic
    -  Urgency of messages
    -  Flowlet identification (for related packets).

This can be implemented using the HBH extension header option.

## Operation and Control Network Option (OCNO) {#ocno}

   The OCN Option (OCNO) is a hop-by-hop option that can
   be included in IPv6 for OCN traffic control specification.

~~~~drawing

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
                                   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                                   |  Option Type  |  Opt Data Len |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | OCNF flags                    |   OCN-TC-Flowlet nonce        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | sequence       |        (bounded latency spec)                |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                (Delay variation spec)                         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                (Result spec)  |                               |
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
  : Some flags require metadata, while others don't.  Flags are processed
    in order from high to low order bits (left to right, from U to R), if the
    flag is off,  the corresponding metadata will not be present.

Flowlet nonce:
:     16-bit. Identifies that a packet is associated with a group of
      packets and shares fate. For example, an application can set the
      same nonce for a set of actuators and sensors. When set to 0,
      flow-id is set to the same value in related flows. When flow-id is
      also 0, no relationship exists.

Flowlet sequence:
:    8-bit. Sequence to be used for ordering within flowlets.

     | Flag | Description                      |
     |------:+-------------------------------- |
     | U  | Urgent. message to be sent immediately. An alarm (no-metadata)  |
     | I  | the flow is part of periodic packet (look for interval in ~ms) |
     |  F | part of flowlet. see Nonce and seq  |
     |  L | bounded latency spec provided    |
     |  P | Reliability with no packet loss, this flag can be used by DetNet for selecting in-network reliability techniques. |
     |  V | Delay variation with no packet loss tolerance |
     |  R | Reply  packet to a command identified by flowlet |

{:#ocn-flags title="OCN Flags to indicate DetNet Functions"}

Bound Latency Spec:
:    32-bit. Encodings, to be defined.\\
     16-bit (upper bound), 16-bit (lower bound). This field will provide upper and lower
     latency bounds describing the latency bounds in milliseconds corresponding
     to the packet.

Delay Variation Spec:
:     16-bit. for a synchronous stream, delay variation tolerance in ms.

Interval Spec:
:     16-bit interval field. TBD.

Reply Spec:
:     16-bit results of network service delivery. TBD.


## OCNO Operation and Signaling

~~~~drawing

   OCN
 Controller         Ingress Relay        Egress Relay      OCN
+----------+             Node                Node        fld-device
|   Appl.  |        <------------DetNet-Service ------>   +--------+
+----------+                                              |Cmd/Res.|
| OCNO-EH  :--UNI-->+----------<<  DetNet >>              +--------+
+----------+        |          |           +----------+   | FBUS   |
| Ipv6     |        |Forwarding|           |Forwarding|---+--------+
+--------.-+        +---.------+           +----------+       |
    :   : OCN scope    :                                      |
    :   +..............+                   +--------+         |
    :--------------------------------------| DATA   |---------+
              extended ocn scope           +--------+
                                           |OCNO-EH |
                                           +--------+
                                           | Ipv6   |
                                           +--------+

~~~~~
{: #ocn-interface title="An interface from 'process-controller' to DetNet"}

The workflow of traffic with EH option happens in the following steps:

1. An end system (industrial controller)  uses the format described in
  {{ocno}} to provide ocn-constraints (e.g. network latency limit) or
  delay variation. It fills option type, len fields along with OCN
  flags and sequence if needed.

1. Platform logic related deterministic processing is not part of the
  network latency in EH; Packet is tranmitted on interface connected to
  DetNet relay node.

1. DetNet relay node processes parameters, and source/destination addresses
  associate an app-flow to DetNet flow. It may or may not remove EH
  see {{encap_pre}}, and inserts its own DetNet encapsulation (technology specific).

1. In case of known exceptions or errors, the relay node could reply to application
   with hints (Reply flag set).

1. DetNet delivers the packet with guarantees of network resources
   requested to the endsystem gateway connecting to field devices.

1. Field device gateway performs protocol translation and deliver packet to
   the field device.

1. Observable errors, such as late delivery or inconsistent OCN header can
  be sent to OC App from the gateway.

1. Similarly, gateways insert new OCN headers for messages originating from
   field devices, such as alarms or other sensor data.


## OCNO EH Processing {#encap_pre}

 - OCNO EH  can be extended for conveying errors from DetNet to the industrial controller application. For example, when a service violation
happened in the DetNet, relay node will set an error flag in OCNO EH.
- Field devices are considered resource-constrained and are not expected to insert or process extension headers.

Two different approaches of hop-by-hop options processing are feasible.

 1. EH is inserted by the application. The relay node performs mapping to DetNet flow.
 2. if the DetNet data plane is IPv6 end to end, then EH can be carried and processed on each hop to the last relay node, which
    acts as a gateway for the fld device and performs EH processing.

The document currently assumes only the first option.


# IANA Considerations

To request an option code.

# Security Considerations

See the section on security above.

# Acknowledgements

--- back

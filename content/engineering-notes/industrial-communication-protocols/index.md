+++
title = "When PLCs Need to Talk: The Languages Behind Industrial Automation"
description = "A practical look at industrial communication protocols, IIoT messaging, and how these systems are configured and used within CODESYS."
date = 2026-08-11

[taxonomies]
tags = ["Industrial Automation", "Communication", "CODESYS", "IIoT"]
+++

Industrial automation rarely consists of a PLC working on its own.

A modern control system may have a PLC communicating with remote I/O, sensors, drives, HMIs, gateways and cloud services at the same time.

The interesting part is that these devices do not all speak the same language.

---

## The PLC is only one part of the conversation

A typical automation system can contain several communication layers.

At the machine level, a PLC may communicate with sensors, actuators and remote I/O.

At the control level, it may exchange data with drives, HMIs and other controllers.

Further up the system, that same controller may publish process data to another system using an IIoT protocol such as MQTT.

The choice of protocol therefore depends heavily on what the devices need to communicate and where they sit within the system.

---

## Industrial protocols

Industrial communication protocols are designed around the requirements of automation systems.

Unlike general-purpose networking, industrial protocols often need to provide predictable data exchange, device diagnostics, defined data structures and reliable communication between controllers and field devices.

Some of the protocols commonly encountered in PLC-based systems include CAN-based communication, CANopen, Modbus and IO-Link.

They are not interchangeable. Each solves a different communication problem.

---

## CAN and CANopen

CAN, or Controller Area Network, is widely used in industrial and mobile machinery because it provides a robust communication bus between controllers and field devices.

In a PLC application, CAN can be used to communicate with distributed I/O modules, sensors, motor controllers and other intelligent devices.

CANopen builds a higher-level communication structure on top of CAN.

Instead of simply exchanging raw CAN frames, CANopen defines concepts such as node IDs, object dictionaries, process data objects and service data objects.

This makes device integration more structured.

In CODESYS, a CANopen device can therefore be represented within the device tree and configured using the information supplied by the device manufacturer.

The practical workflow is often:

```text
Device EDS
     ↓
CODESYS Device Configuration
     ↓
Node ID / Communication Parameters
     ↓
PDO / SDO Mapping
     ↓
PLC Application
```

The important point is that communication configuration and application logic are closely related, but they are not the same thing.

The communication layer provides the data.

The PLC application decides what that data means and how the machine should respond.

---

## IO-Link

IO-Link is another example of a communication technology used closer to the field device.

It is particularly useful for intelligent sensors and actuators because it provides more information than a simple discrete or analogue signal.

A traditional sensor may provide only a switching signal or a 4–20 mA measurement.

An IO-Link device can additionally provide diagnostic information, device identification, parameters and process data.

A typical arrangement can look like:

```text
Sensor
   ↓
IO-Link Master
   ↓
Industrial Network
   ↓
PLC
```

This creates a useful separation between the sensor and the controller.

The IO-Link master handles communication with the field devices, while the PLC receives the resulting process and diagnostic information.

For a PLC programmer, this means that the application can use both the measured process value and the diagnostic state of the device.

That can make fault handling considerably more informative.

---

## Modbus

Modbus is another widely encountered industrial protocol.

It is commonly used where a controller needs to exchange registers or coils with another device such as a drive, meter, pump, power monitor or instrumentation device.

Modbus can operate over different physical and network layers, with Modbus RTU commonly used over serial communication and Modbus TCP operating over Ethernet.

The basic idea is straightforward:

```text
PLC
 ↓
Modbus Request
 ↓
Device Registers
 ↓
Modbus Response
 ↓
PLC
```

The challenge is often not the protocol itself, but understanding the register map provided by the device manufacturer.

The PLC programmer needs to know which register contains the required value, what data type it uses, how the value is scaled and whether the register is read-only or writable.

A communication link can therefore be healthy while the application still interprets the data incorrectly.

---

## Communication is more than connecting a cable

One of the easiest mistakes when working with industrial communication is to treat a successful connection as proof that the system is working correctly.

There are several layers to consider:

```text
Physical Connection
       ↓
Network / Bus Communication
       ↓
Device Configuration
       ↓
Data Mapping
       ↓
Data Interpretation
       ↓
Application Logic
```

A device may be online while the wrong data is being mapped.

A register may be read successfully while its scaling is incorrect.

A CANopen node may be visible while the required PDO has not been configured correctly.

Troubleshooting therefore requires looking at the complete communication chain.

---

## Communication in CODESYS

CODESYS provides a structured environment for configuring and programming many of these communication systems.

The device tree represents the hardware and communication architecture, while the application program uses the resulting I/O and communication variables.

This allows the programmer to separate hardware configuration from application logic.

For example:

```text
CODESYS Device Tree
        ↓
Communication Configuration
        ↓
I/O / Communication Variables
        ↓
GVL / Structures
        ↓
Control Logic
        ↓
HMI / Diagnostics
```

A well-structured application should make this boundary clear.

The application should not need to repeatedly deal with low-level communication details when a meaningful process variable can be provided by a dedicated conversion or communication layer.

---

## From PLC Communication to IIoT

Industrial communication does not stop at the machine controller.

Modern systems increasingly connect PLCs to higher-level monitoring platforms, databases and cloud services.

This is where protocols such as MQTT become useful.

MQTT follows a publish/subscribe model rather than the traditional request/response approach used by many industrial protocols.

A typical architecture might be:

```text
PLC
 ↓
Industrial Router / Gateway
 ↓
MQTT Broker
 ↓
Cloud / Monitoring Platform
```

The PLC can publish information such as:

- Process values
- Equipment states
- Alarm states
- Production information
- System health
- Communication status

The receiving system can then subscribe to the required topics.

This provides a bridge between the control system and an IIoT platform without making the cloud system part of the machine's core control loop.

---

## Keep control and monitoring separate

One of the important design principles when connecting PLCs to external systems is to keep the control system independent from the monitoring system.

The PLC should continue performing its local control functions even if the external network or cloud service becomes unavailable.

For example:

```text
                  +------------------+
                  |  Local Control   |
                  |      PLC         |
                  +--------+---------+
                           |
                    Process Control
                           |
                    Machine / Plant
                           |
                           +--------+
                                    |
                              Telemetry
                                    ↓
                             MQTT / IIoT
                                    ↓
                            Remote Platform
```

The communication path used for remote monitoring should therefore not become a dependency for basic machine operation unless the system has specifically been designed that way.

This separation improves resilience and makes the control system easier to reason about.

---

## Choosing the right protocol

There is no single industrial communication protocol that is best for every application.

The correct choice depends on the devices involved and what needs to be exchanged.

A simple way to think about the different layers is:

| Application | Example Technology |
|---|---|
| Sensor communication | IO-Link |
| Distributed machine I/O | CAN / CANopen |
| Device register exchange | Modbus RTU / TCP |
| Controller and network communication | Industrial Ethernet protocols |
| Remote telemetry | MQTT |
| Cloud integration | MQTT / HTTP-based interfaces |

The protocol should be selected based on the actual communication requirement rather than simply using whatever protocol is familiar.

---

## The programmer needs to understand the whole chain

Working with industrial communication eventually becomes less about memorising protocol names and more about understanding how information moves through the system.

A sensor measurement may begin as a physical quantity.

It becomes an electrical signal.

A field device converts that signal into process data.

A communication interface transfers that data to the PLC.

The PLC interprets it and applies control logic.

The HMI presents the result to the operator.

A gateway may then publish selected information to a remote monitoring system.

The complete chain can therefore look like:

```text
Physical Process
      ↓
Sensor
      ↓
Field Communication
      ↓
Remote I/O / Device
      ↓
PLC
      ↓
Control Logic
      ↓
HMI
      ↓
Gateway
      ↓
IIoT / Cloud
```

Understanding this chain is what makes communication troubleshooting much easier.

---

## Final thoughts

Industrial automation is essentially a collection of systems that need to exchange information reliably.

CAN, CANopen, IO-Link, Modbus and MQTT may operate at different levels, but they all serve the same larger purpose: moving useful information between devices and systems.

As PLC applications become more connected, understanding these communication layers becomes just as important as understanding the control logic itself.

For anyone developing applications in CODESYS, communication should therefore be treated as part of the system architecture rather than an isolated configuration task.

The PLC is not working alone.

It is part of a conversation.
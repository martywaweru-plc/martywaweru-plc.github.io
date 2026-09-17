+++
title = "Why Draw Schematics?"
description = "Why system schematics are essential for industrial automation architecture, hardware selection, wiring and PLC programming."
date = 2026-08-14

[taxonomies]
tags = ["System Architecture", "I/Os", "Wiring", "Configuration"]
+++


When working on an industrial automation system, the schematic is more than a drawing of wires. It provides a clear representation of how the system is intended to be built, wired and controlled.

For a programmer, it is also one of the first documents that defines the relationship between the controller, field I/O, HMI, sensors, actuators and communication networks. A good schematic gives the panel builder a clear understanding of the system architecture before the physical installation begins.

---

## Understanding the Available I/O

Industrial controllers and field I/O modules are not simply collections of generic inputs and outputs. Different channels are designed to accept or produce different types of signals.

Common input configurations include:

- **Digital inputs** — ON/OFF signals from switches, sensors and other devices.
- **Analogue inputs** — commonly **4–20 mA** and **0–10 V** signals.
- **Resistance inputs** — used with resistance-based sensors.
- **Frequency inputs** — used where a sensor provides a frequency or pulse signal.

Outputs can similarly have different configurations, including:

- **Digital outputs** — ON/OFF control signals.
- **Analogue outputs** — commonly **0–10 V** or current-based outputs.
- **PWM outputs** — used where the output is controlled through pulse-width modulation.

The exact configuration depends on the controller or I/O module being selected. This is why the manufacturer's device manual has to be used when developing the system architecture.

---

## Hardware Selection Starts With the Architecture

This is where the schematic becomes useful.

Before selecting an I/O module, the programmer and electrical designer need to understand what the system actually requires.

For example, with the IFM mobile automation I/O families, different device series provide different connector and I/O arrangements. A design may call for a **CR2040/40 series** type module when M12 connections are required, while another application may use a **CR2050/50 series** variant with Deutsch-style connectors.

The I/O arrangement also differs between modules.

For example, the architecture may require:

- **CR2040/40 series** — input-focused modules.
- **CR2041/41 or CR2051/51 series** — output-focused modules.
- **CR2042/42 or CR2052/52 series** — modules providing a combination of inputs and outputs.

The exact capabilities and channel configurations must always be confirmed against the relevant device manual.

This means that selecting an I/O module is not simply a matter of choosing one with enough channels. The **type of signal, connector system, electrical characteristics, physical location and communication requirements** all influence the selection.

---

## The Same Applies to Controllers and HMIs

The controller or HMI is another important part of the architecture.

A larger machine may require a PLC, HMI and several distributed I/O modules communicating over CAN. The same network may also include joysticks, keypads, rotary controls, engine controllers and other machine devices.

In an expandable system, a controller or HMI with multiple CAN interfaces and Ethernet connectivity may be more appropriate. Devices such as the IFM **CR1204** or **CR1077** can be considered where the system requires greater connectivity and expansion.

Other systems may already have a main display and controller. In that case, a smaller display such as the **CR1152** may be sufficient for a local function while communicating with the rest of the machine. While for some independent systems a simple non-touch rugged HMI controller such as **CR1140** is enough to do the job.

The point is that **the architecture determines the hardware**, not the other way around.

---

## The Schematic Defines the Signal Flow

Once the hardware has been selected, the schematic becomes the map of the system.

It should show how the different components are connected, including:

- Power distribution.
- Digital inputs and outputs.
- Analogue signals.
- Sensors and actuators.
- CAN connections.
- IO-Link connections.
- Ethernet connections.
- HMI and PLC connections.
- Field I/O modules.
- Communication between controllers and other machine devices.

This gives the panel builder a clear understanding of what needs to be physically connected.

It also gives the programmer a reference for what each device and channel is expected to do.

---

## Physical Wiring Matters

System architecture is not only about what connects to what.

The physical location of the equipment matters as well.

A sensor located at one end of a machine may be connected to a remote I/O module rather than being wired directly back to the main controller. A valve bank may be located close to a distributed output module. An HMI may be mounted in the operator cabin while the main controller is installed elsewhere.

The schematic should therefore represent the intended physical architecture and signal paths.

This becomes particularly important during commissioning.

A correctly designed system should allow the commissioning engineer to look at a physical device, identify its corresponding connection on the schematic and understand where that signal ultimately terminates in the control system.

---

## From Schematic to CODESYS

The schematic eventually becomes part of the programming process.

Once the physical architecture is known, the programmer can develop the corresponding communication and mapping software in CODESYS.

For a CAN-based system, this may involve:

- Configuring the CAN interface.
- Instantiating the required CAN manager.
- Assigning node IDs.
- Configuring the connected devices.
- Mapping digital and analogue signals.
- Packing and unpacking data.
- Assigning mapped signals to GVLs and application variables.
- Monitoring communication status.

Having the correct architecture beforehand makes this process considerably easier because the programmer already knows **which device is connected, which channel is being used and what signal is expected on that channel**.

---

## Communication Monitoring

The architecture can also determine how communication diagnostics are implemented.

For example, a programmer can monitor CAN or Ethernet communication using a periodic heartbeat, blink or pulse mechanism.

If the expected communication signal stops changing, the application can identify a communication failure and generate an alarm condition.

That alarm can then be handled by the application's alarm-handling system.

This is where the schematic and software architecture start to meet: the physical communication path defined during system design becomes something that can be monitored and diagnosed in the PLC application.

---

## Node IDs and Device Configuration

In a CANopen-based system, the physical devices also have logical identities on the network.

The programmer therefore needs to know which devices are present and which node IDs have been assigned to them.

The schematic provides another useful reference for this.

```text
CAN Network
│
├── PLC / CAN Manager
│
├── HMI
│
├── Remote I/O Module
│   └── Node ID: XX
│
├── Joystick
│   └── Node ID: XX
│
└── Engine Controller
    └── Node ID: XX
```

This information can then be reflected in the CODESYS CAN configuration and application mapping.

---

## CAN Is Not Just One Protocol

It is also important to distinguish between **CAN** and the higher-level protocols that operate over CAN.

CAN is the underlying **Controller Area Network** communication technology.

Protocols such as:

- **CANopen**
- **SAE J1939**

define higher-level communication structures and conventions for devices communicating over CAN.

The way data is mapped, addressed and exchanged therefore depends on the protocol being used.

We'll get into that separately when discussing CAN communication and how these protocols are configured and mapped in CODESYS.

---

## So, Why Draw Schematics?

Because the schematic connects the entire thing.

**Hardware selection → electrical architecture → physical wiring → communication architecture → PLC configuration → I/O mapping → application programming → commissioning**

Without a clear architecture, the programmer can end up discovering the system while writing the software.

With a good schematic, the programmer already has a map.

The panel builder knows what to build.  
The programmer knows what to configure.  
The commissioning engineer knows what to test.

And when something doesn't work, everyone has the same document to refer back to.

**That's why we draw schematics.**
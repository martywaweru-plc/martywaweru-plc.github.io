+++
title = "Is Reusable Code OOP in CODESYS?"
description = "Using reusable Function Blocks, structures, arrays and visualisation frames to build modular PLC applications in CODESYS."
date = 2026-08-10

[taxonomies]
tags = ["Practical Design Principle", "Structured Text", "Reusability",]
+++

As PLC applications grow, writing every function independently quickly becomes difficult to maintain. A better approach is to identify repeated behaviour and turn it into reusable software components.

In CODESYS, this can be achieved using **Function Blocks (FBs), structures, arrays, Global Variable Lists (GVLs), and reusable visualisation frames**.

The principle is simple:

> **Develop the logic once, then reuse it wherever the same behaviour is required.**

This reduces duplicated code, improves consistency and makes larger PLC applications easier to develop and maintain.

---

## Reusable Function Blocks

A Function Block is a **Program Organisation Unit (POU)** that contains logic and its own internal state.

This makes Function Blocks particularly useful for functions that need to maintain state between PLC cycles, such as alarms, timers, motor control, sensor processing and machine sequences.

A good example is alarm handling.

Instead of writing separate timer, permissive, latching and reset logic for every alarm, a common alarm handler can be created.

The Function Block can contain the common behaviour:

- Alarm condition
- Permissive
- Activation delay
- Latching behaviour
- Reset
- Alarm state

Conceptually:

```text
Alarm Condition
       |
       v
+-------------------+
|   FB_AlarmHandler |
|                   |
| Condition         |
| Permissive        |
| Delay             |
| Latching          |
| Reset             |
+---------+---------+
          |
          v
     AlarmActive
```

The same Function Block can then be called for every alarm that requires this behaviour.

---

## Function Block vs Function Block Instance

It is important to distinguish between the **Function Block type** and the **Function Block instance**.

The Function Block defines the behaviour:

```text
FB_AlarmHandler
```

An instance is an object created from that Function Block.

For example:

```text
AlarmHandler : FB_AlarmHandler;
```

The instance can then be called with the parameters required for a particular alarm.

When many instances are required, an array can be used:

```text
AlarmHandlers : ARRAY[1..20] OF FB_AlarmHandler;
```

This creates twenty independent Function Block instances using the same underlying logic.

Each instance maintains its own internal state.

For example:

```text
AlarmHandlers[1]
AlarmHandlers[2]
AlarmHandlers[3]
...
AlarmHandlers[20]
```

The implementation does not need to be duplicated.

Only the parameters and conditions change.

This is the key distinction:

**The Function Block is the reusable design.**

**The instance is a particular use of that design.**

---

## Arrays and Reusable Logic

Arrays become particularly useful when many objects share the same behaviour.

For example:

```text
AlarmHandlers : ARRAY[1..20] OF FB_AlarmHandler;
```

Each element represents an individual alarm.

The array can then be processed systematically:

```text
FOR i := 1 TO 20 DO

    IF AlarmHandlers[i].AlarmActive THEN
        AnyAlarmActive := TRUE;
        EXIT;
    END_IF;

END_FOR;
```

Instead of maintaining twenty separate checks, the program can iterate through the collection.

This approach becomes increasingly valuable as the size of the application grows.

---

## Building an Alarm Hierarchy

The alarm system can also use reusable Function Blocks to establish relationships between alarms.

Not every alarm should necessarily be evaluated independently.

Consider a remote I/O module connected to several field sensors.

If the CAN communication with that remote I/O module is lost, the sensor values may no longer be valid.

Generating individual sensor alarms at the same time could therefore produce a large number of secondary alarms.

A better approach is to use the communication status as a **permissive** for the downstream alarms.

```text
Remote I/O Communication
          |
          v
     Communication OK?
          |
       +--+--+
       |     |
      NO    YES
       |     |
       |     +------> Evaluate Field Sensors
       |
       +-----------> Suppress Field Sensor Alarms
```

This creates a simple hierarchy:

```text
System
  |
  +-- Communication
        |
        +-- Remote I/O
              |
              +-- Sensors
                    |
                    +-- Process Conditions
```

The higher-level fault therefore takes priority over faults that depend on the affected communication path.

This is a useful application of **permissive logic** in PLC alarm design.

---

## Startup Alarm Masking

The same alarm architecture can also handle startup conditions.

When a machine starts, controllers, remote I/O modules and sensors may require some time to initialise.

If alarms are evaluated immediately, temporary startup conditions can generate nuisance alarms.

A common solution is to introduce a startup permissive.

```text
PLC Start
   |
   v
Startup Delay
   |
   v
Alarm Evaluation Enabled
```

For example, the complete alarm system can remain masked for the first few seconds after startup.

Once the delay expires, the individual alarm conditions are evaluated normally.

The important point is that the same permissive can be supplied to every alarm instance.

There is no need to implement separate startup masking logic inside every alarm.

---

## Reusable Sensor Processing

The same design principle can be applied to analogue sensor processing.

Industrial sensors commonly provide a **4–20 mA** signal, while the application requires a meaningful engineering value.

For example:

```text
4–20 mA
   |
   v
Raw Input
   |
   v
Scaling / Validation
   |
   v
Engineering Value
```

A reusable sensor Function Block can handle common operations such as:

- Signal validation
- Sensor fault detection
- 4–20 mA scaling
- Engineering unit conversion
- Range checking
- Sensor status

The same FB can then be configured for different types of sensors.

For example:

```text
Flow        -> L/min
Pressure    -> bar
Temperature -> °C
Level       -> %
```

The underlying processing remains the same.

Only the scaling parameters and application-specific configuration change.

This is much more maintainable than creating separate scaling logic for every sensor.

---

## Separating Raw Data from Process Data

Another important part of this approach is separating the communication layer from the application layer.

A typical structure can be:

```text
Sensor / I/O
     |
     v
Raw Communication Data
     |
     v
Sensor Conversion FB
     |
     v
Processed Sensor Data
     |
     v
PLC Application
```

The raw communication data represents what is received from the field device.

The conversion layer interprets that data and produces meaningful process values.

The rest of the application can then work with values such as:

```text
WaterFlow.Value
AirPressure.Value
TankLevel.Value
```

rather than repeatedly dealing with raw analogue values.

This separation also makes the sensor conversion logic reusable.

---

## Structures for Consistent Data

Structures are another important part of reusable PLC design.

A sensor may have several related properties:

```text
Sensor
├── RawValue
├── ProcessValue
├── SensorFailure
├── State
├── Tag
└── Description
```

Instead of managing each property independently, these values can be grouped into a structure.

For example:

```text
ST_Sensor
```

The same structure can then be used for every sensor.

An array can extend this further:

```text
Sensors : ARRAY[1..20] OF ST_Sensor;
```

Now every sensor follows the same data model.

This creates consistency between the different parts of the application.

It also makes it easier to pass groups of related information between POUs.

---

## Reusable Visualisation Frames

The same concept can be extended to the CODESYS visualisation.

Consider a diagnostic page containing information such as:

- Sensor name
- Sensor tag
- Process value
- Engineering unit
- Sensor state
- Fault status

Creating a separate visualisation object for every sensor would result in a large amount of duplicated configuration.

Instead, a reusable **visualisation frame** can be created.

For example:

```text
+-----------------------------+
| Sensor Diagnostic Frame     |
|                             |
| Tag:     AI_001             |
| Value:   125.4 L/min        |
| State:   OK                 |
| Fault:   FALSE              |
+-----------------------------+
```

The frame is designed once.

It can then be placed repeatedly on a visualisation page.

Each instance is supplied with different data.

```text
Diagnostic Page

+------------------+
| Sensor Frame 1   |
+------------------+

+------------------+
| Sensor Frame 2   |
+------------------+

+------------------+
| Sensor Frame 3   |
+------------------+

+------------------+
| Sensor Frame 4   |
+------------------+
```

The visualisation therefore follows the same modular principle as the PLC logic.

---

## Arrays and Visualisation Frames

Arrays can also be used to provide the data for reusable visualisation frames.

For example:

```text
SensorDisplay : ARRAY[1..20] OF ST_Sensor;
```

An initialisation or mapping program can populate the appropriate array elements with the information required by the visualisation.

Conceptually:

```text
SensorDisplay[1]  -> Diagnostic Frame 1
SensorDisplay[2]  -> Diagnostic Frame 2
SensorDisplay[3]  -> Diagnostic Frame 3
...
SensorDisplay[20] -> Diagnostic Frame 20
```

The same visualisation frame can therefore represent different sensors depending on which data element is assigned to it.

This is particularly useful for diagnostic pages containing many similar I/O objects.

Instead of designing twenty different diagnostic objects, one reusable frame can be developed and reused.

---

## Connecting the Pieces

The real benefit comes when these concepts are combined.

A CODESYS application can be structured around reusable layers:

```text
                    PLC Application
                           |
          +----------------+----------------+
          |                |                |
        Control         Processing      Visualisation
          |                |                |
         FBs            Sensor FBs        Frames
          |                |                |
        Arrays          Structures        Arrays
          |                |                |
          +----------------+----------------+
                           |
                          GVL
                           |
                    Process Variables
```

The **GVL** can provide application-level variables that need to be accessible across the program.

The **Function Blocks** perform reusable processing.

The **structures** define consistent data models.

The **arrays** organise multiple instances.

The **visualisation frames** provide reusable HMI components.

Each part has a defined responsibility.

---

## Reusability Is More Than Writing Less Code

The biggest advantage of reusable software is not simply reducing the amount of code.

It is **consistency**.

If twenty alarms use the same alarm handler, the behaviour of all twenty alarms is governed by the same implementation.

If twenty sensors use the same conversion Function Block, they follow the same processing approach.

If twenty diagnostic objects use the same visualisation frame, the interface remains consistent.

When the reusable component needs to be improved, the improvement can be applied to the component rather than maintaining many independent implementations.

This also makes troubleshooting easier.

A new alarm can be added by creating another instance.

A new sensor can be added using the existing sensor processing structure.

A new diagnostic object can use the existing visualisation frame.

The application grows through **configuration and instantiation**, rather than repeated development.

---

## Is This Object-Oriented Programming?

This approach is closely related to **object-oriented programming (OOP)** concepts, particularly **encapsulation, reuse and instances**.

CODESYS supports object-oriented programming features, but reusable Function Blocks do not automatically mean that a PLC application is fully object-oriented.

The important idea here is the design principle:

> **Encapsulate behaviour into reusable components and create multiple instances where the same behaviour is required.**

Function Blocks are especially well suited to this approach because each instance can maintain its own internal state.

For PLC applications, this provides many of the practical benefits of modular and object-oriented design without requiring every application to be structured as a fully object-oriented software system.

---

## A Practical Design Principle

When developing a new piece of PLC logic, it is worth asking:

> **Will I need this behaviour somewhere else?**

If the answer is yes, consider whether it should become a reusable Function Block, structure, array-based component or visualisation frame.

For example:

```text
Repeated Behaviour
        |
        v
Can it be generalised?
        |
       YES
        |
        v
Create Reusable Component
        |
        v
Create Multiple Instances
        |
        v
Configure Each Instance
```

This mindset changes how a PLC application is developed.

Instead of solving each problem independently, the programmer begins building a library of reusable components.

Over time, this can significantly reduce development effort across different machines and projects.

---

## Conclusion

Reusable POU design is an important part of developing maintainable PLC applications in CODESYS.

Function Blocks provide reusable behaviour.

Function Block instances allow the same behaviour to be used independently across different objects.

Arrays allow large numbers of instances to be organised and processed systematically.

Structures provide consistent data models.

GVLs provide shared application data where appropriate.

Visualisation frames extend the same concept into the HMI.

The result is a PLC application that is easier to expand, troubleshoot and maintain.

The objective is not simply to write less code.

The objective is to build software that can be **reused, understood and maintained across different machines and projects**.

For a PLC programmer, reusable design should therefore become a normal part of the development process.

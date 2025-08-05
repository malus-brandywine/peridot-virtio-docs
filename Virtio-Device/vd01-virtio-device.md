# 'Virtio Device' Concept

(**VD01** Ver 1.1. August 2025)

## 1 Definitions

1) **The Spec** is specification "Virtual I/O Device (VIRTIO) Version 1.2"

2) **Driver** or **Device** with the first capital letter refer, respectively,
to Virtio Driver or Virtio Device **side** of the communication. Each side includes the whole stack of components to supply respective Driver's or Device's functionality.

3) **'Device Config Space', or 'Config Space'** in single quotes is a name of the Peridot Virtio component, **Device Config Space** without quotes refers to the Device Configuration Space described in the Spec.

4) **'Virtqueue'** in single quotes is a name of the Peridot Virtio component,
**Virtqueue** without quotes is a type of an object described in the Spec.

5) **'Virtio Device'** in single quotes is a name of the
Peridot Virtio concept (described in the document),
**Virtio Device** without quotes is a virtual device defined in the Spec as
a device "found in virtual environments, yet by design they look like physical devices to the guest within the virtual machine".

<br>

## 2 Virtio Device States


### 2.1 States indicated by *Status* field

Device States known to Driver are defined in the Spec; we can
consider them external - visible outside, but not necessarily presenting
the whole life cycle of a specific Virtio Device.

**Refresher on the Device States (quoted from the Spec):**

|Bit name,<br>or phase name|Bit used<br>in *Status* field|Description|
|:---|:---:|:---|
|ACKNOWLEDGE (1)|0|Indicates that the guest OS has found the device and recognized it as a valid virtio device.|
|DRIVER (2)|1| Indicates that the guest OS knows how to drive the device. Note: There could be a significant (or infinite) delay before setting this bit. For example, under Linux, drivers can be loadable modules.|
|FAILED (128)|7|Indicates that something went wrong in the guest, and it has given up on the device. This could be an internal error, or the driver didn’t like the device for some reason, or even a fatal error during device operation.|
|FEATURES\_OK (8)|3|Indicates that the driver has acknowledged all the features it understands, and feature negotiation is complete.|
|DRIVER\_OK (4)|2|Indicates that the driver is set up and ready to drive the device.|
|DEVICE\_NEEDS\_RESET (64)|6|Indicates that the device has experienced an error from which it can’t recover.|

**Note:** The value in parentheses is the value, the bit adds to *Status* field.

<br>

Quote from the Spec (Chapter 2.1) on Status bits:

> The device status field provides a simple low-level indication of the completed steps of this sequence.
It’s most useful to imagine it hooked up to traffic lights on the console indicating the status of each device.

The Spec implicitly says that the individual bits of *Status* field are not
mutually exclusive. It means that a bit indicating the current phase, when set,
does not clear a bit indicating the previous phase - the bits are added.
So, the current state can be determined by comparison of previous
and current values of *Status* field.

<br>

#### 2.1.1 INIT State

The Spec implicitly introduces initial state, let's name it "INIT".
It can be described the following way.

**Host side:**

1) Virtio Device is **fully initiated** and is **ready** to provide an information about itself

For MMIO transport:

2) registers *Status* and *InterruptStatus* are set to zero

3) registers *QueueReset* and *QueueReady*, for every Virtqueue, are set to zero

**Guest side:**

Virtio Device Representation on Guest side is in the **initial state** -
some resources might be allocated.

<br>

> Note. "Device Representation" is memory regions/data structures
representing a Virtio Device (including its Virtqueues) in guest OS.
>
> This term is not used in the Spec, but was introduced in the **Tech Illustrations** to make a clear picture of components' roles. 

<br>

State Diagram for external States are presented in document
"Virtio Device State diagram indicated by Status field" located
at docs.kernels-abreeze.dev, 'Virtio Illustrations'.

<br>
 
### 2.2 Life States

'Virtio Device' concept introduces an additional (internal) state:
"RESET\_UNDERWAY".

This State is triggered after Driver has sent a command "Reset Device"
by writing 0 to register *Status*.
In this state, the Device keeps presenting a current value of 
register *Status* until resetting is over, then it zeros the register.

This is [Diagram of States supported in 'Virtio Device' abstraction](./vd01il1-virtio-device-states.jpg)

<br>

## 3 'Virtio Device' Abstraction

<br>

'Virtio Device' abstraction (abstract class) represents generalized Virtio device, it describes common part of all Devices.

'Virtio Device' abstract class includes:

- Device features - both offered and negotiated (in the Spec terminology,
"Device Features" and "Driver Features")
- Device ID fields
- Device's *Status* Field
- size of device-specific part of the Device Config Space
- flag "Device Reset is underway"

In current vision, 'Device Config Space' is aware of and supports the
internal State "RESET\_UNDERWAY".

'Virtio Device' interfaces are declarations of API that every specific
Device should implement.

<br>

## 4 'Virtio Device' Interfaces

<br>

Virtio Core introduces five interfaces, that every 'Specific Virtio Device'
should implement. The interfaces are the following:

1. 'Config Operations' (virtio\_device\_config\_ops) provides access to
all 'Virtio Device' parameters.


2. 'Run Operations' (virtio\_device\_run\_ops) handles Driver events delivered
via Device Config Space.


3. 'Feature Operations' (virtio\_device\_feature\_ops) implements interpretation of Device and Driver feature bits and provides access to features via their IDs.
The interface is designed for components that should stay agnostic of
feature bits coding details.
In contrast, functions 'vdevice\_xxx\_xxxx\_feature\_bits' of 'Config Operations'
interface were designed for interactions with 'Device Config Space'.
The latter uses bit fields in registers *DeviceFeatures* and *DriverFeatures* without any feature interpretation.

4. 'Life Cycle Operations' (virtio\_device\_life\_ops) allows an external
component to set/get Device parameters and setup the Device.


5. 'Notify Driver' is a callback function that component 'Virtqueue Access'
will use to notify Driver of events.
(Interface 4.5 on the Peridot Virtio Component Diagram.)


<br>
  
## 5 Default Implementations

<br>

Interface 'Config Operations' provides access to 'Virtio Device'
abstraction, common for all Devices. 'Spec Virtio Device' implements
the interface following it's internal logic, but there's common
functionality that every Device would do - updating 'Virtio Device'
parameters. Virtio Core provides implementation of this common functionality
in 'Default Config Operations' (default\_virtio\_device\_config\_ops).
Please see example of its use in Virtio Console Stub Device.

Interface 'Feature Operations' provides access to Device features (Feature Bits) - 
both offered and negotiated - by their IDs. There is a number of features
common for all the Devices, so access to them is a common functionality as well.
Virtio Core provides its implementation in 'Default Feature Operations' (default\_virtio\_device\_feature\_ops).
So, 'Spec Virtio Device' only implements access to device-specific features;
access to common ones can be outsourced to 'Default Feature Operations'.
Please see example of its use in 'Virtio Console Stub Device'.

<br>
  
## 6 Consumers

<br>

Component-consumer of 'Config Operations' and 'Run Operations' is 'Device Config Space'.

Component-consumer of 'Feature Operations' and 'Life Cycle Operations' is an
external component. Consumer of 'Feature Operations' can also be
'Specific Virtio Device' itself, when it needs to interpret feature bits.

Component-consumer of 'Notify Driver' is 'Virtqueue Access'.

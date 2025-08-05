# Virtqueue Component

(**VQ01** Ver 1.0. September 2024)

## 1 Definitions


1) **The Spec** is specification "Virtual I/O Device (VIRTIO) Version 1.2"

2) **Driver** or **Device** with the first capital letter refer, respectively,
to Virtio Driver or Virtio Device **side** of the communication. Each side includes the whole stack of components to supply respective Driver's or Device's functionality.

3) **'Virtqueue'** in single quotes is a name of the Peridot Virtio component,
**Virtqueue** without quotes is a type of an object described in the Spec.


## 2 'Virtqueue' Intro


'Virtqueue' component is an abstraction designed to support a bare minimum
of data and functionality to represent the Spec's Virtqueue.
It supports three domains:

- Virtqueue areas - Descriptor Area, Driver Area, Device Area
(three Rings, for Split version)

- 'Virtqueue' parameters
- 'Virtqueue' status

In fact, the component is a simple keeper of Ring's parameters,
Virtqueue parameters and 'Virtqueue' State.
'Virtqueue' is **not** involved into navigation and accessing Virtqueue Rings -
it's covered by component 'Virtqueue Access'.
 
<br>

'Virtqueue' provides three interfaces:

- Life Cycle Operations (virtqueue\_life\_ops)
- Parameter Operations (virtqueue\_param\_ops)
- Status Operations (virtqueue\_status\_ops)

<br>

'Life Cycle Operations' interface currently includes only 'Reset' function.

'Parameter Operations' manages Rings' parameters **"address"**, Virtqueue
parameter **"size"** and enables to get allowable **"maximum Virtqueue size"** ('QueueNumMax').
*Side note: the latter parameter is set via 'reset' function.*



## 3 'Virtqueue' States

<br>

According to the Spec, Device Config registers *QueueReset* and  *QueueReady*
were designed to send commands to Virtqueue and track their fulfillment.

Combination of the Registers' values can't represent 'Virtqueue's' state.
For that, the component keeps variable 'Status'; and interface 'Status Operations' allows to handle it.


**Table 3.1** States description.

|Status Name|Description|
|:---:|:---|
|INIT|'Virtqueue' is newly allocated/statically declared, initiated with parameters provided by Host side and ready for further configuration.<br><br>Also 'Virtqueue' goes into this State after commands "Reset" or "Disable" has been completed.|
|RESET\_UNDERWAY|Command "Reset" is underway|
|CONFIGURED|'Virtqueue' has been fully configured by Driver|
|ENABLED|'Virtqueue' has been enabled by Driver. <br><br>'Underway' state is not reqiured for command "Enable", because everything has been set on Device side by that moment, and the command is just a signal for Device to start using the Virtqueue|
|DISABLE\_UNDERWAY|Command "Disable" is underway|


<br>

Transitions between the States are presented in the [State diagram](./vq01il1-virtqueue-state-diagram.jpg).

There are commands, events and a condition that trigger change of State.
Commands coming from Driver are **"Reset"**, **"Enable"**, **"Disable"**.
Events coming from 'Specific Virtio Device' are
**"Reset" completed**, **"Disable" completed**.

Condition **"Configuration Completed"** is checked
every time Driver sets the next Virtqueue parameter.
If all parameters were set, 'Virtqueue' State changes to CONFIGURED.
Starting from this point, command "Enable" coming from Driver will
make an effect. However, Driver might want to keep
adjusting parameters if its internal logic requires to do so.
In that case, 'Virtqueue' State remains CONFIGURED.

**Design note**. It's important to remember that 'Virtqueue' is solely a keeper
of Virtqueue parameters. Component 'Device Config Space' tracks all those events
and makes the condition checking, so, it's 'Device Config Space' who is responsible for proper maintaining 'Virtqueue' States.


**Table 3.2** Correlation between 'Virtqueue' States and usage of Device Registers **QueueReset** and **QueueReady**:

|QueueReset:<br>Driver Writes / Device Presents|QueueReady:<br>Driver Writes / Device Presents|Status Changed to:|
|:---:|:---:|:---:|
|- / 0|- / 0|INIT|
|1 / 1|- / 0|RESET\_UNDERWAY|
|- / 0|- / 0|CONFIGURED|
|- / 0|1 / 1|ENABLED|
|- / 0|0 / 1|DISABLE\_UNDERWAY|

<br>

## 3 Resets of 'Virtqueue'

<br>

A reset **initiated by Driver** with command "Reset" means only rewinding 'Status' to INIT that allows Driver side to re-configure the Rings in the next step.
Parameter "maximum Virtqueue size" stays unchanged. 

*Side note. 1) Timely resetting of 'Virtq Access' is responsibility of 'Specific Virtio Device'. 2) If a buffer was processed when "reset" came in then 'Specific Virtio Device' should stop processing and report to 'Device Config Space' that Reset has been finished.*

A reset **initiated by Host side** allows to change "maximum Virtqueue size" and
sets 'Status' to INIT. Such resetting forces Device to stop communicating with Driver,
moreover Driver should be informed about it. Obvious way is to make Device
to set it's status to DEVICE\_NEEDS\_RESET and trust that Driver acts
accordingly. Other ways could rely on specifics of a Device type /\* might be elaborated in the future \*/



## 4 Consumers

<br>

In the current vision, component-consumer of both 'Parameter Operations' and
'Status Operations' is 'Device Config Space';
component-consumer of 'Life Cycle Operations' is a 'Specific Virtio Device'.


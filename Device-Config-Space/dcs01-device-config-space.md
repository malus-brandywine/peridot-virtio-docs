# Device Config Space Component

(**DCS01** Ver 1.0. September 2024)

## 1 Definitions

1) **The Spec** is specification "Virtual I/O Device (VIRTIO) Version 1.2"

2) **Driver** or **Device** with the first capital letter refer, respectively,
to Virtio Driver or Virtio Device **side** of the communication. Each side includes the whole stack of components to supply respective Driver's or Device's functionality.

3) **'Device Config Space', or 'Config Space'** in single quotes is a name of the Peridot Virtio component, **Device Config Space** without quotes refers to the Device Configuration Space described in the Spec.

4) **'Virtqueue'** in single quotes is a name of the Peridot Virtio component,
**Virtqueue** without quotes is a type of an object described in the Spec.


## 2 Device Config Space Views

<br>

'Device Config Space' component is designed to handle all interactions with Driver
coming through MMIO-based Device Config Space. 'Device Config Space' **handles**
accesses to common part of Device Config Space (the Spec, section 4.2.2) and
**passes through** accesses to device-specific part of Device Config Space to
'Specific Virtio Device'.


For the clarity of design and further code maintenance, it's convenient to
distinguish three "views" that 'Device Config Space' contains.
"View" is a set of variables inside the component that collectively
allow to cover a specific functionality.

'Device Config Space' includes the three views: Virtio Device view, Virtqueues view and Events view.
 
 <br>
 
**Virtio Device view** allows to handle requests related to a Virtio device as a whole, therefore it keeps a reference to a specific device it serves. Also, the component consumes device-specific implementation of 'Virtio Device' interfaces mandated by Peridot Virtio Core architecture.
On the Peridot Virtio Component Diagram, the interfaces are numbered as 4.1 and 4.2.


<br>

**Virtqueues view** enables to cover all the requests related to a device's Vitrqueues and maintain 'Virtqueue's' States. The view includes:

- array of 'Virtqueue' instances that a specific device supports
- array of 'Virtqueue Config Controls' instances - one for each 'Virtqueue'

'Virtqueue' component is solely a keeper of Virtqueue parameters,
whereas 'Device Config Space' has to take care of interactions
with Driver and handling 'Virtqueue' States. 'Device Config Space' uses
'Vq Controls' to support this functionality.

<br>

**Events view** enables to handle events passed through
'InterruptStatus', 'InterruptACK' registers of Device Config Space.


*Side note. Handling of shared memory regions is not supported in the current version.*

<br>

## 3 Interfaces and Consumers

<br>


'Device Config Space' provides four interfaces to Peridot Virtio components
and one interface to an external component.

Four internal interfaces are:

- Life Cycle Operations (config\_space\_life\_ops)
- Vdevice Operations (config\_space\_vdevice\_ops)
- Vqueue Operations (config\_space\_vqueue\_ops)
- Vevents Operations (config\_space\_vevents\_ops)


In current vision, a component-consumer of all those interfaces is Specific Virtio Device.

'Life Cycle Operations' contains only 'Reset' function at the moment.

When 'Config Space' dispatches Driver requests to 'Specific Virtio Device', it
uses interfaces that the latter provides. To get responses back, 'Config Space'
provides 'Vdevice- and 'Vqueue Operations'.

'Vevents Operations' are used by 'Specific Virtio Device' to inform Driver side
of events.

Interface provided to external component is 'Config Space Handler' (virtio\_device\_config\_space\_handler). It's a function that will be called
when event for the specific Virtio device detected. On the Peridot Virtio Component
Diagram, it's the interface 1.5.


## 4 Virtqueue Config Controls

<br>

'Virtqueue Config Controls' is a tiny component that:

1) tracks 'Virtqueue' configuration progress

2) receives/responds to Driver commands coming via Device Config Space
registers *QueueReset* and *QueueReady*


Design notes:

- 'Vq Controls' is a per-'Virtqueue' instance.<br>
However, it's reasonable to keep component 'Virtqueue' ignorant of " 'Device Config Space' - Driver " interaction details.<br>
That is why array of 'Vq Controls' instances belongs to 'Device Config Space', and no individual 'Vq Controls' should be included in 'Virtqueue'.

- Variable 'Status' naturally belongs to 'Virtqueue', and if other components need to know 'Virtqueue' state, they know where to get it.<br>
But since Driver side regulates Virtqueue State, then the State change is responsibility of 'Device Config Space'.








# Specific Virtio Device

(**SVD01** Ver 1.0. September 2024)

## 1 Definitions

1) **The Spec** is specification "Virtual I/O Device (VIRTIO) Version 1.2"

2) **Driver** or **Device** with the first capital letter refer, respectively,
to Virtio Driver or Virtio Device **side** of the communication. Each side includes the whole stack of components to supply respective Driver's or Device's functionality.

3) **'Device Config Space'** in single quotes is a name of the Peridot Virtio component, **Device Config Space** without quotes refers to the Device Configuration Space described in the Spec.


## 2 Specific Virtio Device

<br>

'Specific Virtio Device' component (further, 'Device' in single quotes) **solely** mediates the logic between of Virtio Driver and a physical device.

It means:

- 'Device' is isolated from specifics of navigation Virtqueue areas 
- 'Device' is not aware of Driver notification scheduling
- 'Device' is isolated from specifics of IRQ channel usage
- 'Device' interacts with Device Config Space through 'Device Config Space' component

As for the first two items, all the mentioned functionality is encapsulated
in 'Virtq Access' component.

<br>

## 3 Driver Notification Call Chain

<br>

**1) Setting**

External component passes a callback function **irq\_based\_notify\_driver**
and an **irq data object** to 'Spec Virtio Device' through vdevice\_setup().
Both the items encapsulate interaction logic and IRQ channel details
associated with a specific 'Device' instance.

'Device' passes a callback function **dev\_notify\_driver** and a **dev data object**
to 'Virtq Access' through reset().

**2) Calling**

'Virtq Access' detects that notification conditions are met - a buffer has been
processed, it's descriptor has been released via Used Ring, Driver notification preferences have been observed; then it calls the function **dev\_notify\_driver**
with the **dev data object** as a parameter.

Implementation of **dev\_notify\_driver** should call a proper function of
'Device Config Space' to update 'InterruptStatus' register and then it should call
the function **irq\_based\_notify\_driver** with the mentioned **irq data object**.

<br>

## 4 Implementation

<br>

'Specific Virtio Device' will contain the following common **mandatory** parameters:

- 'Virtio Device' instance
- pointer to 'Virtqueue' array
- pointer to 'Virtqueue Config Controls' array
- pointer to 'Virtqueue Access' array
- size of 'Virtqueue' array
- variables for keeping **irq\_based\_notify\_driver** and **irq data object**

The remaining parameters are:

- device-specific part of Device Config Space
- other device specific parameters


Specific Virtio Device should implement the five Virtio Device interfaces, described in "Virtio Device Concept" document.

Specific Virtio Device can use 'Default Config Operations' and
'Default Feature Operations'.



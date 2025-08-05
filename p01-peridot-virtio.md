# Peridot Virtio

(**P01** Ver 1.1. August 2025)


“Peridot Virtio” is a Software Architecture, a model of Virtio functionality from standpoint of Devices support (Virtio backend).

## Scope of the Architecture

The Architecture introduces Virtio Core - set of classes that allow a specific Virtio device to interact with Virtqueues and Device Config Space.


It specifies interfaces between:
(1) Virtio Core and a Device
(2) Virtio Core and Dispatching Components
(3) a Device and other Peridot components

Complete introduction to the Architecture locates in "Peridot Virtio Architecture Presentation"




## Components of Virtio Core

In this project, "component" is an instantiation of a class.

**Peridot Virtio Core classes:**

* 'Device Config Space'

* 'Virtqueue'

* 'Virtqueue Access'

* 'Virtqueue Config Controls'

* 'Virtio Device'

</br>

Peridot Virtio component diagram below describes interfaces defined by Peridot Virtio,
using an example of a specific Virtio device with two Virtqueues.

![Peridot Virtio component diagram](p01il2-peridot-virtio-component-diagram.jpg "Peridot Virtio component diagram")



### Naming conventions


**'Device Config Space'**<br>
Short: 'Config Space'<br>
In code: dev-config-space<br>
Structure name: device\_config\_space

**'Virtqueue'**<br>
Short 'Virtq'<br>
In code: virtq<br>
Structure name: virtqueue

**'Virtqueue Access'**<br>
Short 'Virtq Access'<br>
In code: virtq-access<br>
Structure name: virtq\_access

**'Virtqueue Config Controls'**<br>
Short 'Vq Controls'<br>
In code: vq-config-controls<br>
Structure name: virtq\_config\_controls

**'Virtio Device'**<br>
Short: the same<br>
In code: virtio-device<br>
Structure name: virtio\_device

**'Specific Virtio Device'**<br>
Short 'Spec Virtio Device'<br>
In code: spec-virtio-device

In documentation, the short and full names of the components are single quoted.


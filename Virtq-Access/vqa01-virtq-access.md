# Virtqueue Access Component

(**VQA01** Ver 1.0. September 2024)


## Definitions

1) **The Spec** is specification "Virtual I/O Device (VIRTIO) Version 1.2"

2) **Driver** or **Device** with the first capital letter refer, respectively,
to Virtio Driver or Virtio Device **side** of the communication. Each side includes the whole stack of components to supply respective Driver's or Device's functionality.


## Virtqueue Access Component

'Virtqueue Access' covers all the manipulations with the three areas
of Virtqueue - Descriptor Area, Driver Area, Device Area. It takes
care of reading the areas, modifying them, tracking
notification conditions and notifying Driver when notification conditions
are met.

Currently, the component manages only Split Virtqueue type.

'Virtqueue Access' provides slim API that includes two interfaces:

- Descriptor Operations ( 'virtq\_access\_descr\_ops' )
- Operations ( 'virtq\_access\_ops' )


'Descriptor Operations' interface is the most logic-intensive of the two.
It manipulates descriptors and notifications.
Component-consumer of Descriptor Operations is 'Virtio Device' that mediates the logic between of Virtio Driver and a physical device.

'Operations' interface is life cycle functionality: it provides
'Virtq Access's' reset and setting/clearing parameter 'flag\_event\_idx'. The last one reflects a device's feature 'VIRTIO\_F\_EVENT\_IDX'.
Component-consumer of the Operations could be any component responsible
for Virtqueue reset and allowed to set Device feature bits;
in current vision, it's 'Virtio Device'.

Usage of Descriptor Operations is described in "Virtqueue Access Descriptor API".


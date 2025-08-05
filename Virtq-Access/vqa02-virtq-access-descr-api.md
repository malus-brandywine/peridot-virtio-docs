# Virtqueue Access Descriptor API


(**VQA02** Ver 1.0. September 2024)

## 1. Definitions

<br>

1) **The Spec** is specification "Virtual I/O Device (VIRTIO) Version 1.2"

2) **Driver** or **Device** with the first capital letter refer, respectively,
to Virtio Driver or Virtio Device **side** of the communication. Each side includes the whole stack of components to supply respective Driver's or Device's functionality.

3) **Buffer**/**Buffer elements**.
Very often in the 'Split Virtqueue' section, the Spec uses term "buffer" when
it refers to both (a) a chunk of memory described by a single descriptor (addr, len, flags) and (b) a set of memory chunks sequentially chained in a descriptor table.

Nevertheless, the Spec sometimes clearly distinguishes (a) from (b) using "buffer element" for an individual memory chunk, and "buffer" for a whole chain of memory pieces.

<br>

'Virtqueue Access' API uses the general naming scheme:

- **buffer element** is a memory chunk described by a descriptor
- **buffer** is a set of chained buffer elements

<br>

Accordingly, for entities of a Descriptor Table, the API uses:

- **descriptor** and
- **descriptor chain** or **chain**

A single unchained buffer element can be seen as "single-descriptor" chain in contrast to general "multi-descriptor" chain.

<br>


4) **To process a descriptor** means to process an according buffer element.

5) **Buffer completeness** is a condition when all buffer elements of a buffer
were processed by a proper component on Device side.


6) **Reporting a buffer** is a work on (1) updating a Used Ring and (2) notifying Driver that a descriptor has been processed by Device, if notification conditions have been met.

<br>


## 2. Buffer processing model


The API supports sequential reading of descriptors: it extracts descriptors
from a Descriptor Table one by one with every request. It's possible to envision  another model, but it would not be efficient in the environment that Peridot Virtio is designed to run in.

A consumer of 'Virtq Access' API (that is 'Virtio Device') can use the two scenarios
of managing a buffer. Giving them speaking names:

- 'Per-descriptor release' and
- 'Chain release'


### 2.1 Per-descriptor release

In 'Per-descriptor release' scenario , 'Virtio Device' plays role of a "pipe".

For a descriptor chain, 'Virtio Device':

- receives descriptors from 'Virtq Access' one by one,
- passes their buffer elements to other component for processing,
- asynchronously receives processed buffer elements back,
- "releases" descriptors as soon as they've been processed. 

**"Releasing a descriptor"** means informing 'Virtq Access' that an according buffer element has been processed.

Please see  [Illustration 2\.1](./vqa02il1.1-virtq-access-descr-api.jpg)

<br>

API functions used for this scenario are:

- **next\_descr()** to get the next descriptor in the chain,
- **release\_descr()** to release 'id'-th descriptor


In this scenario, it is 'Virtq Access' who detects **completeness of a buffer** and makes the decision when to **report a buffer to Driver** (see Definitions).

'Virtq Access' detects a buffer completeness comparing the number of requested descriptors ( with next\_descr() ) against the  number of released ones ( with release\_descr() ), no matter the order they've been released. Also, release\_descr() informs 'Virtio Device' about buffer completeness via parameter "eob" when 'Virtq Access' detects it was the last unprocessed descriptor.

'Virtq Access' also sums lengths of data actually written to physical device from all
device-writable buffer elements of a chain. The sum is used later, when a
buffer is returned to Driver.


```
Important. For the last released descriptor, 'release_descr()' returns control to 'Virtio Device' after a buffer has been reported.
```

<br>

The fact that 'Virtq Access' keeps track of a chain state, opens opportunity
for more sophisticated checks of what 'Virtio Device' releases. For example,
existing basic check described above is not able to detect the situation
when 'Virtio Device' mistakenly releases the same descriptor N times instead of
releasing N different descriptors.


### 2.2 Chain release

In 'Chain release' scenario, 'Virtio Device' is fully responsible for
buffer completeness detection.


'Virtio Device' obtains descriptors one by one until it reaches the end of a
descriptor chain. Then it passes buffer elements to processing, following it's internal logic. When all the buffer elements have been processed, 'Virtio Device' "releases a chain" - informs 'Virtq Access' that it's time to report a buffer.

Now 'Virtq Access' plays a role of a "pipe", it does what
'Virtio Device' tells it to do without checking of buffer completeness.

```
Important. 'release_chain()' returns control to 'Virtio Device' after a buffer has been reported.
```

<br>

API functions used for this scenario are:

- **next\_descr()** to get the next descriptor in the chain,
- **release\_chain()** to release a chain


Please see [Illustration 2\.2](./vqa02il1.2-virtq-access-descr-api.jpg)



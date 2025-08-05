# Virtqueue Access internals notes

(**VQA03** Ver 1.0. September 2024)


The documents observes definitions introduced in "Virtqueue Access Descriptor API"


## 1 Notifications to Driver

Reporting a buffer includes both modification of 'Used Ring' and
sending notification to Driver. Modification of 'Used Ring' is fulfilled every time a buffer is returned to Driver. However sending notifications is conditional.


The Spec v\.1\.2, chapter 2\.7\.7\.2 shows that Device is not strictly prohibited
("SHOULD NOT") from sending notifications when conditions to SUPPRESS them are met.
The Spec only mandates Device to send notifications when conditions to SEND
are met.

Tables of Device requirements on "Used Buffer" event notification from chapter 2\.7\.7\.2.

<br>

1) feature **VIRTIO\_F\_EVENT\_IDX** **was not** negotiated


|VIRTQ\_AVAIL\_F\_NO\_INTERRUPT<br>is set in Flags|Device action|
|:---:|:---:|
|0|Must Send|
|1|Should Not Send|



|used_event == idx<br>in Used Ring|Device action|
|:---:|:---:|
|0|Ignores|
|1|Ignores|

<br>

2) feature **VIRTIO\_F\_EVENT\_IDX** **was** negotiated


|VIRTQ\_AVAIL\_F\_NO\_INTERRUPT<br>is set in Flags|Device action|
|:---:|:---:|
|0|Ignores|
|1|Ignores|



|used_event == idx<br>in Used Ring|Device action|
|:---:|:---:|
|0|Should Not Send|
|1|Must Send|



## 2 Chain State (internal details)


```
/* At least one descriptor of a chain has been obtained by spec-virtio-device */
#define STATE_CHAIN_OPEN        1

/* The last descriptor of the chain has been obtained by spec-virtio-device */
#define STATE_CHAIN_CLOSED      2

/* All obtained descriptors have been processed by spec-virtio-device.
 * The State is not currently used in the code because the condition
 * is handled as soon as it's detected in the same function.
 * The State left for consistency */
#define STATE_CHAIN_COMPLETED   3

/* Initial state; the last completed chain has been reported,
 * no new descriptors has been read by spec-virtio-device */
#define STATE_CHAIN_DEFAULT     4

```









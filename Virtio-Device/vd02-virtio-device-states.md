# Device Status and *Status* Register (MMIO)


(**VD02** Ver 1.0. October 2024)

## 1 Definitions

1) **The Spec** is specification "Virtual I/O Device (VIRTIO) Version 1.2"


## 2 *Status* Register

### 2.1 States triggered by Driver


In Device Config Space, *Status* register represents a "progress bar" where
finished phases of initialization or a Device failure marked in individual bits.

It's 'Virtio Device' component who keeps a value of *Status* register (variable 'Status').
This value is passed "as is" to Driver, when the latter makes a read access to the Register.

When Driver writes into the *Status* register, 'Device Config Space' component
determines which bit was set with this access and passes newly changed bit to
'Virtio Device'.
The reason why 'Device Config Space' detects the bit is that it needs
to handle some cases.

1) In the case when bit FEATURES\_OK is set, the component mediates feature negotiation.

2) In the case when Driver resets a Device,
the component calls a specific 'Virtio Device' handler, then zeros *Status* register.

3) In the case when bit FAILED is set, the component calls another specific 'Virtio Device' handler.

4) In other cases, the component just passes newly changed bit to 'Virtio Device'.
In it's turn, 'Virtio Device' might want to handle the new State before updating *Status* register.


Summarizing all the above, there is an asymmetry in the **vdevice\_get\_status()**
and **vdevice\_set\_status()** (interface 'virtio\_device\_config\_ops') usage.
Function **vdevice\_get\_status()** passes the whole "progress bar" to
'Device Config Space', whereas **vdevice\_set\_status()** passes a changed bit.
Then 'Virtio Device' adds a new bit to variable 'Status' updating the "progress bar".


### 2.2 The State triggered by Device

When 'Virtio Device' requests Driver to initiate a Device reset, it asks
'Device Config Space' component to do so (calls vdevice\_needs\_reset() of interface
config\_space\_vdevice\_ops). 
'Device Config Space' handles it the same way like if the State change comes
from Driver.

<br>

## 3 'Virtio Device' States

<br>

As mentioned in **'Virtio Device' Concept** (VD01), 'Virtio Device'
supports and mandates additional state "RESET\_UNDERWAY".
Since we cannot use variable 'Status' for that, for the obvious reason, then the component needs additional variable.'Virtio Device' introduces 32-bit variable 'CurStatus'.

'Virtio Core' does not deal with 'CurStatus' directly and is not aware of
how 'Specific Virtio Device' uses it.

'Specific Virtio Device' may use it as a copy of *Status* register
(variable 'Status') appended by "RESET\_UNDERWAY".
However, there is more interesting scenario: for some Devices, it may be convenient to keep in 'CurStatus' a current state, not a whole "progress bar". The 'Device' can determine a current state as the last modified bit (the exception is "INIT" state).

In that case, 'CurStatus' will have has the same bit mapping as variable 'Status', but will keep only the last modified bit.

'Virtio Device' suggests to use 16-th bit of 'CurStatus' for State "RESET\_UNDERWAY", mask 0x10000. However, 'Specific Virtio Device'
may redefine it.

Along with "RESET\_UNDERWAY", 'Spec Virtio Device' can support other
internal States and assign unused bits to them.


Please see [illustration with values of variables 'Status' and 'CurStatus' in various 'Virtio Device' States](./vd02il01-status-curstatus.jpg)


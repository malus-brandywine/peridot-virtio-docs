
## Peridot Virtio Console Stub device


In Peridot Virtio, 'Virtio Console Stub' device is a basic Virtio Device
that can be used to examine whether Peridot Virtio Core running in a hypervisor
and Virtio system in a guest OS can find and interact with each other.


### Sides of the communication 

 - In a guest OS, Virtio Console driver expects to "see" full-fledged
Virtio Console device, so no special modifications are required on the guest side.
In this example Linux guest is used.

 - In a hypervisor, Virtio Console Stub does not interacts with any
physical device either directly or indirectly, so it is self-sufficient
in that respect. The idea behind the use of such a simplified Device is
to set up interaction between a Virtio Device and a Virtio Driver without
designing sophisticated system configurations.

Illustration of the Stub device role and place (see Use-of-Virtio-Console-Stub-Device.pdf).


### What Virtio Console Stub device is doing


In intial version, Virtio Console Stub supports only one port: one virtqueue
per each direction - Rx & Tx.


1. Virtio Device discovery phase

The Console Stub device provides all the necessary information to Virtio
Console driver in the device discovery phase.

Illustration of the communications see Use-of-Virtio-Console-Stub-Device-Communications-v2.pdf


2. Testing Rx virtqueue exchange

In Virtio Device initialization phase, Driver aims to fill in Rx virtqueue
completely sending the device empty device-writable buffer elements.
In the process, the Console Stub device consumes the first `RX_TEST_MSG_NUM`
incoming Rx buffer elements - fills them in with predefined text - and then
passes them back to Driver. Driver, in its trun, confirms the buffer elements retrieval.

While each of those `RX_TEST_MSG_NUM` buffer elements are being returned
to Driver, the latter sends each of them back to maintain Rx virtqueue full.

To verify that this part works correctly we should check what the Stub device
prints out during Linux kernel initialization. In a simple case when the hypervisor
and the guest Linux share the same serial port for their system consoles, 
diagnostic messages from the Stub device will be interlaced with Linux kernel messages.


The diagnostics from the Stub device should confirm the described earlier scenario.

The following figure is a clip of current version of diagnostics - Peridot
messages intermixed with Linux ones. In this configuration `RX_TEST_MSG_NUM` is 2,
the virtueues' length is 4.

```
[    5.347146] i2c-core: driver [si570] registered
[    5.351154] i2c-core: driver [si5324] registered
[    5.355737] i2c-core: driver [idt8t49n24x] registered
Peridot Virtio Device:Virtio Console Stub: Driver requested Device Reset
[    5.387541] Serial: 8250/16550 driver, 4 ports, IRQ sharing disabled
Peridot Virtio Device:Virtio Console Stub: New RX buffer has arrived; Available RX buffers now: 1
Peridot Virtio Device:Virtio Console Stub: Gets the new available RX descriptor with Success (0)
Peridot Virtio Device:Virtio Console Stub: Descriptor params: uin 0, addr 0x1eac1000, len 4096, flag 'Writable' 1, flag 'Is the last in the Chain' 1
Peridot Virtio Device:Virtio Console Stub: The RX buffer was filled in with:
<<<<   <<<<   <<<<
Our hero, our hero
Claims a warrior's heart
>>>>   >>>>   >>>>
Peridot Virtio Device:Virtio Console Stub: Notifying Driver
Peridot Virtio Device:Virtio Console Stub: The RX buffer was released: flag 'End of Buffer': 1
Peridot Virtio Device:Virtio Console Stub: Available RX buffers now: 0
Peridot Virtio Device:Virtio Console Stub: The RX Used Buffer was retrieved by Driver
Peridot Virtio Device:Virtio Console Stub: New RX buffer has arrived; Available RX buffers now: 1
Peridot Virtio Device:Virtio Console Stub: Gets the new available RX descriptor with Success (0)
Peridot Virtio Device:Virtio Console Stub: Descriptor params: uin 0, addr 0x1eac1000, len 4096, flag 'Writable' 1, flag 'Is the last in the Chain' 1
Peridot Virtio Device:Virtio Console Stub: The RX buffer was filled in with:
<<<<   <<<<   <<<<
I tell you, I tell you
The Dragonborn comes
>>>>   >>>>   >>>>
Peridot Virtio Device:Virtio Console Stub: Notifying Driver
Peridot Virtio Device:Virtio Console Stub: The RX buffer was released: flag 'End of Buffer': 1
Peridot Virtio Device:Virtio Console Stub: Available RX buffers now: 0
Peridot Virtio Device:Virtio Console Stub: The RX Used Buffer was retrieved by Driver
Peridot Virtio Device:Virtio Console Stub: New RX buffer has arrived; Available RX buffers now: 1
Peridot Virtio Device:Virtio Console Stub: New RX buffer has arrived; Available RX buffers now: 2
Peridot Virtio Device:Virtio Console Stub: New RX buffer has arrived; Available RX buffers now: 3
Peridot Virtio Device:Virtio Console Stub: New RX buffer has arrived; Available RX buffers now: 4
[    5.391056] cacheinfo: Unable to detect cache hierarchy for CPU 0
[    5.396841] brd: module loaded
[    5.402167] loop: module loaded
[    5.402237] i2c-core: driver [at24] registered
[    5.404280] i2c-core: driver [tps65086] registered

```
 


3. Testing Tx virtqueue exchange


Test 1. After Linux has loaded, if your init scripts initiate getty-like utility
on /dev/hvc0, then you will see the utility greeting message and login prompt,
bounced back by Console Stub Device.

Example of the output:

```

Peridot Virtio Device:Virtio Console Stub: New TX buffer has arrived
Peridot Virtio Device:Virtio Console Stub: Console Stub Gets the new available TX descriptor with Success (0)
Peridot Virtio Device:Virtio Console Stub: Descriptor params: uin 0, addr 0x12455b80, len 1, flag 'Writable' 0, flag 'Is last in the Chain' 1
Peridot Virtio Device:Virtio Console Stub: Contents of the TX buffer intended for a Physical Device:
<<<<   <<<<   <<<<

>>>>   >>>>   >>>>
Peridot Virtio Device:Virtio Console Stub: Notifying Driver
Peridot Virtio Device:Virtio Console Stub: The TX buffer was released: flag 'End of Buffer': 1
Peridot Virtio Device:Virtio Console Stub: The TX Used Buffer was retrieved by Driver
Peridot Virtio Device:Virtio Console Stub: New TX buffer has arrived
Peridot Virtio Device:Virtio Console Stub: Console Stub Gets the new available TX descriptor with Success (0)
Peridot Virtio Device:Virtio Console Stub: Descriptor params: uin 0, addr 0x12455b80, len 2, flag 'Writable' 0, flag 'Is last in the Chain' 1
Peridot Virtio Device:Virtio Console Stub: Contents of the TX buffer intended for a Physical Device:
<<<<   <<<<   <<<<


>>>>   >>>>   >>>>
Peridot Virtio Device:Virtio Console Stub: Notifying Driver
Peridot Virtio Device:Virtio Console Stub: The TX buffer was released: flag 'End of Buffer': 1
Peridot Virtio Device:Virtio Console Stub: The TX Used Buffer was retrieved by Driver
Peridot Virtio Device:Virtio Console Stub: New TX buffer has arrived
Peridot Virtio Device:Virtio Console Stub: Console Stub Gets the new available TX descriptor with Success (0)
Peridot Virtio Device:Virtio Console Stub: Descriptor params: uin 0, addr 0x12455b80, len 61, flag 'Writable' 0, flag 'Is last in the Chain' 1
Peridot Virtio Device:Virtio Console Stub: Contents of the TX buffer intended for a Physical Device:
<<<<   <<<<   <<<<
PetaLinux 2023.2+release-S10121051 u96v2-sbc-base-2023-2 hvc0
>>>>   >>>>   >>>>
Peridot Virtio Device:Virtio Console Stub: Notifying Driver
Peridot Virtio Device:Virtio Console Stub: The TX buffer was released: flag 'End of Buffer': 1
Peridot Virtio Device:Virtio Console Stub: The TX Used Buffer was retrieved by Driver
Peridot Virtio Device:Virtio Console Stub: New TX buffer has arrived
Peridot Virtio Device:Virtio Console Stub: Console Stub Gets the new available TX descriptor with Success (0)
Peridot Virtio Device:Virtio Console Stub: Descriptor params: uin 0, addr 0x12455b80, len 2, flag 'Writable' 0, flag 'Is last in the Chain' 1
Peridot Virtio Device:Virtio Console Stub: Contents of the TX buffer intended for a Physical Device:
<<<<   <<<<   <<<<


>>>>   >>>>   >>>>
Peridot Virtio Device:Virtio Console Stub: Notifying Driver
Peridot Virtio Device:Virtio Console Stub: The TX buffer was released: flag 'End of Buffer': 1
Peridot Virtio Device:Virtio Console Stub: The TX Used Buffer was retrieved by Driver
Peridot Virtio Device:Virtio Console Stub: New TX buffer has arrived
Peridot Virtio Device:Virtio Console Stub: Console Stub Gets the new available TX descriptor with Success (0)
Peridot Virtio Device:Virtio Console Stub: Descriptor params: uin 0, addr 0x12455b80, len 2, flag 'Writable' 0, flag 'Is last in the Chain' 1
Peridot Virtio Device:Virtio Console Stub: Contents of the TX buffer intended for a Physical Device:
<<<<   <<<<   <<<<


>>>>   >>>>   >>>>
Peridot Virtio Device:Virtio Console Stub: Notifying Driver
Peridot Virtio Device:Virtio Console Stub: The TX buffer was released: flag 'End of Buffer': 1
Peridot Virtio Device:Virtio Console Stub: The TX Used Buffer was retrieved by Driver
Peridot Virtio Device:Virtio Console Stub: New TX buffer has arrived
Peridot Virtio Device:Virtio Console Stub: Console Stub Gets the new available TX descriptor with Success (0)
Peridot Virtio Device:Virtio Console Stub: Descriptor params: uin 0, addr 0x12455b80, len 21, flag 'Writable' 0, flag 'Is last in the Chain' 1
Peridot Virtio Device:Virtio Console Stub: Contents of the TX buffer intended for a Physical Device:
<<<<   <<<<   <<<<
u96v2-sbc-base-2023-2
>>>>   >>>>   >>>>
Peridot Virtio Device:Virtio Console Stub: Notifying Driver
Peridot Virtio Device:Virtio Console Stub: The TX buffer was released: flag 'End of Buffer': 1
Peridot Virtio Device:Virtio Console Stub: The TX Used Buffer was retrieved by Driver
Peridot Virtio Device:Virtio Console Stub: New TX buffer has arrived
Peridot Virtio Device:Virtio Console Stub: Console Stub Gets the new available TX descriptor with Success (0)
Peridot Virtio Device:Virtio Console Stub: Descriptor params: uin 0, addr 0x12455b80, len 1, flag 'Writable' 0, flag 'Is last in the Chain' 1
Peridot Virtio Device:Virtio Console Stub: Contents of the TX buffer intended for a Physical Device:
<<<<   <<<<   <<<<
 
>>>>   >>>>   >>>>
Peridot Virtio Device:Virtio Console Stub: Notifying Driver
Peridot Virtio Device:Virtio Console Stub: The TX buffer was released: flag 'End of Buffer': 1
Peridot Virtio Device:Virtio Console Stub: The TX Used Buffer was retrieved by Driver
Peridot Virtio Device:Virtio Console Stub: New TX buffer has arrived
Peridot Virtio Device:Virtio Console Stub: Console Stub Gets the new available TX descriptor with Success (0)
Peridot Virtio Device:Virtio Console Stub: Descriptor params: uin 0, addr 0x12455b80, len 7, flag 'Writable' 0, flag 'Is last in the Chain' 1
Peridot Virtio Device:Virtio Console Stub: Contents of the TX buffer intended for a Physical Device:
<<<<   <<<<   <<<<
login: 
>>>>   >>>>   >>>>
Peridot Virtio Device:Virtio Console Stub: Notifying Driver
Peridot Virtio Device:Virtio Console Stub: The TX buffer was released: flag 'End of Buffer': 1
Peridot Virtio Device:Virtio Console Stub: The TX Used Buffer was retrieved by Driver

```


The useful payload here was:

```


PetaLinux 2023.2+release-S10121051 u96v2-sbc-base-2023-2 hvc0

u96v2-sbc-base-2023-2 login:
```




Test 2. After you've logged in, run command:

echo "Any text you like" > /dev/hvc0


You should see messages from the Stub device that describe the use of Tx
buffer elements:
 - their contents received from a guest Linux will be printed out,
 - buffer elements will be returned to Virtio Console driver,
 - the Stub device will confirm the buffer elements retrieval by Driver


The clip of current version of diagnostics:


```
root@ultra96v2-2020-1:~# echo "With a Voice wielding power  Of the ancient Nord art" > /dev/hvc0
Peridot Virtio Device:Virtio Console Stub: New TX buffer has arrived
Peridot Virtio Device:Virtio Console Stub: Console Stub Gets the new available TX descriptor with Success (0)
Peridot Virtio Device:Virtio Console Stub: Descriptor params: uin 0, addr 0x14b98900, len 52, flag 'Writable' 0, flag 'Is last in the Chain' 1
Peridot Virtio Device:Virtio Console Stub: Contents of the TX buffer intended for a Physical Device:
<<<<   <<<<   <<<<
With a Voice wielding power  Of the ancient Nord art
>>>>   >>>>   >>>>
Peridot Virtio Device:Virtio Console Stub: Notifying Driver
Peridot Virtio Device:Virtio Console Stub: The TX buffer was released: flag 'End of Buffer': 1
Peridot Virtio Device:Virtio Console Stub: The TX Used Buffer was retrieved by Driver
Peridot Virtio Device:Virtio Console Stub: New TX buffer has arrived
Peridot Virtio Device:Virtio Console Stub: Console Stub Gets the new available TX descriptor with Success (0)
Peridot Virtio Device:Virtio Console Stub: Descriptor params: uin 0, addr 0x14b98900, len 2, flag 'Writable' 0, flag 'Is last in the Chain' 1
Peridot Virtio Device:Virtio Console Stub: Contents of the TX buffer intended for a Physical Device:
<<<<   <<<<   <<<<


>>>>   >>>>   >>>>
Peridot Virtio Device:Virtio Console Stub: Notifying Driver
Peridot Virtio Device:Virtio Console Stub: The TX buffer was released: flag 'End of Buffer': 1
Peridot Virtio Device:Virtio Console Stub: The TX Used Buffer was retrieved by Driver
root@ultra96v2-2020-1:~# 
```


Note. The term "released" applied to 'buffer'  and the concept "flag 'End of Buffer'"
are specific to Peridot Virtio implementation, they are not part of the Specification.












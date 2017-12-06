# Peripherals

Peripherals live on the front side bus with other fundamental components. Plugging a peripheral in via PCIx, SCSI, or SATA physically locates that device on the system bus. Other devices like USB devices exist on a separate bus from the system bus, but are synchronized with the system bus using a USB controller and a device driver that can translate USB bus messages into system bus messages.

## Network interfaces

Independent processor caches network traffic:

[OSI model](https://en.wikipedia.org/wiki/OSI_model)

Memory address, stack pointer, data transmission rules
Processor communicates asynchronously over network connection in order to satisfy rules of protocol in use - either TCP or UDP.

IP

[Internet Protocol](https://en.wikipedia.org/wiki/Internet_Protocol)

TCP

[Transmission Control Protocol](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)

UDP

[User Datagram Protocol](https://en.wikipedia.org/wiki/User_Datagram_Protocol)

#### More recommended reading

[Just2Good Description of Networking](http://www.just2good.co.uk/networking.php)

## DMA

[Direct Memory Access](https://en.wikipedia.org/wiki/Direct_memory_access)

### Hard disks

Platter-based hard disks used to be a very interesting subject of research and discussion. Imagine ultra-smooth platters spinning 100 times per second, with binary data encoded on them in sections < 100 nanometers. The hard drive is the most space age piece of equipment in your home.

You can learn about them here:
[Engineer Guy Hard Drive](https://en.wikipedia.org/wiki/File:Harddrive-engineerguy.ogv)

Platter based hard disks have an insane storage cost, less than $0.03 per gigabyte:

[BackBlaze](https://www.backblaze.com/blog/hard-drive-cost-per-gigabyte/)

SSDs are less interesting - they are just flash memory with a controller that engages your system's PCI bus.

## Cyclic Redundancy Checking

### Graphics Accelerators

Dedicated graphics pipelines - hardware designed just for manipulating pixels in an array and updating them in a shared memory used by the display. Separate pipeline from the CPU, again, graphics memory can be written to display without needing to be circled around with the CPU.

#### Shaders

C++ software (gsl, actually) that executes simple mathematics on every vertex simultaneously.

#### Pipeline components



#### Software components

OpenGL, DirectX, WebGL. CUDA, GLSL

#### Onboard memory


# Assignment 1

1. The minimum seek time for an HDD is 9msec, and the maximum seek time is 90msec. The block size of this HDD is 4KB. How long on average does it take to read 100MB of data?

2. Describe a TCP/IP packet in detail. Describe the header, how many bytes it is, and which components it contains. What data can come after the header?

3. How does the network protocol guarantee that a TCP/IP packet is complete after transmission?

4. What is the difference between TCP and IP?

5. Why is 3d performance so much higher with a graphics card than without? Modern CPUs are extremely fast, what is limiting their performance?

# Assignment 2

Add interrupts to the LS-8 emulator from the Computer Architecture 1.

You must have implemented a CPU stack before doing this.


The CPU will have 8 interrupts, 0-7.

* Use another numeric register, e.g. 254, for the Interrupt Status register
  (IS). This shows you which interrupts have occurred.

* Use yet another numeric register, e.g. 253, for the Interrupt Mask register
  (IM). The mask is used to block interrupts you're not interested in.
  Initialize this to 0 so all interrupts are blocked.

* Set up an _interrupt vector table_ in RAM. This is a table of addresses for
  each of the 8 interrupts. When an interrupt occurs, the PC will jump to the
  address listed in the interrupt vector table for that particular interrupt.

  Put the 8 addresses at the top of RAM. But remember to initialize your stack
  pointer to start _below_ the interrupt vector table so you don't overwrite it
  the first time you `PUSH` or `CALL`.

The easiest interrupt to set up would be a timer interrupt. Let's call that
interrupt #0. Once per second, say, it issues an interrupt that the CPU has to
handle. You can use `setInterval()` to set bit #0 in the IS register.

```javascript
this.reg[IS] |= 1; // set bit #0
```

The CPU should check for interrupts before executing the next instruction.

## Example

The assembly program is interested in getting timer interrupts, so it sets the
IM register (253) to `00000001` with `SET` and `SAVE`.

The `setInterval()` timer fires and sets bit #0 in IS.

At the beginning of `tick()`, the CPU checks to see if interrupts are enabled.
If not, it continues processing instructions as normal. Otherwise:

Bitwise-AND the IM register with the IS register. This masks out all the
interrupts we're interested in:

```javascript
let interrupts = this.reg[IM] & this.reg[IS];
```

Step through each bit of `interrupts` and see which interrupts are set.

```javascript
for (let i = 0; i < 8; i++) {
  // Right shift interrupts down by i, then mask with 1 to see if that bit was set
  let interruptHappened = ((interrupts >> i) & 1) == 1;

  ...
}
```

(If the no interrupt bits are set, then stop processing interrupts and continue
executing the next instruction as per usual.)

If `interruptHappened`:

Disable interrupts (this is just setting a boolean flag in the CPU).

Clear the bit in the `IS` register for this interrupt now that we're handling
it.

Push the current `PC` on the stack.

Look up the address to jump to in the interrupt vector table (this will be the
*i*th entry).

Set the `PC` to that address.

We are now handling the interrupt.

Continue processing instructions as normal until we hit an `RETI` instruction
(return from interrupt).

On the `RETI`:

Pop the stack and store the result in `PC`.

Enable interrupts.

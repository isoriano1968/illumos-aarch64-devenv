Repository for SolARM drivers (wip)

framebuffer for VideoCore of BCM2711

- Communication done via mailboxes (mbox)

- The buffer must be 16-byte aligned as only the upper 28 bits of the address can be passed via the mailbox
volatile unsigned int __attribute__((aligned(16))) mbox[36];
- First we include _io.h_ as we need access to the `PERIPHERAL_BASE` definition and also to make use of the `mmio_read` and `mmio_write` functions that _io.c_ provides. MMIO experience is useful here, as sending/receiving mailbox request/responses is achieved using the same technique. We'll just be addressing different offsets from `PERIPHERAL_BASE`, as you see in the code.

Importantly, our mailbox buffer (where messages will be stored) needs to be correctly aligned in memory. This is one example where we need to be strict with the compiler instead of letting it do its thing! By ensuring the buffer is "16-byte aligned", we know that its memory address is a multiple of 16 i.e. the 4 least significant bits are not set. That's good, because only the 28 most significant bits can be used as the address, leaving the 4 least significant bits to specify the **channel**.

- I recommend reading up on the [mailbox property interface](https://github.com/raspberrypi/firmware/wiki/Mailbox-property-interface). You'll see that channel 8 is reserved for messages from the ARM for response by the VideoCore, so we'll be using this channel.

`mbox_call` implements all the MMIO we need to send the message (assuming it's been set up in the buffer) and await the reply. The VideoCore will write the reply directly to our original buffer.

The framebuffer
---------------

Now take a look at _fb.c_. The `fb_init()` routine makes our very first mailbox call, using some definitions from _mb.h_. Remember the email analogy? Well, since it's possible to ask a person to do more than one thing by email, we can also ask a few things of the VideoCore at once. This message asks for two things:

 * A pointer to the framebuffer start (`MBOX_TAG_GETFB`)
 * The pitch (`MBOX_TAG_GETPITCH`)

You can read more about the message structure on the [mailbox property interface](https://github.com/raspberrypi/firmware/wiki/Mailbox-property-interface) page that I shared before.

A **framebuffer** is simply an area of memory that contains a bitmap which drives a video display. In other words, we can manipulate the pixels on the screen directly by writing to specific memory addresses. We will first need to understand how this memory is organised though.

In this example, we ask the VideoCore for:

 * a simple 1920x1080 (1080p) framebuffer
 * a depth of 32 bits per pixel, with an RGB pixel order

So each pixel is made up of 8-bits for the Red value, 8-bits for Green, 8-bits for Blue and 8-bits for the Alpha channel (representing transparency/opacity). We're asking that the pixels are ordered in memory with the Red byte coming first, then Green, then Blue - RGB. In actual fact, the Alpha byte always comes ahead of all of these, so it's really ARGB.

We then send the message using channel 8 (`MBOX_CH_PROP`) and check that what the VideoCore sends back is what we asked for. It should also tell us the missing piece of the framebuffer organisation puzzle - the number of bytes per line or **pitch**.

If everything comes back as expected, then we're ready to write to the screen!

It's a good time to gain an understanding of the [screen resolution settings](https://pimylifeup.com/raspberry-pi-screen-resolution/) for the RPi4.



The code is organized like so.
* Most work is on xio.cpp/.h
* low-level devices are DEVs and have a dev structure
* 

Controller.cpp  controll_init():
<pre>
    SerialUSB.setConnectionCallback([&](bool connected) {
        cs.state_usb0 = connected ? CONTROLLER_CONNECTED : CONTROLLER_NOT_CONNECTED;
    });

    SerialUSB1.setConnectionCallback([&](bool connected) {
        cs.state_usb1 = connected ? CONTROLLER_CONNECTED : CONTROLLER_NOT_CONNECTED;
    });
</pre>
( Yes, those are lambdas, ignore that for now. It’s likely to change, but I’ll handle that. This gets it done fast, but not necessarily efficiently. )

`cs.state_usb0` and `cs.state_usb1` are probably not what you want, after further investigation. It doesn’t matter, they’r easy to set back.

What does matter is that somewhere in an init (one-time call — but if it gets called again it’ll be fine, just wasteful) SerialUSB.setConnectionCallback() to tell it what to call. It can be a function pointer or a labda like shown, as long as it has the signature of
<pre>
	void F(bool)
</pre>
then it’ll work.

This works the exact same:
<pre>
void changedStateCh0(bool connected) {
    cs.state_usb0 = connected ? CONTROLLER_CONNECTED : CONTROLLER_NOT_CONNECTED;
}
void changedStateCh1(bool connected) {
    cs.state_usb1 = connected ? CONTROLLER_CONNECTED : CONTROLLER_NOT_CONNECTED;
}

void controller_init(uint8_t std_in, uint8_t std_out, uint8_t std_err)
{
    //…

    SerialUSB.setConnectionCallback(changedStateCh0);

    SerialUSB1.setConnectionCallback(changedStateCh1);
}
</pre>
 `Connected` is as it says, true for now connected, false for now disconnected. It should only get called on an edge — when it changes. You shouldn’t see two back-to-back connected=true calls with the same callback.


I think you’re going to want to move those to xio.cpp, but you’re going to want to make a way for controller to get that generated info, I suspect.


As for reading and writing:

There are two SerialUSB objects: SerialUSB  (NO zero, but easily changed), and SerialUSB1. They are identical in function. From here on, when I say SerialUSB, I also mean SerialUSB1 — they’re identical.

The interface is simple:
<pre>
int16_t SerialUSB.readByte()

uint16_t read(uint8_t *buffer, const uint16_t length); // Blocking for now

int32_t write(const uint8_t *data, const uint16_t length); // Also blocking
</pre>
There is also `readSome()` and `writeSome()` functions that are non-blocking, but otherwise have the same call syntax. read/readSome and write/writeSome all return the amount they read or wrote, which is possibly 0 in the *Some cases.

We haven’t used the blocking read() yet, and the blocking write only blocks until another 256 (or 128 for dual USB) buffer becomes available and it can push the data to it. This is rarely very long.

`length` is required in all cases. I have fixed this in the Motate repo to allow length==0 to means until the terminating NULL, but that’s not ported yet, and porting it’s not practical yet. So, always give a value length (strlen(ptr) works).

readByte() will return -1 if there’s nothing to read. It’s been doing that this whole time, inside xio -> read_char(), which is in turn called by read_line(), where it’s called _FDEV_ERR and means the same thing.

I modified the _write() system call (that gets called from printf) to take file id == 1 as the SerialUSB1, and everything else as SerialUSB. It calls SerialUSB.write(), the blocking version.
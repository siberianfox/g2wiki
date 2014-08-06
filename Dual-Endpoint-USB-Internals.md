This page provides details of the internals of the dual endpoint USB function and the how G2 handles devices in general.

## General
The code is organized like so.
* The low-level devices are provided by Motate devices
* The device primitives are wrapped as C++ classes in xio.cpp/.h
* xio.cpp also provides higher level functions such as device / channel bindings and line reader semantics
* The interface presented to upper layers (i.e. the controller) provides a simple function that can be used to read the control and data channels from any type of device.

### xio.cpp Notes
_Notes from here out are unfinished. Please don't rely on them just yet_
 
Controller.cpp  controll_init():
```c++
    SerialUSB.setConnectionCallback([&](bool connected) {
        cs.state_usb0 = connected ? CONTROLLER_CONNECTED : CONTROLLER_NOT_CONNECTED;
    });

    SerialUSB1.setConnectionCallback([&](bool connected) {
        cs.state_usb1 = connected ? CONTROLLER_CONNECTED : CONTROLLER_NOT_CONNECTED;
    });
```
( Yes, those are lambdas, ignore that for now. It’s likely to change, but I’ll handle that. This gets it done fast, but not necessarily efficiently. )

`cs.state_usb0` and `cs.state_usb1` are probably not what you want, after further investigation. It doesn’t matter, they’r easy to set back.

What does matter is that somewhere in an init (one-time call — but if it gets called again it’ll be fine, just wasteful) SerialUSB.setConnectionCallback() to tell it what to call. It can be a function pointer or a labda like shown, as long as it has the signature of
```c++
	void F(bool)
```
then it’ll work.

This works the exact same:
```c++
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
```

 `Connected` is as it says, true for now connected, false for now disconnected. It should only get called on an edge — when it changes. You shouldn’t see two back-to-back connected=true calls with the same callback.


I think you’re going to want to move those to xio.cpp, but you’re going to want to make a way for controller to get that generated info, I suspect.


As for reading and writing:

There are two SerialUSB objects: SerialUSB  (NO zero, but easily changed), and SerialUSB1. They are identical in function. From here on, when I say SerialUSB, I also mean SerialUSB1 — they’re identical.

The interface is simple:

```c++
int16_t SerialUSB.readByte()

uint16_t read(uint8_t *buffer, const uint16_t length); // Blocking for now

int32_t write(const uint8_t *data, const uint16_t length); // Also blocking
```

There is also `readSome()` and `writeSome()` functions that are non-blocking, but otherwise have the same call syntax. read/readSome and write/writeSome all return the amount they read or wrote, which is possibly 0 in the *Some cases.

We haven’t used the blocking read() yet, and the blocking write only blocks until another 256 (or 128 for dual USB) buffer becomes available and it can push the data to it. This is rarely very long.

`length` is required in all cases. I have fixed this in the Motate repo to allow length==0 to means until the terminating NULL, but that’s not ported yet, and porting it’s not practical yet. So, always give a value length (strlen(ptr) works).

readByte() will return -1 if there’s nothing to read. It’s been doing that this whole time, inside xio -> read_char(), which is in turn called by read_line(), where it’s called _FDEV_ERR and means the same thing.

I modified the _write() system call (that gets called from printf) to take file id == 1 as the SerialUSB1, and everything else as SerialUSB. It calls SerialUSB.write(), the blocking version.

## C++ Classes, Virtual Functions and Inheritance
This section provides an explanation of what's going on with the device classes, virtual functions and inheritance. It's intended for those who know C but not C++, or need a C++ refresher.

Starting at the beginning. We are constructing a C++ class for the primitive functions for a class. For this example we are only looking at reading characters from a device. 

```c++
struct xioDeviceWrapperBase {				// base class for the reading from a device
	virtual int16_t readchar() = 0;			// Pure virtual. Will be subclassed for every device
};
```

This class is the base class (parent class), and will be subclassed for each device instance. It has a virtual function for reading characters from the device. Each device subclass must provide its own function for reading which will override the virtual function in the base class.

The `= 0` portion of the readchar() definition makes it a _pure virtual function_ and also makes `xioDeviceWrapperBase` an _abstract base class_, which means you cannot instantiate a `xioDeviceWrapperBase` object directly. However, you can have a pointer to a `xioDeviceWrapperBase`, and since you cannot instantiate one that means that it _must_ point to a subclass. (See [this document](http://www.cplusplus.com/doc/tutorial/polymorphism/) for more info.)

The following is the template for the class definition for each device:

```c++
template<typename Device>
struct xioDeviceWrapper : xioDeviceWrapperBase {	// describes the functions such as reading for a given device
    Device _dev;

    xioDeviceWrapper(Device  dev) : _dev{dev} {};

    int16_t readchar() final {
        return _dev->readByte();
    };
};
```
Starting with the template declaration:

```c++
template<typename Device>
struct …
```

This says that whatever structure is defined after struct will take a template parameter that is a typename and it’ll be called Device. Device can be any valid type, such as char *, int, const int, or even other templated structures. Think of it like a macro, except it’s not a brain-dead copy-and-paste that happens in the preprocessor, but it’s part of the compiler and it can deal with it intelligently. (Obviously, macros still have their place…)

Now for the struct declaration:

<pre>
template<typename Device>
struct xioDeviceWrapper : xioDeviceWrapperBase {	// describes the functions such as reading for a given device
  …
};
</pre>

That says we have a template structure named xioDeviceWrapper that is a subclass (child) of xioDeviceWrapperBase and has the following definition (in {…}; like a normal structure). This means that whatever is defined in xioDeviceWrapperBase will also implicitly be defined in xioDeviceWrapper.

Now for the structure declaration, starting with:
<pre>
    Device _dev;
</pre>
We declare a variable (called a member) of type Device (our template parameter, so it could be anything, really, from a pointer to a serial usb object to a char) and name it _dev. When it comes time to compile, if we used _dev in a way that doesn’t make sense for a Device then we’ll get an error.

<This is where I get lost. What is this for? Is this the name or ID of the device? What would be a way that doesn't make sense to the compiler? I can guess and probably be right, but can you provide a correct and incorrect example?

This next part is really confusing. What is the constructor for - does it initialize the methods that follow? Is it callable by the user? Should it read something like "The constructor is required for every subclassed object. They run once to set up the object, and have no return value. This format for constructors is pretty standard, here's what it means:">

The next is a method definition, that is a special method called the constructor, and constructors have no return value. (It’s implied that it’s assisting it setting up a new object of this class.) Here it is:

<pre>
    xioDeviceWrapper(Device  dev) : _dev{dev} {};
</pre>
It also has a special syntax to allow simple member initialization after the : but before the body of the function {…}; In this case it’s _dev{dev} which basically says _dev = dev. (It could have been written _dev(dev) as well, but the {…} construction is a new thing that prevents implicit type conversion.)

<The above is confusing because I don't know where '_dev' came from or what it does, or what 'dev' is. Is it something that is substituted later? Is it generic? What does _dev = dev mean? Assign the value on dev to _dev, presumably, what not knowing what either of these are it's a bit abstract. I surmise from later text that _dev is a self reference to the object like Python's self, but you get to define it.>

We don’t want to do anything else in this constructor so we define the body here and the body is empty: {};

Now we have a member function, which is called a method:
<pre>
    int16_t readchar() final {
        return _dev->readByte();
    };
</pre>
It’s a pretty normal function, except for a few things:

* final in the declaration means that it’s a virtual function that overrides the base class’s virtual function. We could have declared it virtual, but final gets us more protection since the compiler will error if the base class doesn’t have this function declared virtual OR any subclass tries to override this one.

* We use the variable _dev which implies the member _dev in this instance of the xioDeviceWrapper class. If we have an xioDeviceWrapper object called w, then when we call w.readchar() it will use w._dev.
<I'm not sure what this means. Can you explain and/or provide an example? What is "an xioDeviceWrapper object called w"?>

* This allows for encapsulation — the external interface doesn’t have to know about any of the internal variables.

* Another thing to note is that if Device isn’t a pointer type or doesn’t have a readByte() method, then there will be an error at compile time.
Where was the device declared as a pointer type, or is it just always that way?


####More about Virtual Functions and Inheritance
Inheritance without virtual functions is basically the same as dumping the parent class definitions into the child class definitions. The child class can then use the variables and call the methods of the parent class as if they had been defined in the child class directly.

There’s the added benefit that the compiler knows that they are related, so a pointer to the base class can point to the subclasses as well.

<Example? I'm not even sure how one references the parent and subclasses to define the pointer. Assume zero background in C++.>

However, that’s not worth much without virtual functions. Since the pointer refers to the base class, calling a method of that pointer will use the base-class function.

For example, if we have a type called Shape and subclasses for various shapes (Circle, Rectangle, etc), we can to this:
<pre>
struct Shape {
    void draw() {};
};
struct Circle : Shape {
    void draw() {};
};

Shape *nextShapeToDraw;

void drawNextShape() {
    nextShapeToDraw->draw();
}
</pre>
drawNextShape() will always call Shape::draw(), since nextShapeToDraw is a pointer to a Shape object, even if it’s actually pointing to a Circle.

<That's a bit confusing. We might need to walk through this. So I assume that the pointer to the Shape object set up in drawNextShape point to the base class? It might be clearer to call it ShapeBase or ShapeParent. Is there a C++ naming convention that differentiates the parents from the subclasses?>

Virtual functions fix that:
<pre>
struct Shape {
    virtual void draw() = 0; // Error if called directly
};
struct Circle : Shape {
    void draw() {}; // virtual implied here
};

Shape *nextShapeToDraw;

void drawNextShape() {
    nextShapeToDraw->draw();
}
</pre>
Now if nextShapeToDraw is assigned to the pointer of a Circle object, it will call Circle::draw(). 
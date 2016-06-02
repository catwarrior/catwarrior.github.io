## Concept
### In the most fundamental sense, a driver is a software component that lets the operating system and a device communicate with each other.
Our explanation so far is oversimplified in several ways:
- Not all drivers have to be written by the company that designed the device. In many cases, a device is designed according to a published hardware standard. This means that the driver can be written by Microsoft, and the device designer does not have to provide a driver.
- Not all drivers communicate directly with a device. For a given I/O request (like reading data from a device), there are often several drivers, layered in a stack, that participate in the request. The conventional way to visualize the stack is with the first participant at the top and the last participant at the bottom, as shown in this diagram. Some of the drivers in the stack might participate by transforming the request from one format to another. These drivers do not communicate directly with the device; they just manipulate the request and pass the request along to drivers that are lower in the stack.
- The one driver in the stack that communicates directly with the device is called the function driver; the drivers that perform auxiliary processing are called filter drivers.
Some filter drivers observe and record information about I/O requests but do not actively participate in them. For example, certain filter drivers act as verifiers to make sure the other drivers in the stack are handling the I/O request correctly.
We could expand our definition of driver by saying that a driver is any software component that observes or participates in the communication between the operating system and a device.

### A driver is any software component that observes or participates in the communication between the operating system and a device.
https://msdn.microsoft.com/en-us/library/windows/hardware/ff554678(v=vs.85).aspx

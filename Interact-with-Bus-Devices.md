So you just connected to your interface and want to send your first telegrams?  
First i have to tell you about the two types of connections:
- Send data to all device who listen to this groupaddress
- Send data to a specific device (MemoryWrite, DescriptionRead, etc)
  
## Send to multiple devices with groupaddress
To send data to multiple devices with a groupaddress you have to use the BusCommon class.
In the example below you already connected to an interface.  
```C#
IKnxConnection _conn; //An Interface which you already connected 
BusCommon bus = new BusCommon(_conn);
await bus.GroupValueWrite(MulticastAddress.FromString("1/4/3"), 0x1);
await bus.GroupValueWrite(MulticastAddress.FromString("1/4/4"), new byte[] { 0x1, 0x23 });
```
If you dont want to handle conversion by yourself, you can use the DptConverter class.
```C#
DptConverter.ToByteArray(DPTs.DPT5, 15)
DptConverter.ToByteArray(DPTs.DPT1, true)
int position = DptConverter.FromByteArray<string>(DPTs.DPT5, message.Raw);
bool state = DptConverter.FromByteArray<string>(DPTs.DPT1, message.Raw);
```
There is also a way to search for devices with prog mode on
```C#
await bus.IndividualAddressRead();
//And write new PA if one device has prog mode on
await bus.IndividualAddressWrite(UnicastAddress.FromString("1.1.6"));
```

## Send to a single Device
To send data to a single device you will have to know its physical address (1.1.3 for example).  
In the example below you already connected to an interface.  
```C#
IKnxConnection _conn; //An Interface which you already connected with.
BusDevice device = new BusDevice("1.1.3", _conn);
//Connect will also read MaskVersion and MaxAPDULength
//You can disable read MaxAPDULength with bool Parameter = false
await device.Connect();
[...]
await device.Disconnect();
```
And you are connected with the device. You can now send telegrams to it.
If your device is protected by a key you have to authorize.
```C#
MsgAuthorizeRes resauth = await device.Authorize((uint)0xAABBCCDD);
```
Or if you want to read/write data.
If data length is more than the telegram can handle multiple telegrams will be send. 
```C#
//To write data
await device.MemoryWrite(13021, new byte[] { 0x01, 0x02, 0x03, 0x04 });
byte[] data = await device.MemoryRead(13021, 4);
```
You can also read and write Properties:
```C#
//Read Property and convert it to any supportet type
//Supported: string, int, uint, byte[]
int maxapdu = await device.PropertyRead<int>(0, 56);
```
Reading resources (DeviceSerialNumber, Manufacturer) so you dont have to handle if it's stored in an property or in memory.
```C#
//Returns SerialNumber of Device
string serial = await dev.ResourceRead<string>("DeviceSerialNumber");
//Returns SerialNumber of Device as Byte Array
byte[] serialbytes = await dev.ResourceRead("DeviceSerialNumber");
```
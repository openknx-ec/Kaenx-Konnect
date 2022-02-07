There are currently three type of Interfaces:
 - [IP-Interface](#ip-interface)
 - [IP-Router](#ip-router)
 - [USB-Interface](#usb-interface)

All Interfaces have the same four Events:
- OnTunnelRequest: 
  Is Invoked when the Interface receives a Request. (GroupValueWrite, IndividualAddressRead, etc.)
- OnTunnelResponse: 
  Is Invoked when the Interface receives a Response. (GroupValueReadResponse, MemoryReadResponse, etc.)
- OnAck:
  Is Invoked when the Interface receives an Ack.
- OnSearchResponse:
  Is Invoked when the Interface receives a SearchResponse.

## IP-Interface
You can connect to an IP-Interface by using IP-Tunneling.
```C#
IKnxConnection _connIp = new KnxIpTunneling("192.168.0.108", 3671);
await _connIp.Connect();
await Task.Delay(5000);
await _connIp.Disconnect();
```
If you already have the IPEndPoint you can create the connection like this:
```C#
IPEndPoint endpoint = new IPEndPoint(IPAddress.Parse("192.168.0.108"), 3671);
IKnxConnection _connIp = new KnxIpTunneling(endpoint);

```

## IP-Router
To connect to an IP-Router you don't need the ip or port.  
You also don't need the methods connect or disconnect.
```C#
IKnxConnection _connIp = new KnxIpRouting(); //Will connect to broadcast Ip "224.0.23.12"
//await _connIp.Connect(); Method Connect won't have any affect
//await _connIp.Disconnect(); Will also have no affect
```

## USB-Interface
```C#
ConnectedDeviceDefinition def; //See Device.Net on how to get ConnectedDeviceDefinition
IKnxConnection _connUsb = new KnxUsbTunneling(def);
await _connUsb.Connect();
await Task.Delay(5000);
await _connUsb.Disconnect();
```
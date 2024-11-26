## Interface Events
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
IKnxConnection _connIp = new KnxIpRouting(); // Will broadcast to "224.0.23.12"
await _connIp.Connect(); // start handling received telegrams
await _connIp.Disconnect(); // stop handling received telegrams
```

## USB-Interface
```C#
ConnectedDeviceDefinition def; // See Device.Net on how to get ConnectedDeviceDefinition
IKnxConnection _connUsb = new KnxUsbTunneling(def);
await _connUsb.Connect();
await Task.Delay(5000);
await _connUsb.Disconnect();
```
> For now you will have to open ETS to make it work.  
Often you can't connect to the usb interface without ets.

## Search for Interfaces
You can search for Interfaces in your Local Network.  
Please use IP-Routing for searches (do not connect with IP-Tunneling to broadcast IP).
```C#
KnxIpSearch _connIp = new KnxIpSearch();
_connIp.OnSearchResponse += HandleSearchResponse;
_connIp.Search();
```
```C#
private void HandleSearchResponse(MsgSearchResp message)
{
    Debug.WriteLine($"Found: {message.FriendlyName} (PA: {message.PhAddr.ToString()})");
}
```
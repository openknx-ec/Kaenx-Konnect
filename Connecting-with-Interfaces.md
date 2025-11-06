## Interface Events
All Interfaces have the same four Events:
- OnReceivedMessage: 
  Is Invoked when the Interface receives a LDataMessage. (GroupValueWrite, IndividualAddressRead, etc.)
- OnReceivedService: 
  Is Invoked when the Interface receives an IpServiceTelegram. (TunnelConnectRequest, SearchResponse, etc.)


## IP-Interface
You can connect to an IP-Interface by using IP-Tunneling.
```C#
IpKnxConnection conn = KnxFactory.CreateTunnelingUdp("192.168.178.133", 3671);
await _connIp.Connect();
Debug.WriteLine($"Connected with Interface with PA {conn.GetLocalAddress() ?? "unknown"}");
Debug.WriteLine($"Telegrams can use a MaxAPDU of {conn.GetMaxApduLength()}");
await Task.Delay(5000);
await _connIp.Disconnect();
// Create a tunnel with tcp
IpKnxConnection connTcp = KnxFactory.CreateTunnelingTcp("192.168.178.133", 3671);
```
If you already have the IPEndPoint you can create the connection like this:
```C#
IPEndPoint endpoint = new IPEndPoint(IPAddress.Parse("192.168.0.108"), 3671);
IpKnxConnection conn = KnxFactory.CreateTunnelingUdp(endpoint);
```

## IP-Router
To connect to an IP-Router you don't need the ip or port.  
You also don't need the methods connect or disconnect.
```C#
IKnxConnection _connIp = new KnxIpRouting(); // Will broadcast to "224.0.23.12"
IpKnxConnection conn = KnxFactory.CreateRouting(UnicastAddress.FromString("0.0.1")); // Will broadcast to "224.0.23.12"
// or if you want to use another multicastaddress
IpKnxConnection conn2 = KnxFactory.CreateRouting(UnicastAddress.FromString("0.0.1"), "224.0.23.13", 6521); // Will broadcast to "224.0.23.13" at Port 6521
await _connIp.Connect(); // start handling received telegrams
await _connIp.Disconnect(); // stop handling received telegrams
```

## USB-Interface 
> From Version 2.0.0 not supported yet  
  
```C#
// ConnectedDeviceDefinition def; // See Device.Net on how to get ConnectedDeviceDefinition
// IKnxConnection _connUsb = new KnxUsbTunneling(def);
// await _connUsb.Connect();
// await Task.Delay(5000);
// await _connUsb.Disconnect();
```
> For now you will have to open ETS to make it work.  
Often you can't connect to the usb interface without ets.

## Search for Interfaces
You can search for Interfaces in your Local Network.  
Please use IP-Routing for searches (do not connect with IP-Tunneling to broadcast IP).
```C#
IpKnxConnection _conn = KnxFactory.CreateTunnelingUdp(new(IPAddress.Parse("224.0.23.12"), 3671));

_conn.OnReceivedService += HandleSearchResponse;

SearchRequest req = new SearchRequest(_conn.GetLocalEndpoint());
await _conn.SendAsync(req);
await Task.Delay(500); // wait for Interfaces to respond
_conn.Dispose();
```
```C#
private void HandleSearchResponse(IpTelegram message)
{
    if (message.ServiceIdentifier != Kaenx.Konnect.Enums.ServiceIdentifiers.SearchResponse)
        return;

    Debug.WriteLine($"Found: {message.FriendlyName} (PA: {message.PhAddr.ToString()})");
    if (svcFamilies.GetServiceFamilyVersion(Kaenx.Konnect.Enums.ServiceFamilies.Tunneling) > 0)
    {
        int tunnelingVersion = svcFamilies.GetServiceFamilyVersion(Kaenx.Konnect.Enums.ServiceFamilies.Tunneling);
        Console.WriteLine($" - Tunneling v{tunnelingVersion} -> {hpai.Endpoint,-20} ({deviceInfo.UnicastAddress,-9}) [{deviceInfo.FriendlyName}]");
    }

    if (svcFamilies.GetServiceFamilyVersion(Kaenx.Konnect.Enums.ServiceFamilies.Routing) > 0)
    {
        int routingVersion = svcFamilies.GetServiceFamilyVersion(Kaenx.Konnect.Enums.ServiceFamilies.Routing);
        Console.WriteLine($" - Routing   v{routingVersion} -> {hpai.Endpoint,-20} ({deviceInfo.UnicastAddress,-9}) [{deviceInfo.FriendlyName}]");
    }
}
```
---
layout: Meadow
title: Networking
subtitle: Network options and operation.
---

Both the Meadow F7 development board and SMT module have WiFi networking via the ESP32 co-processor. The Meadow F7 embedded SMT module also adds optional ethernet capabilities.

## Current Limitations

Beta 4.0 introduces the first version of our networking stack for Meadow and there are a few known limitations to be aware of when using:

 * **SSL not supported** - Currently, only unencrypted traffic is supported via HTTP. We're working on TLS support.
 * **`HttpServer` not available** - `HttpClient` has been tested and is fully supported, but we're still working on `HttpServer` support.
 * **Network scans require a network connection** - Due to a limitation in the API, you must first connect to a WiFi network before attempting a scan.
 * **`HttpClient` memory leak** - Each `HttpClient.Request` incurs a `~5k` memory leak on the ESP32 coprocessor, so after a number of requests it will no longer be functional. If requests become unresponsive, a call to `Device.InitWiFiAdapter()` will reset the ESP32 coprocessor and get things running again. Note that requests performed low-level socket calls do not incur this memory leak.

# WiFi

## Initializing the `WiFiAdapter`

In order to use wifi networking, you must first initialize the `WiFiAdpater` by calling `InitWiFiAdapter()` on the `F7Micro` device:

```csharp
Device.InitWiFiAdapter().Wait();
```

The intialization method can take 5 or more seconds, and is awaitable.

Once initialized, the `WiFiAdapter` is available as a property on the `F7Micro` class and can be accessed via the `Device` property in your app class:

```csharp
// from within your app class:
Device.WiFiAdapter

// from other classes, where [MeadowApp] is the name of your app class:
[MeadowApp].Device.WiFiAdapter
```

## Connecting to a WiFi Network

Once the `WiFiAdapter` has been initialized, you can connect to a network by calling the `Connect` method and passing in the SSID (network name), and password:

```csharp
if (Device.WiFiAdapter.Connect("SSID", "Pass").ConnectionStatus != ConnectionStatus.Success) {
    throw new Exception("Cannot connect to network, applicaiton halted.");
}
```

## Scanning for WiFi Networks

You can also can for WiFI networks, however, due to a temporary limitation in the current API, you must first be connected to a network.

To scan, call the `Scan()` method on the `WiFiAdapter` and then access the network list via the `Networks` `ObservableCollection` property:

```csharp
protected void ScanForAccessPoints()
{
    Console.WriteLine("Getting list of access points.");
    Device.WiFiAdapter.Scan();
    if (Device.WiFiAdapter.Networks.Count > 0) {
        Console.WriteLine("|-------------------------------------------------------------|---------|");
        Console.WriteLine("|         Network Name             | RSSI |       BSSID       | Channel |");
        Console.WriteLine("|-------------------------------------------------------------|---------|");
        foreach (WifiNetwork accessPoint in Device.WiFiAdapter.Networks) {
            Console.WriteLine($"| {accessPoint.Ssid,-32} | {accessPoint.SignalDbStrength,4} | {accessPoint.Bssid,17} |   {accessPoint.ChannelCenterFrequency,3}   |");
        }
    } else {
        Console.WriteLine($"No access points detected.");
    }
}
```

If you attempt to scan for networks without being first connected, an `Exception` will be thrown. In the future this requirement will be removed.

# Performing Requests

Once the network is connected, you can generally use the built-in .NET network methods as usual, however `HttpServer` is not availble in this beta.

## `HttpClient.Request()` Example

The following code illustrates making a request to a web page via the `HttpClient` class:

```csharp
using (HttpClient client = new HttpClient()) {
    HttpResponseMessage response = await client.GetAsync(uri);
    response.EnsureSuccessStatusCode();
    string responseBody = await response.Content.ReadAsStringAsync();
    Console.WriteLine(responseBody);
}
```

### Posting

You can also modify the request to `POST` data. For example, the following code posts a temperature reading to Adafruit.IO:

```csharp
using (HttpClient client = new HttpClient()) {
    client.DefaultRequestHeaders.Add("X-AIO-Key", [APIO_KEY]);
    client.Timeout = TimeSpan.FromMinutes(10);
    string temperature = "23.70";
    string data = "{\"value\":\"" + temperature + "\"}";
    var content = new StringContent(data, Encoding.UTF8, "application/json");
    var result = client.PostAsync(uri, content).Result;
}
```

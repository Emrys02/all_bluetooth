# all_bluetooth

A Flutter plugin for implementing Bluetooth Classic features on Android. This plugin provides comprehensive support for Bluetooth communication, device discovery, server/client connections, and data transfer.

## Features

- 🔍 **Device Discovery** - Scan for and discover nearby Bluetooth devices
- 📱 **Bonded Devices** - Retrieve a list of previously paired devices
- 🔌 **Client/Server Connections** - Connect as either a client or server
- 📡 **Data Transfer** - Send and receive data over Bluetooth connections
- 📊 **State Management** - Monitor Bluetooth state (on/off) via Future or Stream
- 🎯 **Device Advertising** - Make your device discoverable
- 📝 **Bluetooth Name** - Get and set your device's Bluetooth name
- 🔔 **Real-time Events** - Listen to connection state and data streams

## Platform Support

| Platform | Supported |
|----------|-----------|
| Android  | ✅        |
| iOS      | ❌        |

## Installation

Add `all_bluetooth` as a dependency in your `pubspec.yaml` file:

```yaml
dependencies:
  all_bluetooth: ^1.0.0
```

Then run:

```bash
flutter pub get
```

## Android Setup

Add the following permissions to your `AndroidManifest.xml` file:

```xml
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
```

**Important:** You must request these permissions at runtime. We recommend using the [`permission_handler`](https://pub.dev/packages/permission_handler) package to handle runtime permission requests.

## Quick Start

Create an instance in your app:

```dart
final allBluetooth = AllBluetooth();
```


## Core Concepts

### BluetoothDevice

Represents a Bluetooth device with the following properties:

- `name` (String) - The device name
- `address` (String) - The device MAC address
- `bonded` (bool) - Whether the device is bonded/paired

### ConnectionResult

Represents the state of a Bluetooth connection, emitted by the `listenForConnection` stream:

- `state` (bool) - Whether connected or not
- `response` (String) - Response message
- `device` (BluetoothDevice?) - The connected Bluetooth device (when successful)

## Usage

Import the package:

```dart
import 'package:all_bluetooth/all_bluetooth.dart';
```

### Basic Workflow

1. **Check Bluetooth State** - Ensure Bluetooth is enabled before performing operations
2. **Listen for Connections** - Use `listenForConnection` stream to monitor connection status
3. **Perform Operations** - Connect, discover devices, send/receive data
4. **Handle Data** - Use `listenForData` stream for receiving messages

> **Note:** Most functions require an active Bluetooth connection. Always monitor the connection state using `listenForConnection` stream.

## API Reference

### Methods

#### closeConnection()

Closes the current Bluetooth connection and releases all associated resources. Also used to close a Bluetooth server.

```dart
await allBluetooth.closeConnection();
```

#### connectToDevice(String address)

Connects to a Bluetooth device using its MAC address. The target device must be running as a server.

```dart
await allBluetooth.connectToDevice('00:11:22:33:44:55');
```

> **Note:** Monitor the connection result via `listenForConnection` stream, as the actual connection state is handled by a broadcast receiver.

#### getBondedDevices()

Returns a list of previously paired/bonded Bluetooth devices.

```dart
final devices = await allBluetooth.getBondedDevices();
// Example output:
// [
//   BluetoothDevice(name: "Device 1", address: "DA:4V:10:CE:17:00", bonded: true),
//   BluetoothDevice(name: "Device 2", address: "DA:5V:10:CE:17:00", bonded: true),
// ]
```

#### isBluetoothOn()

Returns whether Bluetooth is currently enabled.

```dart
final isOn = await allBluetooth.isBluetoothOn();
```

#### getBluetoothName()

Returns the current Bluetooth name of your device.

```dart
final name = await allBluetooth.getBluetoothName();
```

#### changeBluetoothName(String name)

Changes your device's Bluetooth name. Returns `true` if successful, `false` otherwise.

```dart
final success = await allBluetooth.changeBluetoothName('MyDevice');
```

#### startBluetoothServer()

Opens a server socket to accept client connections. The socket remains open until explicitly closed with `closeConnection()`.

```dart
await allBluetooth.startBluetoothServer();
```

> **Note:** Monitor connection attempts via `listenForConnection` stream.

#### sendMessage(List<int> message)

Sends data over an established Bluetooth connection. Data is sent as raw bytes. Returns `true` if successful, `false` otherwise.

```dart
final message = 'Hello Bluetooth'.codeUnits;
final success = await allBluetooth.sendMessage(message);
```

> **Security Tip:** Consider implementing your own encryption for sensitive data.

#### startDiscovery()

Starts discovering nearby Bluetooth devices. Use with the `discoverDevices` stream to receive discovered devices.

```dart
await allBluetooth.startDiscovery();
```

If discovery isn't working as expected, call `stopDiscovery()` followed by `startDiscovery()` again.

#### stopDiscovery()

Stops the device discovery process.

```dart
await allBluetooth.stopDiscovery();
```

#### startAdvertising({int? secondDuration})

Makes your device discoverable to other Bluetooth devices. Should be used together with `startBluetoothServer()`.

```dart
await allBluetooth.startAdvertising(secondDuration: 300); // 5 minutes
```

### Streams

#### discoverDevices

Listens for newly discovered Bluetooth devices. Emits one device at a time.

```dart
allBluetooth.discoverDevices.listen((device) {
  print('Discovered: ${device.name} at ${device.address}');
  scannedDevices.add(device);
});
```

#### listenForConnection

Listens for Bluetooth connection state changes. Emits `ConnectionResult` objects.

```dart
allBluetooth.listenForConnection.listen((result) {
  if (result.state) {
    print('Connected to ${result.device?.name}');
  } else {
    print('Disconnected: ${result.response}');
  }
});
```

> **Best Practice:** Wrap your app with a StreamBuilder listening to this stream for proper connection management.

#### listenForData

Listens for incoming data over the Bluetooth connection. Emits raw byte data as `List<int>`.

```dart
allBluetooth.listenForData.listen((data) {
  final message = String.fromCharCodes(data);
  print('Received: $message');
});
```

> **Important:** Only use this when you have an active connection (verified via `listenForConnection`).

#### streamBluetoothState

Monitors the Bluetooth adapter state (enabled/disabled). Emits `true` when Bluetooth is on, `false` when off.

```dart
allBluetooth.streamBluetoothState.listen((isOn) {
  print('Bluetooth is ${isOn ? 'ON' : 'OFF'}');
});
```

## Complete Example

```dart
import 'package:flutter/material.dart';
import 'package:all_bluetooth/all_bluetooth.dart';

class BluetoothScreen extends StatefulWidget {
  @override
  _BluetoothScreenState createState() => _BluetoothScreenState();
}

class _BluetoothScreenState extends State<BluetoothScreen> {
  final allBluetooth = AllBluetooth();
  List<BluetoothDevice> devices = [];
  bool isConnected = false;

  @override
  void initState() {
    super.initState();
    
    // Listen for connection changes
    allBluetooth.listenForConnection.listen((result) {
      setState(() {
        isConnected = result.state;
      });
      if (result.state) {
        print('Connected to: ${result.device?.name}');
      }
    });

    // Listen for incoming data
    allBluetooth.listenForData.listen((data) {
      final message = String.fromCharCodes(data);
      print('Received: $message');
    });

    // Get bonded devices
    _loadBondedDevices();
  }

  Future<void> _loadBondedDevices() async {
    final bondedDevices = await allBluetooth.getBondedDevices();
    setState(() {
      devices = bondedDevices;
    });
  }

  Future<void> _connectToDevice(String address) async {
    await allBluetooth.connectToDevice(address);
  }

  Future<void> _sendMessage(String message) async {
    if (isConnected) {
      final success = await allBluetooth.sendMessage(message.codeUnits);
      print('Message sent: $success');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Bluetooth Devices')),
      body: ListView.builder(
        itemCount: devices.length,
        itemBuilder: (context, index) {
          final device = devices[index];
          return ListTile(
            title: Text(device.name),
            subtitle: Text(device.address),
            trailing: ElevatedButton(
              onPressed: () => _connectToDevice(device.address),
              child: Text('Connect'),
            ),
          );
        },
      ),
    );
  }

  @override
  void dispose() {
    allBluetooth.closeConnection();
    super.dispose();
  }
}
```

## Important Notes


## Important Notes

- **Bidirectional Communication:** It doesn't matter if you connect as client or server—both parties can send and receive data. The client/server distinction only matters for establishing the connection.

- **Connection Monitoring:** Always use `listenForConnection` stream with `startBluetoothServer()` and `connectToDevice()` to monitor connection success/failure. Connection results are handled by a broadcast receiver, not the methods themselves.

- **Active Connection Required:** Methods like `listenForData` and `sendMessage` require an active Bluetooth connection to work properly.

- **Permission Handling:** Ensure you request and handle Bluetooth permissions properly before using this plugin.

## Troubleshooting

### Device Discovery Not Working

If you're not seeing discovered devices:
1. Call `stopDiscovery()`
2. Wait a moment
3. Call `startDiscovery()` again
4. Ensure location permissions are granted (required for Bluetooth scanning on Android)

### Connection Issues

- Verify Bluetooth is enabled on both devices
- Ensure one device is running as a server before the client attempts to connect
- Check that all required permissions are granted
- Monitor `listenForConnection` stream for detailed connection status

## Contributions and Support

Contributions, issues, and feature requests are welcome!

- **Issues:** [GitHub Issues](https://github.com/rans-rans/all_bluetooth/issues)
- **Pull Requests:** Feel free to contribute code improvements
- **Contact:** ransfordowusuansah9@gmail.com

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Author

Created and maintained by Ransford Owusu Ansah

---

**Note:** This plugin currently only supports Android. iOS support may be added in future releases.

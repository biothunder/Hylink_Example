# Hylink_Example

A library that connects to the console.

# Usage
Add credentials to your `build.gradle` in module:
```groovy
repositories {
    maven {
        url 'https://pkgs.dev.azure.com/hyenatek/_packaging/Hylink/maven/v1'
        credentials {
            username "AZURE_ARTIFACTS"
            password "7flkyc5rhwriyt4wtbhyw6cpapzhso3t2oeq6547g3eb6ow32akq"
        }
    }
}
```
The generated token(password) expires on or before 2019/10/27

Add a dependency to your `build.gradle` in module:
```groovy
dependencies {
    implementation(group: 'com.hyena.hylink', name: 'example', version: '1.0.0')
}
```

# Features
- Scan and filter with uuids.
- Connect and disconnect.
- Continuously receiving data.
- One-time inquiry data.

# Sample usage - Scan and filter with uuids
![alt tag](https://media.giphy.com/media/daPFmDWynBAZij4hF7/source.gif)

Before scan, check thast you have ACCESS_COARSE_LOCATION.
```groovy
final String[] permissions = {android.Manifest.permission.ACCESS_COARSE_LOCATION};
        if (EasyPermissions.hasPermissions(this, permissions)) {
            startScan();
        } else {
            requestPermission(permissions);
        }
```

Implements HyLink.Listener and declare your hyLinkListener to override hyLinkScan method.
You can update scanned device to ui when hyLinkScan received new hyDevice.
```groovy
public class HyLinkListenerAdapter implements HyLink.Listener {

    @Override
    public void hyLinkScan(HyDevice hyDevice, int i) {
        // empty default implementation
    }

    @Override
    public void hyLinkDidReceived(HyDevice hyDevice, HyDataType hyDataType, HyDataInfo hyDataInfo) {
        // empty default implementation
    }

    @Override
    public void hyLinkDidReadRSSI(HyDevice hyDevice, int i) {
        // empty default implementation
    }

    @Override
    public void hyLinkDidDisconnected(HyDevice hyDevice) {
        // empty default implementation
    }
}

private HyLink.Listener hyLinkListener = new HyLinkListenerAdapter() {
        @Override
        public void hyLinkScan(HyDevice hyDevice, int RSSI) {
            super.hyLinkScan(hyDevice, RSSI);
            onHyDeviceScanned();
        }
    };
```

Then add your hyLinkListener in HyLink to receive hyLinkScan event.
```groovy
final HyLink hyLink = HyLink.getInstance(this);
        hyLink.addListener(hyLinkListener);
```

Start scan with no uuid filter.
```groovy
HyLinkError result = HyLink.getInstance(context).startScan()
```

Or set timeout and filter by GATT service uuids.
```groovy
UUID[] uuids = {UUID.fromString(BLE_GATT_SERVICE_UUID_CROCO_H),
                UUID.fromString(BLE_GATT_SERVICE_UUID_GORILLA),
                UUID.fromString(BLE_GATT_SERVICE_UUID_DFU)};
HyLinkError result = HyLink.getInstance(this).startScan(uuids, 0);
```

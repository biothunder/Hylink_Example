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

Minimal required sdk version is 18
```groovy
minSdkVersion 18
```

# Features
- Scan.
- Connect and disconnect.
- Continuously receiving data.
- One-time inquiry data.

# Sample usage - Scan.
![alt tag](https://media.giphy.com/media/daPFmDWynBAZij4hF7/source.gif)

Check permission before scan.(Android 6.0 need ACCESS_COARSE_LOCATION or ACCESS_FINE_LOCATION)
```groovy
final String[] permissions = {android.Manifest.permission.ACCESS_COARSE_LOCATION};
        if (EasyPermissions.hasPermissions(this, permissions)) {
            startScan();
        } else {
            requestPermission(permissions);
        }
```

Implements HyLink.Listener and declare your hyLinkListener to override hyLinkScan method.Most of the features require this 
hyLinkListener.

You can update scanned device to ui when hyLinkScan received new hyDevice scanned callback.
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
            //update scanned device to ui
        }
    };
```

Then add your hyLinkListener in HyLink to receive hyLinkScan callback.
```groovy
final HyLink hyLink = HyLink.getInstance(context);
        hyLink.addListener(hyLinkListener);
```

Now you can start scan with call HyLink.getInstance(context).startScan()
```groovy
HyLinkError result = HyLink.getInstance(context).startScan();
```

# Sample usage - Connect and disconnect.
Tap the device name in the list to connect.

![alt tag](https://media.giphy.com/media/iGdcAlHhOotxsQuEuz/source.gif)

HyLink.getInstance(this).connectDevice is an async function, it will return connect is success or not.
```groovy
boolean connectSuccess = HyLink.getInstance(context).connectDevice(device, new HyLink.ConnectCallback() {
                    @Override
                    public void completion(boolean success, int status) {
                        Log.v(TAG, "connect " + device.name + " - " + (success ? "Success" : "Failed"));
                        progressDialog.setVisibility(View.INVISIBLE);
                        if (success) {
                            //connect success
                        } else {
                            //connect failed
                        }
                    }
                });
```

After connected success, stored connected device for later use.
```groovy
private void didConnected(HyDevice device) {
        Log.d(TAG, "didConnected " + device.name);
        stopScan();
        DeviceManager.instance.connectedDevice = device;
    }
```

If you want to disconnect, call HyLink.getInstance(this)disconnectDevice.

Don't forget to check is need to disconnect when you close the app.
```groovy
@Override
    protected void onDestroy() {
        super.onDestroy();
        HyDevice device = DeviceManager.instance.connectedDevice;
        if (device != null) {
            DeviceManager.instance.connectedDevice = null;
            HyLink.getInstance(context).disconnectDevice(device);
        }
    }
```

If a device is disconnected, your hyLinkListener will receive hyLinkDidDisconnected event.
```groovy
@Override
    public void hyLinkDidDisconnected(HyDevice device) {
        Log.i(TAG, "hyLinkDidDisconnected " + device.name);
        //device is disconnected
    }
```

# Sample usage - Continuously receiving data.
![alt tag](https://media.giphy.com/media/KzL07fpAcXhjBWeT8Q/source.gif)

hyLinkDidReceived callback in your hyLinkListener will receive data per 0.6 second.

Data by HyDataInfo contains information from console(display) and driver.They contain current speed, battery capacity, odo, light on/off ...etc information.
```groovy
@Override
        public void hyLinkDidReceived(HyDevice device, HyDataType type, HyDataInfo data) {
            switch (type) {
                case DISP: //update display info to ui
                break;
                case DRV:  //update driver info to ui
                break;
                case ERR_MSG: break;
            }
        }
```

# Sample usage - One-time inquiry data.
![alt tag](https://media.giphy.com/media/VJr2BDqKTp0HIzq89z/source.gif)

We can ask information from the console or driver in an async way.

```groovy
private void getPartNumber() {
        final HyDevice device = DeviceManager.instance.connectedDevice;
        if (device != null) {
            device.consoleParams.getPartNumber(new ReadStringCallback() {
                @Override
                public void completion(boolean success, String partNumber) {
                    if (success) {
                        textViewPartNumber.setText(partNumber);
                    } else {
                        textViewPartNumber.setText(getString(R.string.failed));
                    }
                }
            });
        }
    }
```

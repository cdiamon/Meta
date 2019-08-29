# BtleDemo
Ready to use Bluetooth Low Energy scanner

## Getting started

### Setting up the dependency

The first step is to download [**beaconblescanner**](https://github.com/cdiamon/BtleDemo/tree/master/beaconblescanner) module of this project.

Then import module into your project as described [here](https://stackoverflow.com/a/41764425/7331042)

check these lines in app-level **build.gradle**
```groovy
dependencies {
    implementation project(':beaconblescanner')
}
```
check if module added in **settings.gradle**
```groovy
include ':app', ':beaconblescanner'
```



### Hello World

Easy steps of starting background scanner:

#### Add `service` to app `manifest`
```xml
<service android:name="com.padmitriy.beaconblescanner.BtCesarService" />
```

#### Add permissions to `manifest` 
```xml
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
```


#### Make sure of handling all needed *runtime permissions*

#### If starting `service` from `activity`:

Create `BtServiceLaunchData` object with next example parameters from server:
```kotlin
import com.padmitriy.beaconblescanner.BtServiceLaunchData

/**
 *  `blockAddress` has to be formatted like "FE:FF:1E:36:B1:E0"
 *  `parametersServiceType` block type: 0 for cs9, 1 for cs12t
 *  `authorizationSecret` key to find authorization code
 */
val btServiceLaunchData = BtServiceLaunchData(
            blockAddress,
            parametersServiceType,
            authorizationSecret
        )
 ```


Launch service with `Intent`:
```kotlin
import com.padmitriy.beaconblescanner.BtService

val intent = BtService.getBtServiceIntent(this, btServiceLaunchData)

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    startForegroundService(intent)
} else {
    startService(intent)
}
```


#### If your want to receive events from service, you have to register broadcast receiver in runtime with `LocalBroadcastManager`, for example:

```kotlin
    override fun onStart() {
        super.onStart()
        registerReceiver()
    }
    
    // Our handler for received Intents. This will be called whenever an Intent
    // with an action named "BROADCAST_EXTRA_STRING" is broadcasted.
    private val mMessageReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            // Get extra data included in the Intent
            val message = intent.getStringExtra(BtService.BROADCAST_EXTRA_STRING)
            mainText.text = message
        }
    }

    private fun registerReceiver() {
        // Register to receive messages.
        // We are registering an observer (mMessageReceiver) to receive Intents
        // with actions named "BROADCAST_BT_ACTION".
        LocalBroadcastManager.getInstance(this).registerReceiver(
            mMessageReceiver,
            IntentFilter(BtService.BROADCAST_BT_ACTION)
        )
    }
    
    override fun onStop() {
        super.onStop()
        // Unregister since the activity is about to be closed.
        LocalBroadcastManager.getInstance(this).unregisterReceiver(mMessageReceiver)
    }
```

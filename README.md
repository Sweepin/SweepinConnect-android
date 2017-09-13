<p align="center" >
  <img src="Images/logoSweepinConnect-850x300.png" alt="SweepinConnectLogo" title="SweepinConnectLogo">
</p>

SweepinConnect is a location-based marketing tool for Android. By integrating our system in your application, you are quickly ready to go with the **iBeacon technology**, along with the geofencing system.

Our solution is linked to an intuitive back-office interface, <a href='http://manager.sweepin.fr/admin/login/?ref=/'>Sweepin Manager</a>. It let you create your campaigns online in no time and broadcast to your audience.
Watch the analytics data on our charts, and get the visitor's traffic in real time.

## Add SweepinConnect to your project

Gradle:

`compile 'com.github.sweepin:sweepinconnect:2.0.1'`

Maven: 

```
<dependency>
   <groupId>com.github.sweepin</groupId>
   <artifactId>sweepinconnect</artifactId>
   <version>2.0.1</version>
   <type>pom</type>
</dependency>
```

## Installation

### Step 1 : 

Init the SDK in the onCreate() of your application class.

`ProximitiesConfig.getInstance().initSweepinConnectSdk(this);`

Note : Don't forget to add android:name=".MyApp" in the application element of your manifest.

### Step 2 :

Start the service to enable the reception of campaigns.

```
public class MyActivity extends AppCompatActivity implements OnSweepinConnectServiceReady, OnAccessLocationListener {

	private ProximitiesConfig mPrxConfig;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		mPrxConfig = ProximitiesConfig.getInstance();
		mPrxConfig.startSweepinConnectService(this);
	}

	@Override
	public void onSweepinConnectServiceReady() {
		mPrxConfig.askLocationPermission(this);
	}

	@Override
	public void onAccessLocationGranted() {
		mPrxConfig.startLocationManager();
		mPrxConfig.startBeaconManager();
	}

	@Override
	public void onAccessLocationDenied() {
		
	}

}
```
Once the service is ready, you need to ask the location permission for devices above Api level 23. You can use your own implementation if you want to.
Once the permission is granted, you can start the location manager and/or the beacon manager depending on your needs. 

### Step 3 : 

You need an app id and an app secret to make authorized requests to our back office. Declare the following meta-data in your manifest:

```xml
<meta-data
    android:name="proximities:appId"
    android:value="YOUR_APP_ID" />

<meta-data
    android:name="proximities:appSecret"
    android:value="YOUR_APP_SECRET"/>
```

#### To know more about what you can do with Sweepin Connect, go to the <a href='https://github.com/Sweepin/SweepinConnect-android/blob/master/Advanced_Configurations.md'>Advanced configuration</a> documentation.

### Congratulations, your app is now ready to go ! 
##### Go to <a href='http://manager.sweepin.fr/admin/login/?ref=/'>The Sweepin Manager interface</a> to create your first campaign !

## License
___
SweepinConnect is available under the MIT license. See the LICENSE file for more info.

  [1]: http://www.sweepin.fr/contact

	 

 




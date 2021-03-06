<p align="center" >
  <img src="Images/logoSweepinConnect-850x300.png" alt="SweepinConnectLogo" title="SweepinConnectLogo">
</p>

# SweepinConnect-android : Advanced configuration
## Table of contents

- [Introduction](#introduction)
- [Support for Android 8+](#support_android8)
- [Beacon scanner](#beacon)
- [Use NFC and QR codes](#nfc_qrcodes)
- [Location changes](#locationChanges)
- [SQLite Database](#sqlite_database)
- [Get partners and their content](#getPartners)
- [Get campaigns by user action](#getCampaignsByUserAction)
- [Disable campaigns](#disable_campaigns)
- [Behavior after the reception of a campaign](#behaviorAfterCampaignReception)
- [Simple notifications (deep links..)](#simpleNotification)
- [Register users in our system](#register_users)
- [Favorites](#favorites)
- [Set up your own campaign menu](#campaignMenu)
- [Handle URL in campaigns](#handleURL)
- [Share fragment](#share)
- [Stop Sweepin Connect Service](#stop_service)
- [About some customization](#customization)

<div id='introduction'/>

# Introduction

Once you integrated SweepinConnect-android in your project, you'll have the possibility to configure important features and interact with crucial elements of the library. 
All the possibilities are listed in this document.

Note: It is important to note that most methods and listeners need to be implemented in your application class since the reception of campaigns is available when the app is killed.

<div id="support_android8"/>

# Support for Android 8+

This section only concerns the SweepinConnect SDK installed on devices with Android 8 or superior.

In order to respect background limitations introduced in Android 8, the SDK had to adapt to provide a solution for receiving campaigns when the app is killed. The background detection for beacons can no longer be as responsive as it was in previous versions of the SweepinConnect SDK without drawbacks. The only way to be able to detect beacons as fast is to use a foreground service. 

By default, the SDK enables a foreground service each time a beacon with an active campaign is nearby. You can enable or disable this behaviour with the following methods.

```groovy
prxconfig.enableForegroundNotificationForBeacons();

prxconfig.disableForegroundNotificationForBeacons();
```
If you decide to disable the foreground notification, campaigns on beacons will only be receivable with your app in the foreground or on background for a short period of time.

This foreground notification can be customized entirely.

#### strings.xml
`prxsc_foreground_notification_title` : title of the notification<br/>
`prxsc_foreground_notification_content` : content of the notification<br/>

#### drawable.xml
`prxsc_ic_small_foreground_notification` : replace the small icon<br/>
The large icon uses the `prxsc_ic_notification`.<br/>

<div id="beacon"/>

# Beacon scanner

By default, the beacon scanner is configured to only care for specific uuids. However, it is possible to add a list of uuids to the scanner and configure campaigns on your own beacons.

```groovy
ProximitiesConfig.addUuidsToBeaconScanner(List<String> listOfUuids);
```

You can modify the duration and the frequency of each beacon scan. The default values set below are recommended by the Altbeacon library.

```groovy
ProximitiesConfig.getInstance().setBackgroundScanPeriod(5000, 25000);
ProximitiesConfig.getInstance().setForegroundScanPeriod(1100, 0);
```
Note: When a beacon is scanned for the first time, an api call is sent to determine if a campaign is linked to this beacon.
If no campaign is present, api calls are then spaced by 30 seconds to avoid the abuse of server requests.

You may need to retrieve all scanned beacons simply to identify them or to perform a particular action. If it is the case, you can implement the OnScannedBeaconsListener.

```groovy
@Override
public void onScannedBeacons(Collection<Beacon> beacons) {
	for (Beacon beacon : beacons) {
	    String uuid = beacon.getId1().toString();
	    String major = beacon.getId2().toString();
	    String minor = beacon.getId3().toString();
	    Log.d("BeaconScanned", "Uuid : " + uuid + " Major : " + major + " Minor : " + minor);
	}
}
```

A new way of detecting beacon has been implemented in the v2 of the SweepinConnect SDK called 'Closest Beacon Mode'.
This mode still detects all beacons in its range but only consider the closest one and ignore the rest. It is particularly useful if you plan to have a route of beacons and the order for receiving the information is important.

```groovy
ProximitiesConfig.getInstance().enableClosestBeaconMode(boolean enable);
```
     
Note : The beacon configuration is extremely important for the effectiveness of this method. Configure your beacons with the same transmission power and broadcasting interval.

<div id='nfc_qrcodes'/>

# Use NFC and QR codes

The SweepinConnect library gives you the possibility to use NFC tags and QR codes as transmitters. In order to make those technologies work, you need to add the following intent-filters in one of your activity in the AndroidManifest.

```groovy
<!--NFC-->
<intent-filter>
	<action android:name="android.nfc.action.NDEF_DISCOVERED" />
	<category android:name="android.intent.category.DEFAULT" />
	<data
            android:host="sweepin.fr"
            android:scheme="https"
            android:pathPattern="@string/prxsc_intent_filter_path_pattern" />
</intent-filter>

<!--Qr codes-->
<intent-filter>
<action android:name="android.intent.action.VIEW"/>
<category android:name="android.intent.category.DEFAULT"/>
<category android:name="android.intent.category.BROWSABLE"/>
	<data
            android:host="sweepin.fr"
            android:scheme="https"
            android:pathPattern="@string/prxsc_intent_filter_path_pattern" />
</intent-filter>
```

To properly handle NFC intents, you also need to declare this permission : 

```groovy
<uses-permission android:name="android.permission.NFC" />
```

Once your intent-filters are in place, you will see that path patterns are not defined yet. 
Add the following string to your xml file, do not forget to add your appId at the end:

```groovy
<string name="prxsc_intent_filter_path_pattern" translatable="false">"/r/transmitters/.*\\/.*\\/.*\\/<your-appId>"</string>
```

The only missing part is how to handle the intent generated by a scan of a qr code or a NFC tag. 
See below how your activity should look. 

```groovy
private ProximitiesConfig mPrxConfig;

@Override
protected void onCreate(Bundle savedInstanceState) {
	..
	mPrxConfig = ProximitiesConfig.getInstance();
	handleNfcAndQrCode(getIntent());
	..
}

@Override
protected void onNewIntent(Intent intent) {
	super.onNewIntent(intent);
	handleNfcAndQrCode(intent);
}

private void handleNfcAndQrCode(Intent intent){
        mPrxConfig.readQrCode(intent.getDataString());
        mPrxConfig.readNfc(intent);
}
```

Once is done, every NFC tag and/or Qr code respecting your intent-filter configuration will open your app.

One last thing concerning Qr Codes, if you desire to have your own Qr code reader in your app, just use the one available in SweepinConnect :

```groovy
ProximitiesConfig.getInstance().startQrCodeReader();
```

Then you have access to the OnResponseFromQrScanListener() to handle different cases after the scan :

```groovy
@Override
public void onUnknownQrCodeResponse() {
   Toast.makeText(getApplicationContext(), "This Qr Code is unknown.", Toast.LENGTH_SHORT).show();
}

@Override
public void onNoCampaignOnQrCodeResponse() {
   Toast.makeText(getApplicationContext(), "There is currently no new campaign on this Qr code.", Toast.LENGTH_SHORT).show();
}

@Override
public void onErrorOnQrCodeResponse() {
   Toast.makeText(getApplicationContext(), "Error Qr Code", Toast.LENGTH_SHORT).show();
}
```


<div id='locationChanges'/>

# Location changes

You have the possibility of requesting a location update that will automatically set the system at the new location. It is recommended to not abuse the use of this function since location updates will have an impact on the battery life.

```groovy
ProximitiesConfig.requestLocationUpdate();
```

You can implement the OnLocationChangeListener that will be called when a location request is done.

In case you need to know the last known location of the customer, use this method :

```groovy
LatLng lastLocation = ProximitiesConfig.getInstance().getLastKnownLocation(this);
double latitude = lastLocation.latitude;
double longitude = lastLocation.longitude;
```

By default, location requests uses the PRIORITY_BALANCES_POWER_ACCURACY to limit the battery comsumption but the choice is yours :

```groovy
ProximitiesConfig.getInstance().setLocationPriority(LocationRequest.PRIORITY_BALANCED_POWER_ACCURACY);
```

<div id='sqlite_database'/>

# SQLite Database

A Sqlite database is created by default to store campaign's logs when the network is unavailable and to store campaigns when a user is near a point of interest where 'Background Fetch'* is activated.
If, for example, the network availability is not an issue, you can choose to disable the creation of this database.

```groovy
ProximitiesConfig.getInstance().enableSQLiteDatabase(boolean enable);
```

* Background Fetch : This option concerns points of interest where the network is not reliable. To activate this option, you need to contact us directly.

<div id='getPartners'/>

# Get partners and their content 

You have the possibility of retrieving the list of partners with all their content (points of interest, transmitters, campaigns..).
The request needs 2 parameters : the number of elements you want to retrieve (the x closest partners) and the type of campaign you looking for.

To get all partners, set the 'nbElements' parameter to 0.

For the 'type' parameter, use the following constants :
```groovy
ProximitiesConfig.TYPE_ALL // use this to get all types of campaigns
ProximitiesConfig.TYPE_INFORMATION
ProximitiesConfig.TYPE_PROMOTION
ProximitiesConfig.TYPE_CULTURE
ProximitiesConfig.TYPE_EVENT
```

```groovy
ProximitiesConfig.getInstance().getPartners(int nbElements, String type, OnRetrievePartnersListener listener);
```

You can now handle the response with OnRetrievePartnersListener. You can see below how to retrieve the content of each partner.

```groovy
@Override
public void onRetrievePartners(List<Partner> partners) {
if(partners != null && !partners.isEmpty()){
    for(Partner partner : partners){
        List<Poi> pois = partner.getPois();
        if(pois != null && !pois.isEmpty()){
            for(Poi poi : pois){
                List<Campaign> campaigns = poi.getCampaigns();
                if(campaigns != null && !campaigns.isEmpty()){
                    for(Campaign campaign : campaigns){
                        //campaigns of the current poi in the current partner (on geofencing)
                    }
                }
                List<Transmitter> transmitters = poi.getTransmitters();
                if(transmitters != null && !transmitters.isEmpty()){
                    for(Transmitter transmitter : transmitters){
                        //transmitters of the current poi in the current partner
                        List<Campaign> campaignsOnTransmitter = transmitter.getCampaigns();
                        if(campaignsOnTransmitter != null && !campaignsOnTransmitter.isEmpty()){
                            for(Campaign campaign : campaignsOnTransmitter){
                                // campaigns on the current transmitter of the current poi in the current partner
                            }
                        }
                    }
                }
            }
        }
    }
}
}
```

<div id='getCampaignsByUserAction'/>

# Get campaigns by user action

You have a listener at your disposal to request specific campaigns based on user actions:


`ProximitiesConfig.USER_ACTION_ALL_CAMPAIGN_RECEIVED` : returns all campaigns received by the user
`ProximitiesConfig.USER_ACTION_CAMPAIGN_RECEIVED_IN_FOREGROUND` : returns all campaigns received in foreground by the user
`ProximitiesConfig.USER_ACTION_CAMPAIGN_RECEIVED_IN_BACKGROUND` : returns all campaigns received in background by the user
`ProximitiesConfig.USER_ACTION_CAMPAIGN_SAVED` : returns all campaigns added in favourites by the user

```groovy
ProximitiesConfig.getCampaignsByUserAction(getApplicationContext(), ProximitiesConfig.USER_ACTION_CAMPAIGN_RECEIVED, 
	new OnGetCampaignsByUserActionListener() {
            @Override
            public void onGetCampaignsByUserAction(List<Campaign> campaigns) {
              // TO DO
            }

            @Override
            public void onGetCampaignsByUserActionError() {
              // TO DO
            }
});
```

You can then display a campaign using : 

```groovy
 ProximitiesConfig.getInstance().openCampaign(Campaign campaign);
```

<div id='disable_campaigns'/>

# Disable campaigns

It is possible to prevent the display of campaigns at anytime and re-enable it later using the following method.

```groovy
ProximitiesConfig.getInstance().disableAllCampaigns(boolean disable);
```

<div id='behaviorAfterCampaignReception'/>

# Behavior after the reception of a campaign

If a campaign is received when your app is not running, use the method below to configure an activity that will be brought to front as soon as the campaign is closed.

```groovy
ProximitiesConfig.setMainActivity(MainActivity.class);
```

In case the application was opened or in background at the reception, users will retrieve the last activity they were in.

A listener is available to catch the reception and opening of campaigns :

```groovy
ProximitiesConfig.setOnOpenCampaignListener(new OnOpenCampaignListener() {
            @Override
            public void onOpenCampaign(Activity activity, Campaign campaign) {
                // TO DO
            }
});
```

You may need a callback everytime the user closes a campaign activity (multi or not) in order to refresh a list for example. You can use the following one :

```groovy
ProximitiesConfig.setOnCloseCampaignListener(new OnCloseCampaignListener() {
            @Override
            public void onCloseCampaign(boolean isReceived, List<Campaign> campaigns) {
               // TO DO
            }
        });
```

<div id='simpleNotification'/>

# Simple notification (deep links, ad servers, etc.)

When you create a simple notification on the manager, it is possible to add a unique identifier that can be use later by the developer to run a certain piece of code (and use deep links for example). 

By default, a simple notification displays a dialog (at the condition the app was open at the reception) and leaves a choice to close or accept the information. A method allows you to skip this behavior and automaticaly activate the event of the specific identifier.

```groovy
ProximitiesConfig.getInstance().enableDialogOnSimpleNotification(boolean isEnabled);
```

You also have a listener at your disposal to catch the identifier in 3 cases :
- The user received a notification and opened it (imply the app was in background),
- If the dialog is enable, the user accepted the information (imply the app was in foreground),
- If the dialog is disable, the user received the simple notification with this app in foreground.

To use this listener, you need to implements OnCatchIdentifierListener in your application class.

```groovy
@Override
    public void onCatchIdentifier(String identifier) {
        if (identifier.equals("test-simple")) {
            Toast.makeText(SweepinConnectApp.this, "test-simple", Toast.LENGTH_SHORT).show();
        }
    }
```

<div id='register_users'/>

# Register users in our system

At one point, your app's customer is going to login (or auto-login) to your app. If you want the SweepinConnect system to be aware of your users information, you'll need to register this user to the system. For example, you can call the registerUser() method on your login/autologin routine method. 
Registering users is the way to send segmented campaigns on a specific group of users.

```groovy
Map<String, String> parameters = new HashMap<>();
parameters.put("civility", civility);
parameters.put("name", name);
parameters.put("company", company);
parameters.put("email", email);
// and more...
ProximitiesConfig.registerUser(identifier, parameters);
```

Note : You need to choose one of you parameter as identifier, it is highly recommended to use the customer's email as such.

<div id='favorites'/>

# Favorites

The SDK contains a default fragment named 'FavoritesFragment' that you can call where you desire to display the favorites in your application. 

You can also create your own using the 'BaseFavoritesFragment' by inheritance and override its methods as follow :

```groovy
public class MyFavoritesFragment extends BaseFavoritesFragment{

	@Override
    	public void onGetFavorites(List<Campaign> favorites) {
		super.onGetFavorites(favorites);
	}

	@Override
    	public void onGetFavoritesError() {
        	super.onGetFavoritesError();
    	}	
}
```

Finally, you can choose to disable the favorite menu item in your campaigns if you don't want to add the favorites system in your app.

```groovy
ProximitiesConfig.getInstance().disableFavorites(boolean disableFav);
```

<div id='campaignMenu'/>

# Set up your own campaign menu

By default, a campaign's toolbar contains only one menu item to allow users to save the campaign in their favorites or to remove it.
Those icons can be modify in the case you want to display your own creations by adding this line :

```groovy
ProximitiesConfig.getInstance().setFavoritesIcons(int favoritesAddedResource, int favoritesDeletedResource, boolean useColorFilter);
```

The third parameter 'useColorFilter' uses the font color of the campaignr's title as color filter for your icons.

You may want to create your own menu if you choose not to use the favorite's item. You can then set up your own icons and their actions using OnCustomMenuCampaignListener() :

```groovy
@Override
public void customizeMenu(Menu menu) {
	menu.add(0, 1, Menu.NONE, R.string.example).setIcon(R.drawable.ic_launcher).setOnMenuItemClickListener(new MenuItem.OnMenuItemClickListener() {
	    @Override
	    public boolean onMenuItemClick(MenuItem item) {
		Toast.makeText(getApplicationContext(), "Test", Toast.LENGTH_SHORT).show();
		return true;
	    }
	}).setShowAsAction(MenuItem.SHOW_AS_ACTION_ALWAYS);
}
```

Note : Don't forget you need to add those lines in the OnCreate() of your application class.

<div id='handleURL'/>

# Handle URL inside campaigns

Most of the campaigns available in the back-office can contain URLs in the text section. By default, a click on an URL will open an activity with a full size webview to host the webpage but this behavior can be modified using OnCampaignURLClickListener.

```groovy
@Override
public boolean onCampaignURLCLick(String url) {
	if (condition)) {
		Toast.makeText(getApplicationContext(), "Condition OK", Toast.LENGTH_SHORT).show();
		return true;
	} else {
		return false;
	}
}
```

In case you want to enable the default view anyway inside the listener, just return false.

Keep in mind that you need to put this in the onCreate() of your application class.

<div id='share'/>

# Share campaigns

It is possible to add a fragment at the bottom of the received configurable campaign to give the possibility of sharing the content. You can create yours using the ShareFragment by inheritance. 
Then, you can retrieve a SharedCampaign object containing all the information you need concerning the campaign with the function getSharedCampaign(). To avoid issues, retrieve the object once your view is created.

```groovy
public class ShareCampaignFragment extends ShareFragment {

    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        SharedCampaign campaign = getSharedCampaign();
    }
}
```

Once your fragment is created, you need to send it to the SweepinConnect library in the OnCreate() of your application class.

```groovy
ProximitiesConfig.getInstance().setShareFragment(MyShareFragment.class);
```

<div id='stop_service'/>

# Stop Sweepin Connect Service

Once the Sweepin Connect service is started, it will run in background on your device. You can choose to stop the service anytime you want using :

```groovy
ProximitiesConfig.getInstance().stopSweepinConnectService();
```

<div id='customization'/>

# About some customization

To customize, you just need to name resources in your app as follow :

### strings.xml
`prxsc_multi_campaigns_title` : title of the activity listing all campaigns just received (and of the notification if in background)<br/>
`prxsc_multi_campaigns_notification_content` : content of the notification

### drawable.xml
`prxsc_ic_notification` : icon of the notification for multiple campaigns<br/>
`prxsc_ic_small_notification` : icon for the status bar<br/>
`prxsc_ic_favorite_empty` : action bar icon when the campaign is not in the user's favorites<br/>
`prxsc_ic_favorite_added` :action bar icon when the campaign is in the user's favorites

### colors.xml
`prxsc_notification_light` : color of the led when you received a notification<br/>
`prxsc_multi_campaign_title` : your title's font color (for multiple campaigns activity)<br/>
`prxsc_multi_campaign_toolbar` : color of the toolbar (for multiple campaigns activity)<br/>
`prxsc_simple_push_title` : your title's font color (for Simple notification template)<br/>
`prxsc_simple_push_dialog` : color of the dialog (for Simple notification template)


Note : Use 'setTintOnFavoritesIcons(boolean useTint)' or not depending on your icons (uses the font color).








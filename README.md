![](./readmeAssets/wordmark.png)
##### playPORTAL <sup>TM</sup> provides a service to app developers for managing users of all ages and the data associated with the app and the app users, while providing compliance with required COPPA laws and guidelines.

# <b>playPORTAL SDK for iOS apps</b></br>

## Getting Started

* ### <b>Step 1:</b> Create playPORTAL Partner Account

	* Navigate to [playPORTAL Partner Dashboard](https://partner.playportal.io)
	* Click on <b>Sign Up For Developer Account</b>
	* After creating your account, email us at [info@playportal.io](mailto:info@playportal.io?subject=Developer%20Sandbox%20Access%20Request) to verify your account.
  </br>

* ### <b>Step 2:</b> Register your App with playPORTAL

	* After confirmation, log in to the [playPORTAL Partner Dashboard](https://partner.playportal.io)
	* In the left navigation bar click on the <b>Apps</b> tab.
	* In the <b>Apps</b> panel, click on the "+ Add App" button.
	* Add an icon, name & description for your app.
	* For "Environment" leave "Sandbox" selected.
	* Click "Add App"
  </br>

* ### <b>Step 3:</b> Generate your Client ID and Client Secret

	* Tap "Client IDs & Secrets"
	* Tap "Generate Client ID"
	* The values generated will be used in 'Step 5'.
  </br>

* ### <b>Step 4:</b> Add a "Registered Redirect URI"

	* Tap "Registered Redirect URIs"
	* Tap "+ Add Redirect URI"
	* Enter "helloworld://redirect" in to the prompt and click "Submit".
  </br>

* ### <b>Step 5:</b> Add Client ID and Client Secret to App

	* In "AppDelegate.m" replace the following values with the values generated in 'Step 3'.
		```
		NSString *cid = @"YOUR_CLIENT_ID_HERE";
		NSString *cse = @"YOUR_CLIENT_SECRET_HERE";
		```
		###### For the purpose of running this HelloWorld app, these keys are in plain text in the file, but for a production app you must store them securely - they uniquely identify your app and grant the permissions to your app as defined in the playPORTAL Partner Dashboard.

* ### <b>Step 6:</b> Add URL Query Scheme to Info.plist

	* Right click on the 'Info.plist' file in the navigation window of XCode.
	* Select "Open as > Source Code"
	* Be sure the following lines are included, if not add them:
		```
		<key>CFBundleURLTypes</key>
		<array>
		<dict>
			<key>CFBundleTypeRole</key>
			<string>Editor</string>
			<key>CFBundleURLName</key>
			<string>helloworld</string>
			<key>CFBundleURLSchemes</key>
			<array>
				<string>helloworld</string>
			</array>
		</dict>
		</array>
		```
		###### For the purpose of running this HelloWorld app, these keys are in plain text in the file, but for a production app you must store them securely - they uniquely identify your app and grant the permissions to your app as defined in the playPORTAL Partner Dashboard.

* ### <b>Step 7:</b> Run the app from XCode.

* ### <b>Step 8:</b> Generate "Sandbox" users for testing.
	* In the [playPORTAL Partner Dashboard](https://partner.playportal.io), click on "Sandbox" in the left navigation pane.
	* Here you can generate different types of "Sandbox Users" so you can log in to your app and try it out.
	* "Sandbox Users" can be of type "Adult", "Parent", or "Child".
	* You can also create friendships between the users using the dropdowns in each "profile preview".
	
* ### <b>Step 9:</b> XCode Setup
    * Select Targets/AppName --> Build Phases
    * Add the following frameworks by tapping --> Link Binary with Libraries, the "+" for each of:
       - Security.framework
       - Webkit.framework
       - MobileCoreServices.framework
       - SafariServices.framework
       - playPortal.framework

    * Select an attached target device (or simulator)
		- Tap "Play button" (run)
		- App should build and run on the attached iOS device
     	- Note: If Xcode generates an error regarding Objective-c exceptions; fix error in Xcode by selecting "Project" and search for "exception". Set "Enable Objective-C Exceptions" to "Yes".
		- Tap "Play" button again

-----

### Utilizing SDK functionality
After the Xcode environment configuration is complete, it's time to configure your application to utilize the playPORTAL SDK for various operations.

#### Configuring the SDK runtime
The SDK must be configured in conjunction with the playPORTAL, so that your app can consume playPORTAL services. The following must be done (statements in grey blocks are code snippets / examples):

* Edit your AppDelegate.m

	* Include the PlayPortal namespace by adding the following statement at the top of your app.
	```
	 #import <PlayPortal/PPManager.h>
	```

	* Update your myClientID and mySecret string vars with the information from your app definition/configuration in playportal.io.
		```
		NSString myClientID = @"<YOUR_CLIENT_ID_HERE>";
		NSString mySecret = @"<YOUR_CLIENT_SECRET_HERE";
		NSString myRedirectURI = @"appname/redirect";  
		```

	* All playPORTAL SDK services are accessed thru a single or static class method. Initialize the SDK:
		```
	    [PPManager setPpsdkEnvironment:env];  // where env is one of { "SANDBOX", "PRODUCTION" }
	    [[PPManager sharedInstance] configure:cid secret:cse andRedirectURI:redirectURI];
		```

	* Register a method to receive callbacks upon completion of SSO authentication
	```
	- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url {
	    [[PPManager sharedInstance] handleOpenURL:url];
		return true;
	}

	```


Your app is now ready to begin making calls into the PPSDK plugin.


#### Making calls into the PPSDK plugin from within your application:

##### Setup User listener
The user listener is invoked by the SDK on changes in the auth status of the app user. This example shows the auth listener being instantiated in the initially presented viewController specified by the Storyboard.

    [PPManager sharedInstance].PPusersvc.addUserListener = ^(PPUserObject *user, NSError *error){
		if (error) {
			NSLog(@"%@ error: %@", NSStringFromSelector(_cmd), error);
		} else {
   		        UIStoryboard *sb = [UIStoryboard storyboardWithName:@"Main" bundle:nil];
			if(auth) {
			    NSLog(@"username=%@", user.handle);
			    UserViewController *vc = [sb instantiateViewControllerWithIdentifier:@"userViewController"];
		            vc.user = user;
			    vc.modalTransitionStyle = UIModalTransitionStyleFlipHorizontal;
           		    [self presentViewController:vc animated:YES completion:NULL];

    			} else {
			    // auth has failed, present SSO again
			    PPLoginButton *loginButton = [[PPLoginButton alloc] init];
			    loginButton.center = self.view.center;
			    [self.view addSubview:loginButton];
			}

		}			
	};


##### SSO Login
The SSO login validates a single player (user) against the playPORTAL. Players may log in with a valid playPORTAL set of credentials, or as a guest player.
This method will initiate the login process. If a player is already logged in, it will reconnect that Player to their playPORTAL account

    // Instantiate a playPORTAL login button, PPLoginButton. This will cause the SDK to handle the auth flow (on press)
    PPLoginButton *loginButton = [[PPLoginButton alloc] init];
    loginButton.center = self.view.center;
    [self.view addSubview:loginButton];


--
##### Data
The SDK provides a simple Key Value (KV) read/write model. On login, there are two data stores opened / created for this player. There is a private data store for this players exclusive use, and there is a global data store this player shares with all other players of this same app. If a player logs out and logs in at a later date, the data in the private data store should be as left upon logout. The contents of the global data store will most likely have changed.


	    void writeMyData(string key, string value);

   			string key - a key to associate with this data
    		string value - value to store

    	This method will write a KV pair to this user's private data store. If a key is used more than once, the value associated with the key will be updated.



	    void writeGlobalData(string key, string value);

   			string key - a key to associate with this data
    		string value - value to store

    	This method will write a KV pair to this application's global data store. Again, if a key is used more than once (by any user), the value associated with the key will be updated.

--

         public void readMyData(string key, Action<string>callback);

			string key - a key to read from.
			callback - C# method that takes a string parameter containing the returned value

			The callback method is defined as:
						
			private delegate void ReadDataDelegate(string value);
			[AOT.MonoPInvokeCallback(typeof(ReadDataDelegate))]  
			protected static void ReadCallback(string value)
			{
				// do something with the value
			}

	public void readMyDataAsDictionary(string key, Action<string>callback);
		// This returns the KV pair in a JSON string format. The JSON string can be converted to a C# object with:
	        someObject = (SomeObject)JsonUtility.FromJson(theJsonString, typeof(SomeObject)); 
		// where SomeObject is defined as appropriate for the data being stored (a simple example is included).


	public void readGlobalData(string key, Action<string>callback);

			string key - a key to read from.
			callback - C# method that takes a string parameter containing the returned value (see previous example for method of defining a callback fnx.)


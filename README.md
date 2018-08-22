![](./readmeAssets/studio.png)
# <b>playPORTAL SDK for iOS</b></br>
##### playPORTAL Studio<sup>TM</sup> provides a service to app developers for managing users of all ages and the data associated with the app and the app users, while providing compliance with required COPPA laws and guidelines.

## Getting Started

* ### <b>Step 1:</b> Create playPORTAL Studio Account

	* Navigate to [playPORTAL Studio](https://studio.playportal.io)
	* Click on <b>Sign Up For FREE Account</b>
	* After creating your account, email us at [info@playportal.io](mailto:info@playportal.io?subject=Developer%20Sandbox%20Access%20Request) to verify your account.
  </br>

* ### <b>Step 2:</b> Register your App with playPORTAL

	* After confirmation, log in to the [playPORTAL Studio](https://studio.playportal.io)
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
	* Enter your app's redirect uri (e.g. - helloworld://redirect) in to the prompt and click "Submit".
  </br>

* ### <b>Step 5:</b> Add Client ID and Client Secret to App

	* In "AppDelegate.m" replace the following values with the values generated in 'Step 3'.
		```
		NSString *cid = @"YOUR_CLIENT_ID_HERE";
		NSString *cse = @"YOUR_CLIENT_SECRET_HERE";
		```
		###### For the purpose of this guide, these keys are in plain text in the file, but for a production app you must store them securely - they uniquely identify your app and grant the permissions to your app as defined in playPORTAL Studio.

* ### <b>Step 6:</b> Add URL Query Scheme to Info.plist

	* Right click on the 'Info.plist' file in the navigation window of XCode.
	* Select "Open as > Source Code"
	* Add the following lines:
		```
		<key>CFBundleURLTypes</key>
		<array>
		<dict>
			<key>CFBundleTypeRole</key>
			<string>Editor</string>
			<key>CFBundleURLName</key>
			<string>
				YOUR_APP_BUNDLE_URL
			</string>
			<key>CFBundleURLSchemes</key>
			<array>
				<string>
					YOUR_APP_REDIRECT
				</string>
			</array>
		</dict>
		</array>
		```

* ### <b>Step 7:</b> Run the app from XCode.

* ### <b>Step 8:</b> Generate "Sandbox" users for testing.
	* In [playPORTAL Studio](https://studio.playportal.io), click on "Sandbox" in the left navigation pane.
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

	* Include the PlayPortal definitions by adding the following statement at the top of your app.
	```
	 #import <PlayPortal/PPManager.h>
	```

	* Update your myClientID and mySecret string vars with the information from your app definition/configuration in studio.playportal.io.
		```
		NSString myClientID = @"<YOUR_CLIENT_ID_HERE>";
		NSString mySecret = @"<YOUR_CLIENT_SECRET_HERE>";
		NSString myRedirectURI = @"yourappname/redirect";  
		```

	* All playPORTAL SDK services are accessed either thru a singleton or a static class method.
		```
	    [PPManager setPpsdkEnvironment:env];  // where env is one of { "SANDBOX", "PRODUCTION" }
	    [[PPManager sharedInstance] configure:cid secret:cse andRedirectURI:redirectURI];
		```

	* Register a method to receive the SSO callback from the playPORTAL server:
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
The SSO login authenticates a single user with the playPORTAL service. Users may log in with a valid set of playPORTAL credentials, or as a guest.
This method will initiate the login process. If a user is already logged in, it will reconnect that user to their playPORTAL account

    // Instantiate a playPORTAL login button, PPLoginButton. This will cause the SDK to handle the auth flow (on press)
    PPLoginButton *loginButton = [[PPLoginButton alloc] init];
    loginButton.center = self.view.center;
    [self.view addSubview:loginButton];

On successful SSO authentiation, the user's profile is returned via the userListener() method. The user profile (PPUserObject) contains the following fields:

	NSString* userId;
	NSString* handle;
	NSString* firstName;
	NSString* lastName;
	NSString* country;
	NSString* accountType;
	NSString* userType;
	NSDictionary* parentFlags;
	NSString* profilePicId;
	UIImage* profilePic;
	NSString* coverPhotoId;
	UIImage* coverPhoto;

--

##### Friends
The SDK provides a method to retrieve a user's friends and their profiles (which contain the same info as a PPUserObject):

	PPManager *ppm = [PPManager sharedInstance];


	[ppm.PPusersvc getFriendsProfiles:^(NSError *error) { 
    		if(error == NULL) {
			int fc = [ppm.PPfriendsobj getFriendsCount];
		}
	}];
    
    
	Individual friends profiles can be viewed by the following method:
	
	NSDictionary* d = [ppm.PPfriendsobj getFriendAtIndex:index];  

		where d = NSDictionary containing a friend record
		      index = int (< friends count) of the friend's record to view;


##### Data
The SDK provides a simple JSON Key Value (KV) read/write capability. On login, there are two data stores opened / created for this user. There is a private data store for this user's exclusive use, and there is a global data store this user shares with all other users of this same app. If a user logs out and logs in at a later date, the data in the private data store should be as left upon logout. The contents of the global data store will most likely have changed depending on other app users actions.
	
	PPManager *ppm = [PPManager sharedInstance];
	
	[ppm.datasvc writeBucket:  andKey:(NSString*)key andValue:[NSString stringWithFormat:@"%ld, value] push:FALSE handler:^(NSError *error) { }

		string bucket - bucket name, where (myAppGlobalDataStorage and myDataStorage are predefined)
		string key - a key to associate with this data
    		string value - value to store

    	This method will write a KV pair to this user's private data store. If a key is used more than once, the value associated with the key will be updated.

---

	    [ppm.PPdatasvc readBucket:bucket andKey:(NSString*)key handler:^(NSDictionary* d, NSError* error) {}
		string bucket - bucket name, where (myAppGlobalDataStorage and myDataStorage are predefined)
		string key - a key to associate with this data
		NSDictionary *d - an NSDictionary is returned



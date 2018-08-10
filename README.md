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

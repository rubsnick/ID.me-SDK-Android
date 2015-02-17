# ID.me SDK (Android)

## General

### Release Information

* **SDK Version:** 1.0.0 (February 16, 2015)
* **Maintained By:** [Ruben Roman](https://github.com/rubsnick).

For more information please email us at mobile@id.me or visit us at [http://developer.id.me.](http://developer.id.me.)


### General Information
The **ID.me SDK** for Android is a Java library that adds the following functionality to your Android application:

* Single sign-on (incl. sign-up) using a users **ID.me** account(see **Section C.**)
* Verification of a user's military affinity stat (see **Section D.**)
* Scan and parse all information found on a user's physical credential (see **Section E.**)


### Device Compatibility
* Android 4.0+

### Future Updates
* Student, teacher, and first-responder affinity verification coming soon.
* Have a suggestions, contacts us at mobile@id.me.


____
## Table of Contents
- [A. Installation & Setup](https://github.com/IDme/ID.me-SDK-Android#a-installation--setup)
- [B. Activation](https://github.com/IDme/ID.me-SDK-Android#c-activiation)
- [C. Authenticating a User](https://github.com/IDme/ID.me-SDK-Android#d-authenticating-a-user)
- [D. Verifying a User's Attributes](https://github.com/IDme/ID.me-SDK-Android#e-verifying-a-users-attribute)
- [E. Credential Scanning and Parsing](https://github.com/IDme/ID.me-SDK-Android#f-credential-scanning-and-parsing)
- [F. Error Handling](https://github.com/IDme/ID.me-SDK-Android#g-error-handling)
- [G. Terms of Service](https://github.com/IDme/ID.me-SDK-Android#h-terms-of-service)

___

## A. Installation & Setup

**Manual:**

1. Drag the **ID.me SDK** file into the libs folder of your project.
2. Add the following dependency to your `build.gradle` file:   

```Gradle
repositories {

flatDir {
   dirs 'libs'
     }
}

compile 'com.android.support:appcompat-v7:21.0.3'
compile 'com.squareup.retrofit:retrofit:1.9.0'
compile 'com.rengwuxian.materialedittext:library:1.8.1'
compile 'com.squareup.okhttp:okhttp:2.2.0'
compile 'com.squareup.okhttp:okhttp-urlconnection:2.2.0'

compile (name:'IDmeSDK',ext:'aar')
```

**Gradle:**

Add these lines to build.gradle

```Gradle
repositories {
    maven { url 'https://oss.sonatype.org/content/groups/public' }
}

dependencies {
    compile 'me.id.sdk:SDK:1.0'
}
```

The SDK also comes bundled with the following third party libraries:

* MWBarcode Scanner ([http://www.manateeworks.com]( http://www.manateeworks.com))

## B. Activation

The SDK must be activatefd using a Serial Key, which can be obtained by emailing mobile@id.me:

To activate the SDK, simply run the following command with the provided serial-key:

```Java
IDmeSDK.getInstance().startSDK(ClientID, ClientSecret);
```

where the two parameters are described below:

* `ClientID`: The ClientID that is provided by ID.me upon registering an app at http://developer.id.me
* `ClientSecret`: The ClientSecret that is provided by ID.me upon registering an app at http://developer.id.me

To scan physical credentials, you must call the following method:

* `IDmeSDK.getInstance().setScannerKey(ScannerKey);`
* `IDmeSDK.getInstance().setScannerUser(ScannerUsername);`

where the two parameters are described below:

* `ScannerKey`: The scanner key is needed to use the barcode scanner. This Key is provided by [Manatee Works.](https://manateeworks.com/)
* `ScannerUsername`: The scanner username is needed to use the barcode scanner. This username is provided by [Manatee Works.](https://manateeworks.com/)

As the `IDmeSDK` class is a singleton, activation can occur at any point in time. It is our recomendation that activation occur in the Launcher Activity. Here is some sample code on what a full activation will look like:

```Java
// Required
IDmeSDK.getInstance().startSDK(ClientID, ClientSecret);

// Optional
IDmeSDK.getInstance().setScannerKey(ScannerKey);
IDmeSDK.getInstance().setScannerUser(ScannerUser);
``` 

### Important Note
Once you've integrated, activated, and launched the SDK, please contact us at [mobile@id.me](mobile@id.me) so we may provide you sample data for verification.

## C. Authenticating a User
Authenticating a user is done via an activity that is started using the following method:

`IDmeSDK.getInstance().startAuthActivity(activity);`

where the parameter `activity` refers to your activity.

If a user successfully signs-in/up to ID.me, you will receive their entire profile in the `onActivityResult()` callback, where you should implement your `resultListener()`. It should look like the following:

```Java
protected void onActivityResult(int requestCode, int resultCode, Intent data)
{
  super.onActivityResult(requestCode, resultCode, data);

  HashMap profile = (HashMap)   IDmeSDK.getInstance().resultListener(requestCode, resultCode,
                data);
}
```

## D. Verifying a User's Attributes
Unlike authentication, the verification process has no pre-defined UI. We have chosen to give you, the developer, full access to rolling your own UI layer when presenting the end-user with a verification screen.

To streamline the process, all affinity verifications can be performed through a single method.

```Java
 IDmeSDK.getInstance().performAPIRequestOfType(activity,idmeRequestType,idmeFetchData)
```

where the three parameters are described below:

* `activity`: The activity from where the call is being made.
* `idmeRequestType`: The request to be made.
* `idmeFetchData` : The defined callback where data will be recieved.

### List of Affinity Groups (IDmeRequestType.AFFINITY_GROUPS)

```Java
IDmeSDK.getInstance().performAPIRequestOfType(this,
                IDmeRequestType.AFFINITY_GROUPS, null, new IDmeFetchData()
                {
                    @Override 
                    public void dataRetriever(Object apiResult)
                    {
                        if (apiResult instanceof List){
                            ArrayList<Map> result = (ArrayList<Map>)apiResult;
                            //Do Something with Results
                        }
                    }
                });
```
### Verify Military Service Record (Self)

```Java
HashMap<String, String> params = new HashMap<>();
        params.put(IDmeAPIParams.RELATIONSHIP, "self");
        params.put(IDmeAPIParams.SUBGROUP_ID, "6");// Any number obtained from IDmeAPIRequestTypeAffinityGroups
        params.put(IDmeAPIParams.SERVICE_MEMBER_FIRST_NAME, "<service_member_first_name>");
        params.put(IDmeAPIParams.SERVICE_MEMBER_LAST_NAME, "<service_member_last_name>");
        params.put(IDmeAPIParams.SOCIAL, "<9_digit_ssn>");
        params.put(IDmeAPIParams.BIRTH_DATE, "<MM/DD/YYYY>");
        params.put(IDmeAPIParams.SERVICE_DATE, "<MM/DD/YYYY>");
        
        IDmeSDK.getInstance().performAPIRequestOfType(activity,
        IDmeRequestType.MILITARY_VERIFY_SERVICE_RECORD, params,
                new IDmeFetchData<Object>()
                {
                    @Override public void dataRetriever(Object apiResult)
                    {
                         if (apiResult instanceof Map){
    						  HashMap map = (HashMap) apiResult;
                             
                              if(!map.containsKey("Error")){
                                //Do something with result
                            	}
    						 
    						}
                    }
                });
```

### Verify Military Service Record (Spouse)

```Java
HashMap<String, String> params = new HashMap<>();
        params.put(IDmeAPIParams.RELATIONSHIP, "spouse");
        params.put(IDmeAPIParams.SUBGROUP_ID, "6");// Any number obtained from IDmeAPIRequestTypeAffinityGroups
        params.put(IDmeAPIParams.SERVICE_MEMBER_FIRST_NAME, "<service_member_first_name>");
        params.put(IDmeAPIParams.SERVICE_MEMBER_LAST_NAME, "<service_member_last_name>");
        params.put(IDmeAPIParams.SOCIAL, "<9_digit_ssn>");
        params.put(IDmeAPIParams.BIRTH_DATE, "<MM/DD/YYYY>");
        params.put(IDmeAPIParams.SERVICE_DATE, "<MM/DD/YYYY>");
        params.put(IDmeAPIParams.FIRST_NAME, "<spouse's_first_name>");
        params.put(IDmeAPIParams.LAST_NAME, "<spouse's_last_name>");
        
        IDmeSDK.getInstance().performAPIRequestOfType(activity,
        IDmeRequestType.MILITARY_VERIFY_SERVICE_RECORD, params,
                new IDmeFetchData<Object>()
                {
                    @Override public void dataRetriever(Object apiResult)
                    {
                         if (apiResult instanceof Map){
    						  HashMap map = (HashMap) apiResult;
    						  
                             if(!map.containsKey("Error")){
                                //Do something with result
                            	}
    						 
    						}
                    }
                });
```

### Verify Military Service Record (Family Member)

```Java
HashMap<String, String> params = new HashMap<>();
        params.put(IDmeAPIParams.RELATIONSHIP, "family member");
        params.put(IDmeAPIParams.SUBGROUP_ID, "6");// Any number obtained from IDmeAPIRequestTypeAffinityGroups
        params.put(IDmeAPIParams.SERVICE_MEMBER_FIRST_NAME, "<service_member_first_name>");
        params.put(IDmeAPIParams.SERVICE_MEMBER_LAST_NAME, "<service_member_last_name>");
        params.put(IDmeAPIParams.SOCIAL, "<9_digit_ssn>");
        params.put(IDmeAPIParams.BIRTH_DATE, "<MM/DD/YYYY>");
        params.put(IDmeAPIParams.SERVICE_DATE, "<MM/DD/YYYY>");
        params.put(IDmeAPIParams.FIRST_NAME,"<family_member's_first_name>");
        params.put(IDmeAPIParams.LAST_NAME,"<family_member's_last_name>");
        
       IDmeSDK.getInstance().performAPIRequestOfType(activity,
        IDmeRequestType.MILITARY_VERIFY_SERVICE_RECORD, params,
                new IDmeFetchData<Object>()
                {
                    @Override public void dataRetriever(Object apiResult)
                    {
                    	if (apiResult instanceof Map){
    						  HashMap map = (HashMap) apiResult;
                             if(!map.containsKey("Error")){
                                //Do something with result
                            	}
    						}
                    }
                });
```

### List of Service Branches for USAA-based verification (IDmeAPIRequestType.MILITARY\_SERVICE\_BRANCHES)

```Java
IDmeSDK.getInstance().performAPIRequestOfType(activity,
IDmeRequestType.MILITARY_SERVICE_BRANCHES, null, new IDmeFetchData()
{
  @Override 
  public void dataRetriever(Object apiResult)
  {
    if (apiResult instanceof List){
     	ArrayList<Map> mapArrayList = (ArrayList<Map>) apiResult;
    	 //Do something with Result
    }
  }
});
```

### Verify Military Affinity using USAA Account (Self)

```Java
HashMap<String, String> params = new HashMap<>();
        params.put(IDmeAPIParams.RELATIONSHIP, "self");
        params.put(IDmeAPIParams.SUBGROUP_ID, "6");
        params.put(IDmeAPIParams.SERVICE_MEMBER_FIRST_NAME, "<service_member_first_name>");
        params.put(IDmeAPIParams.SERVICE_MEMBER_LAST_NAME, "<service_member_last_name>");
        params.put(IDmeAPIParams.SOCIAL, "<last_4_digits_ssn>");
        params.put(IDmeAPIParams.BIRTH_DATE, "<MM/DD/YYYY>");
        params.put(IDmeAPIParams.SERVICE_BRANCH_ID, "3"); // Any number obtained from `IDmeRequestType.MILITARY_SERVICE_BRANCHES`
        params.put(IDmeAPIParams.SERVICE_RANK_ID, "63"); // Any number obtained from `IDmeRequestType.MILITARY_SERVICE_BRANCHES`
        params.put(IDmeAPIParams.SERVICE_MONTH, "<MM>");
        params.put(IDmeAPIParams.SERVICE_YEAR, "<YYYY>");
        params.put(IDmeAPIParams.NON_ACTIVE_DUTY, "false");


        IDmeSDK.getInstance().performAPIRequestOfType(activity, IDmeRequestType.MILITARY_USAA,
                params, new IDmeFetchData()
                {
                    @Override public void dataRetriever(Object apiResult)
                    {
                        if (apiResult instanceof Map){
                            HashMap map = (HashMap) apiResult;
                             if(!map.containsKey("Error")){
                                //Do something with result
                            	}
                        }
                    }
                });
```

### Verify Military Affinity using USAA Account (Spouse)

```
HashMap<String, String> params = new HashMap<>();
        params.put(IDmeAPIParams.RELATIONSHIP, "self");
        params.put(IDmeAPIParams.SUBGROUP_ID, "6");
        params.put(IDmeAPIParams.SERVICE_MEMBER_FIRST_NAME, "<service_member_first_name>");
        params.put(IDmeAPIParams.SERVICE_MEMBER_LAST_NAME, "<service_member_last_name>");
        params.put(IDmeAPIParams.SOCIAL, "<last_4_digits_ssn>");
        params.put(IDmeAPIParams.BIRTH_DATE, "<MM/DD/YYYY>");
        params.put(IDmeAPIParams.SERVICE_BRANCH_ID, "3"); // Any number obtained from `IDmeRequestType.MILITARY_SERVICE_BRANCHES`
        params.put(IDmeAPIParams.SERVICE_RANK_ID, "63"); // Any number obtained from `IDmeRequestType.MILITARY_SERVICE_BRANCHES`
        params.put(IDmeAPIParams.SERVICE_MONTH, "<MM>");
        params.put(IDmeAPIParams.SERVICE_YEAR, "<YYYY>");
        params.put(IDmeAPIParams.NON_ACTIVE_DUTY, "false");
        params.put(IDmeAPIParams.FIRST_NAME, "<spouse's_first_name>");
        params.put(IDmeAPIParams.LAST_NAME, "<spouse's_last_name>");

        IDmeSDK.getInstance().performAPIRequestOfType(activity, IDmeRequestType.MILITARY_USAA,
                params, new IDmeFetchData()
                {
                    @Override public void dataRetriever(Object apiResult)
                    {
                        if (apiResult instanceof Map){
                            HashMap map = (HashMap) apiResult;
                             if(!map.containsKey("Error")){
                                //Do something with result
                            	}
                        }
                    }
                });
```

### Verify Military Affinity using USAA Account (Family Member)

```
HashMap<String, String> params = new HashMap<>();
        params.put(IDmeAPIParams.RELATIONSHIP, "self");
        params.put(IDmeAPIParams.SUBGROUP_ID, "6");
        params.put(IDmeAPIParams.SERVICE_MEMBER_FIRST_NAME, "<service_member_first_name>");
        params.put(IDmeAPIParams.SERVICE_MEMBER_LAST_NAME, "<service_member_last_name>");
        params.put(IDmeAPIParams.SOCIAL, "<last_4_digits_ssn>");
        params.put(IDmeAPIParams.BIRTH_DATE, "<MM/DD/YYYY>");
        params.put(IDmeAPIParams.SERVICE_BRANCH_ID, "3"); // Any number obtained from `IDmeRequestType.MILITARY_SERVICE_BRANCHES`
        params.put(IDmeAPIParams.SERVICE_RANK_ID, "63"); // Any number obtained from `IDmeRequestType.MILITARY_SERVICE_BRANCHES`
        params.put(IDmeAPIParams.SERVICE_MONTH, "<MM>");
        params.put(IDmeAPIParams.SERVICE_YEAR, "<YYYY>");
        params.put(IDmeAPIParams.NON_ACTIVE_DUTY, "false");
          params.put(IDmeAPIParams.FIRST_NAME, "<family_member's_first_name>");
        params.put(IDmeAPIParams.LAST_NAME, "<family_member's_last_name>");


        IDmeSDK.getInstance().performAPIRequestOfType(activity, IDmeRequestType.MILITARY_USAA,
                params, new IDmeFetchData()
                {
                    @Override public void dataRetriever(Object apiResult)
                    {
                        if (apiResult instanceof Map){
                            HashMap map = (HashMap) apiResult;
                             if(!map.containsKey("Error")){
                                //Do something with result
                            	}
                        }
                    }
                });
```

### Verify Military Affinity using .mil Email Address (Self)

```
HashMap<String, String> params = new HashMap<>();
        params.put(IDmeAPIParams.RELATIONSHIP, "self");
        params.put(IDmeAPIParams.SUBGROUP_ID, "6");
        params.put(IDmeAPIParams.SERVICE_MEMBER_FIRST_NAME, "<service_member_first_name>");
        params.put(IDmeAPIParams.SERVICE_MEMBER_LAST_NAME, "<service_member_last_name>");
        params.put(IDmeAPIParams.BIRTH_DATE, "<MM/DD/YYYY>");
          params.put(IDmeAPIParams.EMAIL, "<email_address_ending_in_.mil>");
        params.put(IDmeAPIParams.SERVICE_MONTH, "<MM>");
        params.put(IDmeAPIParams.SERVICE_YEAR, "<YYYY>");
        params.put(IDmeAPIParams.NON_ACTIVE_DUTY, "false");


        IDmeSDK.getInstance().performAPIRequestOfType(activity, IDmeRequestType.MILITARY_DOT_MIL_EMAIL,
                params, new IDmeFetchData()
                {
                    @Override public void dataRetriever(Object apiResult)
                    {
                        if (apiResult instanceof Map){
                            HashMap map = (HashMap) apiResult;
                             if(!map.containsKey("Error")){
                                //Do something with result
                            	}
                        }
                    }
                });
```

### Verify Military Affinity using .mil Email Address (Spouse)

```Java
HashMap<String, String> params = new HashMap<>();
        params.put(IDmeAPIParams.RELATIONSHIP, "self");
        params.put(IDmeAPIParams.SUBGROUP_ID, "6");
        params.put(IDmeAPIParams.SERVICE_MEMBER_FIRST_NAME, "<service_member_first_name>");
        params.put(IDmeAPIParams.SERVICE_MEMBER_LAST_NAME, "<service_member_last_name>");
        params.put(IDmeAPIParams.BIRTH_DATE, "<MM/DD/YYYY>");
          params.put(IDmeAPIParams.EMAIL, "<email_address_ending_in_.mil>");
        params.put(IDmeAPIParams.SERVICE_MONTH, "<MM>");
        params.put(IDmeAPIParams.SERVICE_YEAR, "<YYYY>");
        params.put(IDmeAPIParams.NON_ACTIVE_DUTY, "false");
        params.put(IDmeAPIParams.FIRST_NAME, "<spouse's_first_name>");
        params.put(IDmeAPIParams.LAST_NAME, "<spouse's_last_name>");

        IDmeSDK.getInstance().performAPIRequestOfType(activity, IDmeRequestType.MILITARY_DOT_MIL_EMAIL,
                params, new IDmeFetchData()
                {
                    @Override public void dataRetriever(Object apiResult)
                    {
                        if (apiResult instanceof Map){
                            HashMap map = (HashMap) apiResult;
                             if(!map.containsKey("Error")){
                                //Do something with result
                            	}
                        }
                    }
                });
```

### Verify Military Affinity using .mil Email Address (Family Member)

```Java
HashMap<String, String> params = new HashMap<>();
        params.put(IDmeAPIParams.RELATIONSHIP, "self");
        params.put(IDmeAPIParams.SUBGROUP_ID, "6");
        params.put(IDmeAPIParams.SERVICE_MEMBER_FIRST_NAME, "<service_member_first_name>");
        params.put(IDmeAPIParams.SERVICE_MEMBER_LAST_NAME, "<service_member_last_name>");
        params.put(IDmeAPIParams.BIRTH_DATE, "<MM/DD/YYYY>");
          params.put(IDmeAPIParams.EMAIL, "<email_address_ending_in_.mil>");
        params.put(IDmeAPIParams.SERVICE_MONTH, "<MM>");
        params.put(IDmeAPIParams.SERVICE_YEAR, "<YYYY>");
        params.put(IDmeAPIParams.NON_ACTIVE_DUTY, "false");
        params.put(IDmeAPIParams.FIRST_NAME, "<family_member's_first_name>");
        params.put(IDmeAPIParams.LAST_NAME, "<family_member's_last_name>");

        IDmeSDK.getInstance().performAPIRequestOfType(activity, IDmeRequestType.MILITARY_DOT_MIL_EMAIL,
                params, new IDmeFetchData()
                {
                    @Override public void dataRetriever(Object apiResult)
                    {
                        if (apiResult instanceof Map){
                            HashMap map = (HashMap) apiResult;
                             if(!map.containsKey("Error")){
                                //Do something with result
                            	}
                        }
                    }
                });
```
## E. Credential Scanning

Another aspect of the SDK is to scan and parse the information of dozens of physical credentials. The following credentials can be parsed:

* All U.S. State Licenses (incl. Washington, D.C.)
*  Common Access Cards (a.k.a. CACs)
  * Military CACs
  * Civilian CACs
* Uniformed Services ID Cards (also known as Geneva Conventions cards)
	* Cards held by active military personnel
	* Cards held by retired military personnel
	* Cards held by family members and dependents of military personnel

The following barcode is found the above-mentioned credentials. The scanner is fine-tuned to scan this specific bardocde:

![Barcode](Screenshots/readmeSampleBarcode.png)

The user is taught to scan this type of barcode on the first launch of the credential scanner. A help button is added to the main screen of the scanner, allowing the user to reference this information on the subsdequent launches of the scanner.

**Note**

* To scan physical credentials, you'll need credentials from [Manatee Works](https://manateeworks.com/).
* To parse those credentials, you'll need to be granted access by ID.me.
* Please contact us at [mobile@id.me](mobile@id.me) if you would like more information.

**Execution**

The credential scanner is presented in an activity. Once a credential is scanned, it is parsed in the background and data is returned in the `onActivityResult()` listener.

The scanner is activated with the following method:

```Java
IDmeSDK.getInstance().startScanner(Context);
``` 

This method has only one parameter, `Context`, which the context from which the scanner was called.

The results of the scanner will be sent to your `onActivityResult` listener for your convinience you can implement the SDK Result Listener and cast the result to `IDmeCredentialScannerResult` object.

```Java
protected void onActivityResult(int requestCode, int resultCode, Intent data)
{
    super.onActivityResult(requestCode, resultCode, data);

	IDmeCredentialScannerResult credentialScannerResult = 	
	(IDmeCredentialScannerResult) IDmeSDK.getInstance().resultListener(requestCode, resultCode,data);
	
	 	if(IDmeCredentialScannerResult.getErrorCode()== IDmeErrorTypes.NONE){
		   //Do something with result
	     }
}
```

If there are no errors, the following snippet of code may be helpful in your `if` statement:

```Java
 IDmeCredential idmeCredential=idmeCredentialScannerResult.getidmeCredential();
        
        if(idmeCredential instanceof IDmeStateLicense){
            IDmeStateLicense idmeStateLicense = (IDmeStateLicense)idmeCredential;
            // Extract data from stateLicense
        }
        else if(idmeCredential instanceof IDmeCommonAccessCard){
            IDmeCommonAccessCard idmeCommonAccessCard = (IDmeCommonAccessCard)idmeCredential;
            // Extract data from commonAccessCard

        }
        else if(idmeCredential instanceof IDmeUniformedServicesCard){
            IDmeUniformedServicesCard idmeUniformedServicesCard = (IDmeUniformedServicesCard)idmeCredential;
            // Extract data from uniformedServicesCard
        }
        else if(idmeCredential instanceof IDmeUniformedServicesDependentCard){
            IDmeUniformedServicesDependentCard idmeUniformedServicesDependentCard = (IDmeUniformedServicesDependentCard)idmeCredential;
            // Extract data from uniformedServicesCard
        }
        else if(idmeCredential instanceof IDmeUniformedServicesDependentCard){
            IDmeUniformedServicesDependentCard idmeUniformedServicesDependentCard = (IDmeUniformedServicesDependentCard)idmeCredential;
            // Extract data from uniformedServicesDependentCard
        }
```

### Credentials

**Hierarchy**
The data returned in `onActivityResult` in the aforementioned method are all instances or subclassed instances of the `IDmeCredential` class. For reference, the class hierarchy is as follows:

* `IDmeCredential`
  * `IDmeStateLicense` extends `IDmeCredential`
  * `IDmeDefenseCredential` extends `IDmeCredential`
    * `IDmeCommonAccessCard` extends `IDmeDefenseCredential`
    * `IDmeUniformedServicesCard` extends `IDmeDefenseCredential`
      * `IDmeUniformedServicesDependentCard` extends `IDmeUniformedServicesCard`

### Known Exceptions
* **Illinois**: Licenses from this state only encode the first-name, last-name and birthdate on the barcode.
* **Iowa**: License may not scan properly. We are working to address this issue.
* **North Dakota**: License may not scan properly. We are working to address this issue.
* **South Dakota**: License may not scan properly. We are working to address this issue.
* **Vermont**: License may not scan properly. We are working to address this issue.

## F. Error Handling
* **Authentication Request:** All authentication errors are handled by an alert dialog within the SDK.
* **API request errors:** All API errors are returned in a Hashmap with the key `Error`. If a map contains no `Error` key your request was successful.
* **Credential errors:** These are stored in `IDmeCredentialScannerResult.getErrorCode()`. The full list is contained in `IDmeErrorTypes`.

## G. Terms of Use
By using this SDK, you agree to [ID.me's Terms of Service](https://www.id.me/terms).

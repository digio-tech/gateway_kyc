## Digio Kyc Workflow SDK
### **How to Integrate?**
1. Add it in your root build.gradle at the end of repositories:

```
allprojects {
    repositories {
        ...
        maven { url 'https://jitpack.io' }
    }
}

plugins {
   id 'com.android.application' version '7.4.2' apply false
   id 'com.android.library' version '7.4.2' apply false
   id 'org.jetbrains.kotlin.android' version '1.7.20' apply false
   id 'androidx.navigation.safeargs' version '2.4.2' apply false
   id 'com.google.gms.google-services' version '4.3.10' apply false
}

```

2. Add the dependency and view/data Binding build config:

```
plugins {
   id 'com.android.application'
}

android {
   ...
   buildFeatures {
       viewBinding true
       dataBinding true
   }
}

dependencies {
    implementation 'com.github.digio-tech:gateway:v4.0.10'
    implementation 'com.github.digio-tech:gateway_kyc:v4.0.13'
    
    // Other dependencies
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.9.0'
    implementation 'androidx.navigation:navigation-fragment-ktx:2.5.3'
    implementation 'androidx.navigation:navigation-ui-ktx:2.5.3'
    // Added in version 4.0.6
    implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0'

    Following dependency is also not required if org.jetbrains.kotlin.android plugin version is 1.8.*
    implementation 'androidx.core:core-ktx:1.10.0'
    
    implementation 'androidx.exifinterface:exifinterface:1.3.6'
    implementation 'androidx.preference:preference:1.2.0'

    implementation "androidx.camera:camera-camera2:1.2.2"
    implementation "androidx.camera:camera-lifecycle:1.2.2"
    implementation "androidx.camera:camera-view:1.2.2"

    implementation "androidx.navigation:navigation-fragment-ktx:2.5.3"
    implementation "androidx.navigation:navigation-ui-ktx:2.5.3"

    // Use this dependency to use dynamically downloaded model in Google Play Service
    implementation 'com.google.android.gms:play-services-mlkit-face-detection:17.1.0'
    implementation 'com.google.android.gms:play-services-location:21.0.1'
}
```

3. Permissions : Add required permissions in manifest file and run time. Note - This is the common SDK for various KYC flows

```
<!--RECORD_AUDIO and MODIFY_AUDIO_SETTINGS Permission required for Video KYC -->
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />

/** Required for geo tagging */
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />


/** Required for ID card analysis, selfie  and face match**/
<uses-permission android:name="android.permission.CAMERA" />

<uses-feature android:name="android.hardware.camera" android:required="true" />
<uses-feature android:name="android.hardware.camera.autofocus"   android:required="false" />
```
A fintech android app can't access following permission
- Read_external_storage
- Read_media_images
- Read_contacts
- Access_fine_location
- Read_phone_numbers
- Read_media_videos

**Note**: All required permission should be define in manifest file as well as take runtime permission, digio kyc sdk also handles the runtime permission starting from sdk version 4.0.1 please go through how to request permission
Permissions on Android , optionally required permissions should be asked before starting the workflow.

4. After updating dependencies click File =>  Sync Project with Gradle Files


**Digio SDK supports android version 5.0 and above (SDK level 21 above)**


**Note - For hybrid applications, you need to create a channel/bridge/plugin to communicate with the native SDK.**




### **Steps to Invoke Kyc workflow**
1. Configure Digio instances : should be called on activity/fragment onCreate
```
DigioWorkflowSession digioWorkflowSession = new DigioWorkflowSession();

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    DigioConfig digioConfig = new DigioConfig();
    DigioTheme theme = new DigioTheme();
    theme.setPrimaryColor(android.R.color.holo_red_dark);
    theme.setFontFamily("Unbounded");
    theme.setFontUrl("https://fonts.googleapis.com/css2?family=Unbounded:wght@200&display=swap");
    digioConfig.setTheme(theme);
    digioConfig.setLogo("https://www.digio.in/images/digio_blue.png"); // Your company logo url
    digioConfig.setEnvironment(DigioEnvironment.SANDBOX); // SANDBOX or PRODUCTION
    digioConfig.setServiceMode(DigioServiceMode.OTP);  // FP/OTP/IRIS
    try {
        digioWorkflowSession.init(this, digioConfig);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```


2. DigioResponseListener and import onDigioSuccess, onDigioFailure override function in your activity/fragment. Below are function signatures

```
@Override
public void onWorkflowSuccess(@NonNull WorkflowResponse workflowResponse) {
    Log.d("Response", "onDigioKycSuccess--> documentId " + workflowResponse.getDocumentId() + " message " + workflowResponse.getMessage());
}

@Override
public void onWorkflowFailure(@NonNull WorkflowResponse workflowResponse) {
    Log.d("Response", "onDigioKycFailure--> documentId " + workflowResponse.getDocumentId() + " message " + workflowResponse.getMessage());
    if (workflowResponse.getErrorCode() != null && workflowResponse.getErrorCode() == DigioErrorCode.DIGIO_PERMISSIONS_REQUIRED.getErrorCode()) {
        String[] requiredPermissions = workflowResponse.getPermissions();
        if (requiredPermissions != null && requiredPermissions.length > 0) {
            this.requiredPermissions = requiredPermissions;
            // Digio provides list of permission which is required to complete the workflow.
            // take necessary action whether close the process or ask permission and restart workflow session.
            // explain user why permission is required and how to enable permission.
           
        }
    }
}

@Override
public void onGatewayEvent(@NonNull GatewayEvent gatewayEvent) {
    System.out.println("gatewayEvent = " + gatewayEvent);
}
```

3. Starting the sign flow
```
try {
    // First args : Document Id starting with KID
    // Second args : Identifier (Email/Mobile)
    // Third args (optional) : GWT token to skip the first factor authentication
    digioWorkflowSession.start("KID***", "**Email/Mobile**", "GWT***");
} catch (Exception e) {
    e.printStackTrace();
}
```


4. **Proguard :** No action is required for latest stable android studio, proguard-rules are already added to the sdk.
#### It is required to test the release build for possible proguard exceptions before prod releases.


#### **App crash after starting digio flow :**
- Make sure init is called before start and all the parameter values are proper as the documentation.
- There is no missing dependency as described above.


[Check out our demo App implementation](https://drive.google.com/file/d/1ZWSKIi_dkkalPohsoSwBNQQeWLp9UyJp/view?usp=sharing)

Demo App Apks : [PRODUCTION](https://drive.google.com/file/d/1E2irUHo1xCa21uChgcl4cKLG7RD6YsBS/view?usp=sharing) , [SANDBOX](https://drive.google.com/file/d/18DEALMpZMSAVnJjIuraYu_CavzGWpY5c/view?usp=sharing)


### **Environment**


|<a name="_g2lkuhshpqkl"></a>**DigioEnvironment**  |<p>SANDBOX</p><p>PRODUCTION</p>|Mandatory|
| :- | :- | :- |

### **Parameter and uses**


|**Params**|**Param Type**|**Reason**|
| :- | :- | :- |
|TokenId|Optional|TokenId is optional to avoid first factor authentication|
|requestId|Mandatory|You can get it from kyc request APIs|
|Identifier|Mandatory|Identifier can be email or phone number. Which has passed to the kyc request APIs.|

### **Sample Success Response**


|requestId|KID211022123015947IKIP3BM1R1RE1Y|String format (Request ID Passed by parent app)|
| :- | :- | :- |
|message|Success|String format|


### **Sample Failure Response**


|requestId|KID211022123015947IKIP3BM1R1RE1Y|String format (Request ID Passed by parent app)|
| :- | :- | :- |
|message|<p>{</p><p>“screen":"starting\_digio",</p><p>“code”:-1000,</p><p>"error\_code":-1000,</p><p>"message":"User cancelled before completion."</p><p>“permissions”: [],</p><p>“stack\_trace” : “”</p><p>}</p>|JSON format|

### **Possible error codes and reason :**


|**Code**| **Reason**                                                                                                                                            |
| :- |:------------------------------------------------------------------------------------------------------------------------------------------------------|
|-1000| User Cancellation before KYC is complete.                                                                                                             |
|1002| <p>KYC Failed</p><p>Webview Loading Errors</p>                                                                                                        |
|1003| System Webview crash due to low memory or version compatibility issue (Added in V4.0.6)                                                               |
|1004| Digio SDK crashes due to unknown reason. Check stactTrace param of WorkflowResponse, (Added in V4.0.6)                                                |
|1008| <p>Required permission not provided</p><p>(permissions : Array<String></p><p>param in WorkflowResponse permissions which required and user denied)</p>|


**Gateway Events :**

Refer Gateway document for all posible events and error data : [Gateway Event Doc](https://docs.google.com/document/d/15LHtjGyXd_JNM0de8uH9zB7WllJikRl1d9e4qdy0-C0)



DigioEvent<br>`    `documentId: string;<br>`    `txnId: string;<br>`    `entity: string;<br>`    `identifier: string;<br>`    `event: string;<br>`    `payload: <br>`        `type: 'error' | 'info';<br>`        `data?: HashMap<String,Any>;<br>`        `error?: <br>`            `code: string;<br>`            `message: string;



**Webview Error Codes and messages. The following webview errors are already handled by digio SDK, The name,description and message will be displayed during sdk journey if any internet connectivity issue happens.**

|**name**|**error code description from webview client**|**error code**|**messages (manually mapped, error\_code may differ)**|
| :- | :- | :- | :- |
|*ERROR\_UNKNOWN*|*Generic error*|-1|<p>net::ERR\_FAILED</p><p>net::ERR\_CACHE\_MISS</p>|
|*ERROR\_HOST\_LOOKUP*|*Server or proxy hostname lookup failed*|-2|<p>net::ERR\_INTERNET\_DISCONNECTED</p><p>net::ERR\_NAME\_RESOLUTION\_FAILED</p><p>net::ERR\_NAME\_NOT\_RESOLVED</p>|
|*ERROR\_UNSUPPORTED\_AUTH\_SCHEME*|*Unsupported authentication scheme (not basic or digest)*|-3|net::ERR\_ACCESS\_DENIED|
|*ERROR\_AUTHENTICATION*|*User authentication failed on server*|-4||
|*ERROR\_PROXY\_AUTHENTICATION*|*User authentication failed on proxy*|-5||
|*ERROR\_CONNECT*|*Failed to connect to the server*|-6|<p>net::ERR\_CONNECTION\_FAILED</p><p>net::ERR\_ABORTED</p><p>net::ERR\_CONNECTION\_ABORTED</p><p>net::ERR\_CONNECTION\_CLOSED</p>|
|*ERROR\_IO*|*Failed to read or write to the server*|-7|<p>net::ERR\_INVALID\_RESPONSE</p><p>net::ERR\_EMPTY\_RESPONSE</p><p>net::ERR\_CONTENT\_DECODING\_FAILED</p><p>net::ERR\_CONTENT\_LENGTH\_MISMATCH</p>|
|*ERROR\_TIMEOUT*|*Connection timed out*|-8|net::ERR\_CONNECTION\_TIMED\_OUT|
|*ERROR\_REDIRECT\_LOOP*|*Too many redirects*|-9|net::ERR\_TOO\_MANY\_REDIRECTS|
|*ERROR\_UNSUPPORTED\_SCHEME*|*Unsupported URI scheme*|-10|net::ERR\_UNKNOWN\_URL\_SCHEME|
|*ERROR\_FAILED\_SSL\_HANDSHAKE*|*Failed to perform SSL handshake*|-11|<p>net::ERR\_SSL\_PROTOCOL\_ERROR</p><p>net::ERR\_CERT\_DATE\_INVALID</p><p>net::ERR\_CERT\_AUTHORITY\_INVALID</p><p>net::ERR\_BAD\_SSL\_CLIENT\_AUTH\_CERT</p><p>net::ERR\_SSL\_CLIENT\_AUTH\_CERT\_NEEDED</p>|
|*ERROR\_BAD\_URL*|*Malformed URL*|-12|net::ERR\_INVALID\_URL|
|*ERROR\_FILE*|*Generic file error*|-13|net::ERR\_FILE\_TOO\_BIG|
|*ERROR\_FILE\_NOT\_FOUND*|*File not found*|-14|net::ERR\_FILE\_NOT\_FOUND|
|*ERROR\_TOO\_MANY\_REQUESTS*|*Too many requests during this load*|-15||
|*ERROR\_UNSAFE\_RESOURCE*|*Resource load was canceled by Safe Browsing*|-16|net::ERR\_INSUFFICIENT\_RESOURCES|
||||net::ERR\_INVALID\_HANDLE|
||||net::ERR\_CONTENT\_DECODING\_FAILED|
||||net::ERR\_CONTENT\_LENGTH\_MISMATCH|




### Change Logs
- **Version 4.0.11 => 4.0.13**
  -  UIDAI OKYC changes


- **Version 4.0.11**
    - Fixed camera target resolution issue when used with flutter.

- **Version 4.0.9 :**
    - Removed firebase crashlytics
    - For reverse penny drop added upi apps intent flow.

- **Version 4.0.8 :**
  - Removed firebase crashlytics
  - For reverse penny drop added upi apps intent flow.

- **Version 4.0.6 :**
    - Introduced webview connection error handling with in digio sdk.
    - Added internet connection observability, if internet connection get disconnected message will be displayed with in digio sdk.
    - Digio activity will run in portrait mode only and will not re-create on any system configuration changes
    - Digio Activity will not run in separate process, there is no requirement of any handling in Application class.
    - Introduced Gateway events


### Migration Guide
- **4.0.6 => 4.0.9**
  - Remove firebase crashlytics dependencies and gradle plugin if not required by app or other sdk's.
  
- **4.0.3 => 4.0.6**
    - implemented onGatewayEvent on DigioWorkflowResponseListener
    - add following dependencies
    - add  ```implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0'``` to build.gradle




**Note:** Digio reserves the right to modify this API document from time-to-time. If you are a business using this API, you will be notified  well in advance, prior to any change is made

© Copyright 2016-23 | [www.digio.in](http://www.digio.in) | For Limited Circulation


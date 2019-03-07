# Modirum 3DS SDK: Getting Started
March 7th, 2019

Confidential

This document and its content is property of Modirum and any information contained herein may not be shared or given to third parties without prior written permission from authorized Modirum representative.

# 1 Overview

This document serves as an initial guide to integrating the Modirum 3-D Secure SDK (later, Modirum SDK) into an Android or iOS mobile application. More details about the technical implementation can be found in the **Modirum 3DS SDK Technical Guide**.

The Modirum SDK is an implementation of the **EMV 3-D Secure SDK Specification**. Please see the Technical Guide for the SDK version-to-specification mapping table.

In addition to the EMV 3DS SDK Core, the Modirum SDK includes support for direct communication with the Modirum MPI using the HTTP POST interface as described in the **Modirum MDpay 3-D Secure MPI Interface Description (versions 3.0 and 4.0)**. However, this document focuses only on the EMV 3DS SDK Core functionality. Please contact Modirum for additional details about the MPI communication feature.

# 2 Installation

An SDK release package includes the the following:

For Android,
- Android AAR library file
- License file

For iOS,
- iOS Framework zip file
- Podspec
- License file

## 2.1 Android

### 2.1.1 SDK

The Android Modirum SDK library is copied into the `app/libs` directory of the Android studio project for the Merchant Application and the following lines included in the `build.gradle` (Module:app) configuration file: 

```
repositories {
    mavenCentral()
    flatDir {
        dirs 'libs' //this way sdk.aar file can be found in libs folder
    }
}
dependencies {
    :
    :
    implementation 'com.modirum.threedsv2:modirum-sdk-development:1.0.0@aar'
                                                //should be the appropriate aar filename ("sdk" from "sdk.aar")
                                                //but the actual value of the "1.0.0" version is irrelevant
                                                //since the aar is being loaded locally from the libs folder
}
```

If the Merchant Application project will use proguard, the following lines have to be added to the application proguard configuration file, ex. proguard-rules.pro:

```
-dontwarn com.modirum.threedsv2.**
-keep class com.modirum.threedsv2.** { *;}
```

### 2.1.2 License

During initialization, the SDK tries to locate the license file from a fixed location in the application assets.
The "modirum_license.json" file has to be included in the root directory of the application asset folder.
(Ex. `app/src/main/assets/modirum_license.json`)

## 2.2 iOS

### 2.2.1 SDK

The iOS Modirum SDK can be installed directly as a framework or via Pods. To see the details for direct installation, please see the Technical Guide. However, installation via Pods is recommended.

Copy the Framework zip file and Podspec to a desired location within the Merchant App project.

ex. 
```
<Merchant_App_root>/LocalPods/MI_SDK_DEVELOPMENT.framework.zip
<Merchant_App_root>/LocalPods/MI_SDK_DEVELOPMENT.podspec
```

Update the Merchant App Podfile to include the Modirum SDK framework.
```
pod 'MI_SDK_DEVELOPMENT', :path => 'LocalPods/MI_SDK_DEVELOPMENT.podspec'
```

**Objective-C**

To access the SDK, just use this import in Merchant Application:

```
#import <MI_SDK_DEVELOPMENT/MI_SDK_DEVELOPMENT.h>  //should be the appropriate framework name
```

**Swift**

To access the SDK from a Merchant Application that is implemented using Swift, just use this import:

```
import MI_SDK_DEVELOPMENT  //should be the appropriate framework name
```

### 2.2.2 License

During initialization, the SDK tries to locate the license file from a fixed location in the application bundle resources.
The "modirum_license.json" file has to be included in the root directory of the application bundle.


# 3 EMV 3DS Transaction

The following figure shows an overview of an EMV 3DS transaction. For the details of each step, please see the EMV 3DS Specification or the Modirum SDK Technical Guide.

#### Figure 1 - Overview of EMV 3-D Secure transaction

![figure1](https://user-images.githubusercontent.com/4307756/46349698-4d9ec180-c685-11e8-8b55-8362fac5e9d5.png)
*(Please refer to “Figure 2 - Overview of EMV 3-D Secure transaction” of the Modirum 3DS SDK Technical Guide)*


## 3.1 Android

### 3.1.1 SDK Service Initialization

Merchant Application creates new instances of ThreeDSecurev2Service (the Modirum 3DS SDK implementation of the ThreeDS2Service interface), ConfigParameters and UICustomization as follows:

```java
ThreeDS2Service service = new ThreeDSecurev2Service(); 
ConfigParameters configParam = new ConfigParameters(); 
UiCustomization uiConfig = new UiCustomization();
```

#### 3.1.1.1 Applying UICustomization

EMV 3-D Secure offers a possibility for the Merchant to customize the native UI pages shown by the SDK. The Merchant can bring their own look-and-feel to the native pages. Note that the UI customization does not apply, if the issuer wants to present the challenge dialogue in HTML format.

Should the Merchant wish to have its own look-and-feel in the challenge pages, the UICustomization class is used. For example, the following sample would set the challenge page UI as:
- 10px rounded corners on text input filed
- Monospace 18px font as label font
- Sets the toolbar background as dark red with white 28px font
- Sets the Verify button to have 10px rounding on corners and dark red background with white text

The example below shows the UI Customization as an example for Android implementation.

```java
UiCustomization uiConfig = new UiCustomization();
TextBoxCustomization textBoxCustomization = new TextBoxCustomization(); 
textBoxCustomization.setCornerRadius(10); 
uiConfig.setTextBoxCustomization(textBoxCustomization);

LabelCustomization labelCustomization = new LabelCustomization(); 
labelCustomization.setTextFontName(Typeface.MONOSPACE.toString()); 
labelCustomization.setTextFontSize(18); 
uiConfig.setLabelCustomization(labelCustomization);

ToolbarCustomization toolbarCustomization = new ToolbarCustomization(); 
toolbarCustomization.setBackgroundColor("#951728"); 
toolbarCustomization.setTextColor("#FFFFFF"); 
toolbarCustomization.setTextFontSize(28); 
uiConfig.setToolbarCustomization(toolbarCustomization);

ButtonCustomization buttonCustomization = new ButtonCustomization(); 
buttonCustomization.setBackgroundColor("#951728"); 
buttonCustomization.setCornerRadius(10); 
buttonCustomization.setTextColor("#FFFFFF");
uiConfig.setButtonCustomization(buttonCustomization, UiCustomization.ButtonType.SUBMIT);
```

#### 3.1.1.2 Call to initialize ThreeDSecurev2Service

Finally, the Merchant Application can call the `initialize()`-method of the ThreeDS2Service object.

```java
service.initialize(currentActivity, configParam, locale, uiConfig);
```

Depending on implementation, calling the initialize()-method may take some time and possibly cause a short delay. Thus, it is recommended to call it in a separate thread.

After the `initialize()`-method has been called, the Merchant Application may call the `getWarnings()`-method of the ThreeDS2Service to query possible warnings generated during initialization.

```java
List<Warning> warnings = service.getWarnings();
```

### 3.1.2 Transaction initiation and the AReq/ARes phase

Once the cardholder proceeds to the check-out in the Merchant Application, the actual EMV 3-D Secure transaction may be started.

The transaction is created using the `createTransaction()`-method of the `ThreeDS2Service` object service. This returns an instance of `Transaction`, through which Merchant Application can access the transaction data.

```java
Transaction transaction = service.createTransaction(directoryServerId, messageVersion);
```

#### 3.1.2.1 Progress dialog

While the Merchant Application is communicating with the Merchant Environment, a progress dialog is displayed. `Transaction` implements a `ProgressDialog` class, which can be accessed through `getProgressView()`-method.

The example below creates `progressDialog` object and displays it in the current activity.

```java
ProgressDialog progressDialog = transaction.getProgressView(currentActivity); 
progressDialog.show();
```

#### 3.1.2.2 Authentication Request Parameters

During the construction of the AReq, some fields can be obtained from the SDK using the transaction `getAuthenticationRequestParameters`-method.

```java
AuthenticationRequestParameters authRequestParams = transaction.getProgressView(currentActivity);
String deviceData = authRequestParams.getDeviceData();
String sdkTransID = authRequestParams.getSDKTransactionID();
String sdkAppID = authRequestParams.getSDKAppID();
String sdkReferenceNumber = authRequestParams.getSDKReferenceNumber();
String messageVersion = authRequestParams.getMessageVersion();
JWK sdkJwk = ECKey.parse(authenticationRequestParameters.getSDKEphemeralPublicKey());
JSONObject *sdkEphemPubKey = new JSONObject(sdkJwk.toJSONString());
```

There is no interface defined in the EMV 3DS SDK Specification to provide the Device Rendering Options Supported. However, the Modirum SDK supports all device rendering options.

Sample Device Rendering Options Supported for AReq:
```
{
  "deviceRenderOptions":{ "sdkInterface":"03", "sdkUiType":["01", "02", "03", "04", "05"] }
}
```

### 3.1.3 Challenge dialogue

Based on information received from the 3DS Server, the Merchant Environment has to decide if a Challenge is needed. If so, the Merchant Application needs to pass the required data to the SDK so that the SDK can complete the challenge dialogue.

To perform the actual challenge, the Merchant Application calls the `transaction.doChallenge()`- method with the appropriate `ChallengeStatusReceiver`.

The example below shows one way of implementing this phase in the Merchant Application.

```java
String transactionStatus = authenticationResponse.getString(ProtocolConstants.TransactionStatus); 
boolean challenge = transactionStatus.contentEquals("C");

if (challenge) —> 
ChallengeParameters challengeParameters = new ChallengeParameters(); 
challengeParameters.set3DSServerTransactionID(authenticationResponse.getString(ProtocolConstants.ThreeDSServerTransID));
challengeParameters.setAcsTransactionID(authenticationResponse.getString(ProtocolConstants.ACSTransactionID)); 
challengeParameters.setACSSignedContent(authenticationResponse.getString(ProtocolConstants.ACSSignedContent));
challengeParameters.setAcsRefNumber(authenticationResponse.getString(ProtocolConstants.ACSReferenceNumber));
transaction.doChallenge(currentActivity, challengeParameters,
  new ChallengeStatusReceiver() { 
    @Override
    public void completed(CompletionEvent e) { 
      log.debug("<- Completed");
      //At this point, the Merchant app can contact the 3DS Server to 
      //determine the result of the challenge
    }
    @Override
    public void cancelled() {
      log.debug("<- Cancelled");
      //can go to Cancelled view if desired
    }
    @Override
    public void timedout() {
      log.debug("<- timedout"); 
      //can show error alert 
    }
    @Override
    public void protocolError(ProtocolErrorEvent e) {
      log.debug("<- protocolError"); 
      //can show error alert 
    }
    @Override
    public void runtimeError(RuntimeErrorEvent e) {
      log.debug("<- runtimeError"); 
      //can show error alert 
    }
}, timeout );
```

### 3.1.4 Ending the Transaction

For challenge flows, the transaction object is automatically closed. However, for frictionless flows, the transaction object has to be closed explicitly.

```java
transaction.close();
```

### 3.1.5 SDK Service Cleanup

If the Merchant App only plans to use the SDK during certain situations and not always for the whole lifetime of the app, it can choose to initialize/cleanup the SDK service when desired. However, note that initializing the SDK may take some time since payment system data has to be loaded into memory, device data collected, and any security warnings checked.

To cleanup the SDK service:

```java
service.cleanup(applicationContext);
```


## 3.2 iOS


### 3.2.1 SDK Service Initialization

Merchant Application creates new instances of ThreeDSecurev2Service (the Modirum 3DS SDK implementation of the ThreeDS2Service interface), ConfigParameters and UICustomization as follows:

**Objective-C**

```objc
id<ThreeDS2Service> service = [[ThreeDSecurev2Service alloc] init]; 
ConfigParameters *configParam = [[ConfigParameters alloc] init]; 
UiCustomization *uiConfig = [[UiCustomization alloc] init];
```

**Swift**

```
let service : ThreeDSecurev2Service = ThreeDSecurev2Service() 
let configParam : ConfigParameters = ConfigParameters()
let uiConfig : UiCustomization = UiCustomization()
```

#### 3.2.1.1 Applying UICustomization

EMV 3-D Secure offers a possibility for the Merchant to customize the native UI pages shown by the SDK. The Merchant can bring their own look-and-feel to the native pages. Note that the UI customization does not apply, if the issuer wants to present the challenge dialogue in HTML format.

Should the Merchant wish to have its own look-and-feel in the challenge pages, the UICustomization class is used. For example, the following sample would set the challenge page UI as:
- 10px rounded corners on text input filed
- Courier 18px font as label font
- Sets the toolbar background as dark red with white 28px font
- Sets the Verify button to have 10px rounding on corners and dark red background with white text

The example below shows the UI Customization as an example for iOS implementation.

**Objective-C**

```objc
UiCustomization *uiConfig = [[UiCustomization alloc] init];
TextBoxCustomization *textBoxCustomization = [[TextBoxCustomization alloc] init];
[textBoxCustomization setCornerRadius:10];
[uiConfig setTextBoxCustomization:textBoxCustomization];

LabelCustomization *labelCustomization = [[LabelCustomization alloc] init];
[labelCustomization setTextFontName:@"Courier"]; 
[labelCustomization setTextFontSize:18]; 
[uiConfig setLabelCustomization:labelCustomization];

ToolbarCustomization *toolbarCustomization = [[ToolbarCustomization alloc] init]; 
[toolbarCustomization setBackgroundColor:@"#951728"]; 
[toolbarCustomization setTextColor:@"#FFFFFF"]; 
[toolbarCustomization setTextFontSize:28]; 
[uiConfig setToolbarCustomization:toolbarCustomization];

ButtonCustomization *buttonCustomization = [[ButtonCustomization alloc] init];
[buttonCustomization setBackgroundColor:@"#951728"]; 
[buttonCustomization setCornerRadius:10]; 
[buttonCustomization setTextColor:@"#FFFFFF"];
[uiConfig setButtonCustomization:buttonCustomization buttonType:SUBMIT];
```

**Swift**

```
let uiConfig : UiCustomization = UiCustomization();
let textBoxCustomization : TextBoxCustomization = TextBoxCustomization()
textBoxCustomization.setCornerRadius(10)
uiConfig.setTextBox(textBoxCustomization)

let labelCustomization : LabelCustomization = LabelCustomization()
labelCustomization.setTextFontName("Courier") 
labelCustomization.setTextFontSize(18) 
uiConfig.setLabel(labelCustomization)

let toolbarCustomization : ToolbarCustomization = ToolbarCustomization()
toolbarCustomization.setBackgroundColor("#951728")
toolbarCustomization.setTextColor("#FFFFFF")
toolbarCustomization.setTextFontSize(28)
uiConfig.setToolbar(toolbarCustomization)

let buttonCustomization: ButtonCustomization = ButtonCustomization()
buttonCustomization.setBackgroundColor("#951728")
buttonCustomization.setCornerRadius(10) 
buttonCustomization.setTextColor("#FFFFFF")
uiConfig.setButton(buttonCustomization, buttonType: .SUBMIT)
```

#### 3.2.1.2 Call to initialize ThreeDSecurev2Service

Finally, the Merchant Application can call the `initialize()`-method of the ThreeDS2Service object.

**iOS – Objective-C**

```objc
[service initialize:configParam locale:locale uiCustomization:uiConfig];
```

**iOS – Swift**

```
service.initialize(configParam, locale:locale, uiCustomization:uiConfig)
```

Depending on implementation, calling the initialize()-method may take some time and possibly cause a short delay. Thus, it is recommended to call it in a separate thread.

After the `initialize()`-method has been called, the Merchant Application may call the `getWarnings()`-method of the ThreeDS2Service to query possible warnings generated during initialization.

**iOS – Objective-C**

```objc
NSArray *warnings = [service getWarnings];
```

**iOS – Swift**

```
service.getWarnings() as! Array<Warning>
```

### 3.1.2 Transaction initiation and the AReq/ARes phase

Once the cardholder proceeds to the check-out in the Merchant Application, the actual EMV 3-D Secure transaction may be started.

The transaction is created using the `createTransaction()`-method of the `ThreeDS2Service` object service. This returns an instance of `Transaction`, through which Merchant Application can access the transaction data.

**iOS – Objective-C**

```objc
Transaction *transaction = [service createTransaction:directoryServerId messageVersion:msgVersion];
```

**iOS – Swift**

```
let transaction: Transaction = service.createTransaction(directoryServerId, messageVersion: msgVersion)
```

#### 3.1.2.1 Progress dialog

While the Merchant Application is communicating with the Merchant Environment, a progress dialog is displayed. `Transaction` implements a `ProgressDialog` class, which can be accessed through `getProgressView()`-method.

The example below creates `progressDialog` object and displays it in the current ViewController.

**iOS - Objective-C**

```objc
id<ProgressDialog> progressDialog = [transaction getProgressView]; 
[progressDialog show];
```

**iOS - Swift**

```
let progressDialog : ProgressDialog = transaction.getProgressView()! 
progressDialog.show()
```

#### 3.1.2.2 Authentication Request Parameters

During the construction of the AReq, some fields can be obtained from the SDK using the transaction `getAuthenticationRequestParameters`-method.

**iOS - Objective-C**

```objc
AuthenticationRequestParameters *authRequestParams = [transaction getAuthenticationRequestParameters];
NSString *deviceData = authRequestParams.deviceData;
NSString *sdkTransID = authRequestParams.sdkTransactionID;
NSString *sdkAppID = authRequestParams.sdkAppID;
NSString *sdkReferenceNumber = authRequestParams.sdkReferenceNumber;
NSString *messageVersion = authRequestParams.messageVersion;
NSDictionary* sdkJwk = [NSJSONSerialization JSONObjectWithData:[authRequestParams.sdkEphemeralPublicKey dataUsingEncoding:NSUTF8StringEncoding] options:0 error:&err];
```

**iOS - Swift**

```
let authRequestParams : AuthenticationRequestParameters = transaction.getAuthenticationRequestParameters()
let deviceData : String = authRequestParams.deviceData
let sdkTransID : String = authRequestParams.sdkTransactionID
let sdkAppID : String = authRequestParams.sdkAppID
let sdkReferenceNumber : String = authRequestParams.sdkReferenceNumber
let messageVersion : String = authRequestParams.messageVersion
let sdkEphemPubKey : String = authRequestParams.sdkEphemeralPublicKey
```

There is no interface defined in the EMV 3DS SDK Specification to provide the Device Rendering Options Supported. However, the Modirum SDK supports all device rendering options.

Sample Device Rendering Options Supported for AReq:
```
{
  "deviceRenderOptions":{ "sdkInterface":"03", "sdkUiType":["01", "02", "03", "04", "05"] }
}
```

### 3.1.3 Challenge dialogue

Based on information received from the 3DS Server, the Merchant Environment has to decide if a Challenge is needed. If so, the Merchant Application needs to pass the required data to the SDK so that the SDK can complete the challenge dialogue.

To perform the actual challenge, the Merchant Application calls the `transaction.doChallenge()`- method with the appropriate `ChallengeStatusReceiver`.

The example below shows one way of implementing this phase in the Merchant Application.

**iOS -- Objective-C**

```objc
NSString* status = [authenticationResponse objectForKey:KTransactionStatus];
if ([status isEqualToString:@"C"]) {
    ChallengeParameters *challengeParameters = [[ChallengeParameters alloc] init];
    challengeParameters.acsSignedContent = [authenticationResponse objectForKey:KACSSignedContent];
    challengeParameters.threeDSServerTransactionID = [authenticationResponse objectForKey:KThreeDSServerTransID];
    challengeParameters.acsTransactionID = [authenticationResponse objectForKey:KACSTransactionID];
    challengeParameters.acsRefNumber = [authenticationResponse objectForKey:KACSReferenceNumber];
    
    [transaction doChallenge:challengeParameters challengeStatusReceiver:delegate timeOut:txnTimeout];
: 

// ------ challengeStatusReceiver delegate -----

- (void) completed:(CompletionEvent *)e {
    SDKLogI(@"<- Completed");
    //At this point, the Merchant app can contact the 3DS Server to 
    //determine the result of the challenge
}

- (void) cancelled {
    SDKLogI(@"<- Cancelled");
    //can go to Cancelled view if desired
}

- (void) timedout {
    SDKLogI(@"<- timedout");
    //can show error alert 
}

- (void) protocolError:(ProtocolErrorEvent *)e {
    SDKLogI(@"<- protocolError"); 
    //can show error alert 
}

- (void) runtimeError:(RuntimeErrorEvent *)e {
    SDKLogI(@"<- runtimeError"); 
    //can show error alert 
}
```

**iOS - Swift**

```
if (aRes.transStatus == "C") {
    let challengeParameters = ChallengeParameters()
    challengeParameters.setAcsSignedContent(aRes.acsSignedContent!)
    challengeParameters.setAcsRefNumber(aRes.acsReferenceNumber!)
    challengeParameters.setAcsTransactionID(aRes.acsTransID!)
    challengeParameters.set3DSServerTransactionID(aRes.threeDSServerTransID!)
    transaction.doChallenge(challengeParameters, challengeStatusReceiver: delegate, timeOut: Int32(timeOut))
: 

// ------ challengeStatusReceiver delegate -----

func completed(_ e: CompletionEvent) {
    SDKLogI("<- Completed")
    //At this point, the Merchant app can contact the 3DS Server to 
    //determine the result of the challenge
}

func cancelled() {
    SDKLogI("<- Cancelled")
    //can go to Cancelled view if desired
}

func timedout() {
    SDKLogI("<- timedout")
    //can show error alert 
}

func protocolError(_ e: ProtocolErrorEvent) {
    SDKLogI("<- protocolError");
    //can show error alert 
}

func runtimeError(_ e: RuntimeErrorEvent) {
    SDKLogI("<- runtimeError") 
    //can show error alert 
}
```

### 3.1.4 Ending the Transaction

For challenge flows, the transaction object is automatically closed. However, for frictionless flows, the transaction object has to be closed explicitly.

**iOS - Objective-C**

```objc
[transaction close];
```

**iOS - Swift**

```
transaction.close()
```

### 3.1.5 SDK Service Cleanup

If the Merchant App only plans to use the SDK during certain situations and not always for the whole lifetime of the app, it can choose to initialize/cleanup the SDK service when desired. However, note that initializing the SDK may take some time since payment system data has to be loaded into memory, device data collected, and any security warnings checked.

To cleanup the SDK service:

**iOS - Objective-C**

```objc
[service cleanup];
```

**iOS - Swift**

```
service.cleanup()
```

# 4 Development SDK

For initial development and integration with a Modirum SDK and MPI, a development SDK is provided that is configured for the Modirum test environment (MPI, DS, ACS). 

| | Development SDK | Production SDK |
|-- |-- |-- |
| **sdkReferenceNumber** | internal value configured in Modirum test DS | official EMVCo-issued values |
| **directoryServerID parameter for `createTransaction`** | Internal values: VISA, MASTERCARD, AMEX, DINERSCLUB, JCB | Registered Application Provider Identifier (RID) that is unique to the Payment System. RIDs are defined by the ISO 7816-5 standard. |
| **Directory Server public keys** | internals keys for Modirum test DS | official public keys provided by the brands |
| **ACS Server certificate validation** | disabled | enabled |
| **acsSignedContent JWS x5c cert chain validation** | can be self-signed | must be signed by DS CA |
| **debug logs** | enabled | disabled (only info/warning/error-level logs are shown) |
| **screenshots (Android)** | enabled | disabled |

Please contact Modirum to obtain test environment access and accounts.



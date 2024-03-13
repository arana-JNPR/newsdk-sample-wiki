<h1> Welcome to the Mist iOS Location SDK Integration Guide! </h1>

## Table of Contents:     
1. [Introduction](#Introduction)   
2. [System requirements](#system-requirements)   
3. [Installation](#installation)
    - [Installation using Cocoa-pods](#installation-using-cocoa-pods)  
    - [Installation using Swift Package Manager](#installation-using-swift-package-manager)
    - [Installation - Manually using framework](#installation---manually-using-framework)
4. [Concepts](#concepts)
    1. [App Permissions](#app-permissions)
    1. [Mobile SDK Token](#access-token)
    1. [Floor Plan](#floor-plan)
    1. [Position Access Points](#positioning-ap)
    1. [WayFinding Paths](#the-path-towards-the-floor-plan---for-wayfinding-use-case)
    1. [Virtual Beacon](#virtual-beacon)
    1. [Zones](#zones)
    1. [App Wakeup](#app-wakeup)                 
5. [Integration steps](#integration-steps)    
6. [Methods Available in IndoorLocationManager](#methods-available-in-indoorlocationmanager)
7. [FAQs](https://github.com/mistsys/mist-vble-ios-sdk/wiki/FAQs)    
      
   

# Introduction

The Indoor Location Service and Outdoor Location Service differ significantly. This is because most Outdoor Location Services rely on GPS and other Global Navigation Satellite Systems (GNSS), which are obstructed by buildings' roofs and walls. Consequently, GPS is not a suitable solution for our case study in terms of accuracy.
To attain indoor location services, we utilize MistSDK, which employs BLE technology for indoor positioning, wayfinding systems, proximity notification, and other features.

<u><b>MistSDKDR - Mist Software Development Kit Dead Reckoning.</u></b>
> By leveraging Mist's 16 vBLE antenna array Access point, Mist SDK facilitates an indoor blue dot experience. With this SDK, you will have the ability to determine the user's location and deliver proximity-based notifications utilizing Mist's patented vBeacon technology.<br><br>__Why the word Dead Reckoning?__ <br>
MistSDK employs a dead reckoning technique to compute the user's current location. It does so by using their previous location and extrapolating it based on known and estimated speeds over elapsed time. This implies that DR is devised to make use of all available location data that a user had when connected and continues to function as though the client is connected, even when facing disconnections or interruptions in the connection.

<br>
<u>Important features offered by our Mist SDK: </u>

- Indoor Location,
    - Providing the coordinates in the map for your app to draw the blue dot location of the device.
- Virtual Beacon Notification.
    - A beacon, which is not physically present, but a virtual one. A virtual beacon that is added on the web portal and known by your app through the SDK. It provides notifications when the device is near the location of the virtual beacon.
- Virtual Zone notification.
    - Zones are defined on the Mist portal as areas within a map. The SDK provides notifications on going into a zone. 
- Wayfinding Path Information
    - Wayfinding paths are defined on the Mist portal. The SDK provides the coordinate information from the current blue dot to the nearest point in the paths, through the shortest path to a given coordinate in the same map. This information can be used by the app to draw the shortest route a user can take to a destination.


## System requirements  

1. Hardware Requirements:
    <ol type="a">
    <li> <a href=https://www.juniper.net/us/en/products/access-points.html?filters=ble-antenna:vble-array>Mist Access Point with vBLE support (AP)</a> - very important</li>
    - Mist uses an AP with 16 vBLE Antennas to get the position of device in the floor.
    <li><b>iOS Mobile Device</b>: Since the iOS Simulator doesn’t have Bluetooth support, it will not work properly. We need a physical device with BLE support for application testing and development with the SDK.</li>
    </ol>
 
2. Software Requirements:
    <ol type="a">
    <li> Xcode: 12 or later. </li>
    <li> iOS deployment Target: 14.0 or later. </li>
    <li> Access to <a href="https://manage.mist.com/signin.html#!signIn">Mist Account </a>
    <li> Mobile SDK secret</li>
    </ol>


# Installation 
## Installation using Cocoa-pods         
Follow the steps to integrate Cocoa-Pod in the Xcode project or [[Refer CocoaPods Getting Started guide]](https://cocoapods.org/):         
1. Open the terminal and change directory to your project folder, and create and initialize Podfile using the commands below.  

    ```bash
    $ cd ./project_folder_path
    ```
    - If pod not installed, run the below command
        ```bash
        $ sudo gem install cocoapods
        ```
2. Initilize project 
    ```bash
    $pod init
    ```
    - this will create a folder and required files in the project directory

2. Open the Podfile created and add the package to the target section in the pod file.

    ```pod
    source 'https://github.com/CocoaPods/Specs.git'
    source 'https://github.com/mistsys/mist-vble-ios-sdk.git'

    platform :ios, '14.0'

    target 'MyApp' do
    #Pods for MyApp
        pod 'MistSDK', '2.0.8'
    end
    ```

3. Go back to the terminal and Run the command in your project directory to complete the SDK installation with Cocoapods.
    ```bash
    $pod install
    ```
4. Open <<<em><b>projectname</b></em>>>.___xcworkspace___ and build. This way, you can get the benefit of the pod and its installed packages.        

5. Add __import MistSDK__ in swift OR __#import <MistSDK/MistSDK.h>__ in Objective c to get started.


> Note: Incase Mist SDK DR version is not compliant with bitcode. You need to set BitCode -> Enable = No in the build setting of your XCode Project.  

##  Installation using Swift Package Manager

Once you have your Swift package set up, adding MistSDK as a dependency is as easy as adding it to the dependencies value of your Package.swift.

dependencies: [ .package(url: "https://github.com/mistsys/mist-vble-ios-sdk.git", from: "2.0.8") ]

##  Installation - Manually using the framework

* For the latest SDK version, download the [MistSDK.framework](https://github.com/mistsys/mist-vble-ios-sdk/releases)           
* Drag and drop the MistSDK.framework in the project file. 
* In your Xcode project file add MistSDK.framework to Targets > General > Framework, Libraries and Embedded contents > select Embed and Sign.          
* Add `__mport MistSDK__ in swift OR __#import <MistSDK/MistSDK.h>__ in Objective c . 


# Concepts 
#### Things needed to make SDK ready to work.
There are some of the important steps to be configured initially to get started with an SDK
1. App Permissions.    
2. Mobile SDK token.
3. Floor Plan - (MistMap)
4. Position Access Points (APs) in the floor plan

And the following are necessary on the need basis,
1. Way-finding Paths
2. Virtual Beacons
3. Zones
4. App Wakeup

## App Permissions

To proceed, our SDK gathers certain sensitive information, which is deemed necessary. For instance, the SDK leverages Bluetooth to scan for BLE devices. Therefore, it is crucial to provide sufficient permissions to begin. These permissions include:
1. __Location Permission.__: 
    1. Privacy - Location When in Use Usage Authorization.
        - To listen for iBeacon and other location services in foreground.
    1. Privacy - Location Always Usage Authorization.
        - To listen for iBeacon and other location services even in background.
2. *Bluetooth Permission*:
    1. Privacy - Bluetooth Peripheral Usage Description.
        - To listen and make use of Bluetooth in foreground.
    1. Privacy - Bluetooth Always Usage Description.
        - To listen and make use of Bluetooth in background.
    
To provide those permissions open ___info.plist___ in a source code mode and add the following snippet.
```
<key>NSLocationWhenInUseUsageDescription</key>
<string> Mist wants to access the Location services </string> <key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string> Mist wants to access the Location services in background </string> <key>NSBluetoothPeripheralUsageDescription</key>
<string> Mist wants to access the Bluetooth </string> <key>NSBluetoothAlwaysUsageDescription</key>
<string> Mist wants to access the Bluetooth in background </string>
```
Now we are done with adding permissions. The next step is to create a Mobile SDK token to enroll the device in the Mist Cloud.

## Mobile SDK Token

Mobile SDK token is the secret key which can be created from the [Mist portal](https://manage.mist.com/signin.html#!signIn). This key authenticates an SDK instance and identifies that it belongs to a specific organization in the Mist cloud.

<u>Why is this Mobile SDK token for?</u>: <br>
    Mist provides the solution for indoor location services. With the Mobile SDK token we ensure that SDK instances are only used for specific organizations in the Mist cloud.
    <br>
    <u> Token Creation Steps:</u>
- To create a Mobile SDK token, you should have registered with the [Mist portal](https://manage.mist.com/signin.html#!signIn) 
- Go to Organization --> Mobile SDK as,        

<img width="646" alt="Screenshot 2023-04-15 at 8 26 15 PM" src="https://user-images.githubusercontent.com/122332297/232232499-aaa34657-c2e0-4a86-ba3c-75faa2b15c40.png">

- Create a new invitation. It will ask for the organization name and once you give the name and hit save, you will get your token, Refer the image for better understanding.
        
<img width="646" alt="Screenshot 2023-04-15 at 8 26 15 PM" src="https://user-images.githubusercontent.com/122332297/232232538-4dcb2445-b54c-47ec-b8b1-8f2bbaddf16a.png">



## Floor plan - (MistMap)

- Our goal is to provide indoor location services. This requires a map, specifically a floor plan.
- In the Mist portal, we can upload the floor plans. We call these the MistMaps.

<u>Maps Supported</u>
- Mist Map: Map Type (jpeg, png, gif)
- Micello
- Jibestream

Conversion from Mist Coordinates to other maps __Jibestream__:<br>
jibestreamX = (mistX*1000)/mmpp
jibestreamY = (mistY*1000)/mmpp

Where mmpp is MilliMeterPerPixel which is specific to map and will be provided by jibestream itself

<u>How to add the Floor Plan in Mist Portal:</u>

- Go to __Location--> Live View__ – here it will show all of the existing floorplans, if there are none then it will tell you to add the floorplan with a button.
<img width="731" alt="Screenshot 2023-04-15 at 8 26 32 PM" src="https://user-images.githubusercontent.com/122332297/232232555-c68cd442-fd37-4d93-8a4f-0399a120972d.png">


- To add a floor plan, click on the __"Add FloorPlan"__ or __"Import FloorPlan"__ button based on your requirements. You will be prompted to provide a name for the floor. Once you have entered the name, press __"Enter"__, and you will be asked to upload the floor plan image. Simply drag and drop the image into the designated area and click on the __"Upload"__ button.
<img width="540" alt="Screenshot 2023-04-15 at 8 26 39 PM" src="https://user-images.githubusercontent.com/122332297/232232572-84a5fe71-919d-437b-8881-5ec656be39fa.png">


- The final step in configuring the floor plan is to add the PPM (Pixels per meter) value. This indicates the distance in meters that each pixel on the floor plan represents.
    
## Positioning Access Points

Positioning the AP in the Mist Portal exactly as it is in the real world is crucial. Failing to do so will result in an inaccurate Mist Point.

<u>How to position APs in the floor plan in the <b>Mist Portal</b></u>

Go to __Location -> Live View -> Setup Floorplan__, from here you can see the list of APs that are claimed to that site and move them around to whereever you want. 

This is one of the examples of floor plan with AP installed.

<img width="601" alt="Screenshot 2023-04-15 at 8 26 46 PM" src="https://user-images.githubusercontent.com/122332297/232232592-96ab55a3-d677-4585-8a27-bdc85eb06344.png">


## Wayfinding Paths

Wayfinding paths can be created to help the location estimate gravitate towards the paths drawn out. The wayfinding paths can also be used by your app to draw the shortest route for wayfinding. To generate the path, we can use the Mist portal and define the route directly on the map. For that

- Go to Location -> Live View -> Choose the floor plan
- Click the __Wayfinding paths__ button
<img width="477" alt="Screenshot 2023-04-15 at 8 26 50 PM" src="https://user-images.githubusercontent.com/122332297/232232602-4307212f-6ede-48dc-b529-7e3fddd932db.png">

- You will see the Following options
<img width="265" alt="Screenshot 2023-04-15 at 8 26 54 PM" src="https://user-images.githubusercontent.com/122332297/232232612-16ebbbc8-3fb5-49ee-9fa3-e43119b56c0c.png">

- Draw paths in areas where there will be people walking and moving through; be sure to draw paths in every area where you intend to utilize the SDK clients. Make sure that all path segments are connected to each other.
<img width="310" alt="Screenshot 2023-04-15 at 8 26 59 PM" src="https://user-images.githubusercontent.com/122332297/232232619-f0b3a1b5-9b9a-45bd-8b50-2547c6395116.png">

for more [https://www.mist.com/documentation/adding-wayfinding-paths/]

## Virtual Beacons

vBeacons are a Mist-patented technology that enables the provision of proximity-related information. The advantage of vBeacons is that they can be placed anywhere on the floor plan without concerns about battery life or relocation issues. Similar to real beacons, ranging APIs can be used for vBeacons. It’s important to note that these do not act as stand-ins for real beacons and are only for proximity-related notification information. Meaning that placing these vBeacons on your floorplan will not help increase accuracy.

__To add the vBeacons,__
- Go to __Location->Live View->__ select the floor plan->click the __Beacons and Zones__
<img width="512" alt="Screenshot 2023-04-15 at 8 30 28 PM" src="https://user-images.githubusercontent.com/122332297/232232669-7b02012f-0cbc-4774-9fa5-08f1109508cc.png">

- Now the user can create and move the vBeacon wherever he wants. All the Green circles are vBeacons
<img width="334" alt="Screenshot 2023-04-15 at 8 27 05 PM" src="https://user-images.githubusercontent.com/122332297/232232694-1e353cfa-7e93-4bea-b953-15e40cad7a46.png">


- Furthermore, we have the flexibility to customize the beacon according to your requirements. If you wish to remove a beacon, you can choose the "__Remove__" option, or if you want to make any modifications to the beacon, you can select the "__Edit__" option.
<img width="330" alt="Screenshot 2023-04-15 at 8 27 09 PM" src="https://user-images.githubusercontent.com/122332297/232232706-fa801797-5392-4e7d-bd4c-6ee8c0058bf4.png">


- If you intend to edit a beacon, you should fill the necessary details in the edit popup and save the changes. On the other hand, if you wish to delete a beacon, simply click the "__Delete__" button.
<img width="508" alt="Screenshot 2023-04-15 at 8 27 13 PM" src="https://user-images.githubusercontent.com/122332297/232232711-5f4103b5-d0e6-4829-a782-0f80942be72d.png">



## Zones

- A zone is a customized area defined on the Mist map that enables notifications to be sent to provide location-specific information to customers. We can send notifications when a user enters the specified zone in the Mist portal.
- To add zones, Go to Location->Live View-> select the floor plan->click the Beacons and Zones.
- And click add location Zone and drag the area on the floor plan to create the zone.
- Refer the highlighted area in the image for the zone.
<img width="282" alt="Screenshot 2023-04-15 at 8 27 18 PM" src="https://user-images.githubusercontent.com/122332297/232232716-94a229c9-3229-4747-ac6e-afe83d879977.png">
    
As same as virtual beacons, we have options to edit and delete the zones.

## App Wakeup

App wakeup refers to the act of bringing the app back to an active state when it enters the range of a registered beacon. This feature is primarily related to the app itself and not the Mist SDK. However, you can initiate the SDK in the background when you receive the callback in the killed state and utilize it for analytics purposes instead of navigation or location.

For wakeup we are using the [Apple CLBeaconRegion API using iBeacon](https://developer.apple.com/documentation/corelocation/determining_the_proximity_to_an_ibeacon) in iOS.
Apple had a method didEnterRegion and didExitRegion delegate methods in Core Location
package to handle such a scenario better.
For Android we are using ALTBeacon to achieve AppWakeup.


<br> 

# Integration Steps:       

1. Import MistSDK

    _Swift_:
    ```swift 
    import MistSDK
    ```
    _Obj-c_:
    ```objc
    #import <MistSDK/MistSDK.h>
    ```

2. Create and initialize the SDK with the Mobile SDK token that you get through the token creation process.

    _Swift_:
    ```swift 
    self.sdkConfig = Mist.Configuration(sdkToken: MistConstants.mistAccessToken, logLevel: .debug, enableLog: true)
    self.mistManager = IndoorLocationManager.shared
    ```
    _Obj-c_:
    ```objc
    self.indoorLocationManager = [IndoorLocationManager sharedInstance: @"<AccessToken>"];

    ```

3. Start the location SDK.

4. Assign an IndoorLocationDelegate.
    
    _Swift_:
    ```swift 
    mistManager.start(with: sdkConfig, delegate: self)
    ```
    _Obj-c_:
    ```objc
    [_indoorLocationManager startWithIndoorLocationDelegate:self];

    ```

5. Once we have done all the steps, we will be able to get all the required parameters in the respective delegate methods.

6. There are roughly around five delegates available, and they are 
    <ol type ="a">
        <li>IndoorLocationDelegate.</li>
        <li>ZonesDelegate.</li>
        <li>MapsListDelegate.</li>
        <li>VirtualBeaconsDelegate.</li>
        <li>ClientInformationDelegate.</li>
    </ol>
    <br><br>

### IndoorLocationDelegate
<u> ```didReceive()``` method needs to be overridden to handle all MistSDK callbacks using events with ```case``` </u>



```swift
 func didReceive(event: Mist.Event) {
        switch event {
            case .callbackName():
                // handle callbacks here
```

The following are the callbacks in detail:

1.  To get client information when enrolment sucessfull or  client name has been updated.
    ```swift
    case onReceivedClientInfo(MistSDK.Mist.Client)
    ```

    ><center> Mist.Client </center>
    | Property | Datatype | Description |
    |----------|----------|-------------|
    | name     | String?  | The name of the client. |
    | orgId    | String?  | The organization ID associated with the client. |
    | orgName  | String?  | The organization name associated with the client. |
    | deviceId | String?  | The device ID of the client. |

2.  To get current map data
    ```swift
    case onMapUpdate(MistSDK.Mist.Map)
    ```
3.  To receive all sites maps
    ```swift
    case onReceivedAllMaps([MistSDK.Mist.Map])
    ```
    ><center> Mist.Map </center>
    | Property             | Datatype               | Description                                                                 |
    |----------------------|------------------------|-----------------------------------------------------------------------------|
    | id                   | String?                | The ID of the map.                                                          |
    | name                 | String?                | The name of the map.                                                        |
    | siteId               | String?                | The site ID associated with the map.                                        |
    | orgId                | String?                | The organization ID associated with the map.                                 |
    | type                 | String?                | The type of the map.                                                        |
    | locked               | Bool?                  | Indicates whether the map is locked.                                        |
    | originX              | Double?                | The x-coordinate origin of the map.                                          |
    | originY              | Double?                | The y-coordinate origin of the map.                                          |
    | width                | Double?                | The width of the map.                                                       |
    | height               | Double?                | The height of the map.                                                      |
    | ppm                  | Double?                | The pixels per meter ratio of the map.                                      |
    | widthM               | Double?                | The width of the map in meters.                                             |
    | heightM              | Double?                | The height of the map in meters.                                            |
    | createdTime          | Int64?                 | The timestamp of when the map was created.                                  |
    | modifiedTime         | Int64?                 | The timestamp of when the map was last modified.                            |
    | url                  | String?                | The URL of the map.                                                         |
    | thumbnailUrl         | String?                | The URL of the map thumbnail.                                               |
    | wayFindingPath       | MistSDK.Mist.Path?     | The path data for wayfinding on the map.                                    |
    | wallPath             | MistSDK.Mist.Path?     | The path data for walls on the map.                                         |
    | siteSurveyPath       | [MistSDK.Mist.Path]?  | The path data for site survey on the map.                                   |
    | wayfinding           | MistSDK.Mist.Wayfinding? | Information about wayfinding on the map.                                   |
    | latlngBr             | MistSDK.Mist.LatlngBr? | The bottom-right latitude and longitude coordinates of the map.           |
    | latlngTl             | MistSDK.Mist.LatlngTl? | The top-left latitude and longitude coordinates of the map.               |
    | drInternal           | MistSDK.Mist.DRInternal? | Internal data related to dynamic refresh for the map.                      |
    | geoRefParamsv2       | MistSDK.Mist.GeoRefParamsV2? | Georeferencing parameters for the map.                                 |
    | orientation          | Int?                   | The orientation of the map.                                                 |
    | occupancyLimit       | Int64?                 | The occupancy limit of the map.                                             |
    | intendedCoverageAreas| [[String : String]]?   | The intended coverage areas of the map.                                     |


4. To know when the client  is in the range of  a VirtualBeacon
    ```swift
    case onRangeVirtualBeacon(MistSDK.Mist.VirtualBeacon)
    ```

5. To get all  Virtual Beacons for the current map
    ```swift
    case onUpdateVirtualBeaconList([MistSDK.Mist.VirtualBeacon])
    ```

    ><center> Mist.VirtualBeacon </center>
    | Property        | Datatype                    | Description                                                                  |
    |-----------------|-----------------------------|------------------------------------------------------------------------------|
    | orgId           | String?                     | The organization ID associated with the virtual beacon.                      |
    | siteId          | String?                     | The site ID associated with the virtual beacon.                              |
    | mapId           | String?                     | The map ID associated with the virtual beacon.                               |
    | vbId            | String?                     | The ID of the virtual beacon.                                               |
    | vbUUID          | String?                     | The UUID of the virtual beacon.                                             |
    | vbMajor         | Int?                        | The major value of the virtual beacon.                                      |
    | vbMinor         | Int?                        | The minor value of the virtual beacon.                                      |
    | power           | Int?                        | The power of the virtual beacon.                                            |
    | powerMode       | String?                     | The power mode of the virtual beacon.                                       |
    | message         | String?                     | A message associated with the virtual beacon.                               |
    | createdTime     | Int64?                      | The timestamp of when the virtual beacon was created.                       |
    | modifiedTime    | Int64?                      | The timestamp of when the virtual beacon was last modified.                  |
    | tagId           | String?                     | The tag ID associated with the virtual beacon.                              |
    | name            | String?                     | The name of the virtual beacon.                                             |
    | url             | String?                     | The URL associated with the virtual beacon.                                 |
    | position        | MistSDK.VBeaconPosition?    | The position information of the virtual beacon.                             |
    | additionalInfo  | MistSDK.Mist.AdditionalInfo?| Additional information about the virtual beacon.                            |


6.  To get current mist point
    ```swift
    case onRelativeLocationUpdate(MistSDK.Mist.Point)
    ```

    ><center> Mist.Point </center>
    | Property   | Datatype  | Description                                       |
    |------------|-----------|---------------------------------------------------|
    | map        | MistSDK.Mist.Map | The map associated with the point.     |
    | x          | Double    | The x-coordinate of the point.                   |
    | y          | Double    | The y-coordinate of the point.                   |
    | lat        | Double    | The latitude of the point.                       |
    | lon        | Double    | The longitude of the point.                      |
    | latency    | Int64     | The latency associated with the point.           |
    | heading    | Double    | The heading (direction) of the point.            |
    | speed      | Double    | The speed associated with the point.             |
    | hasMotion  | Bool      | Indicates whether the point has motion.          |


7.  To receive zone notification onEnter
    ```swift
    case onEnterZone(MistSDK.Mist.Zone)
    ```

8. To receive zone notification onExit
    ```swift
    case onExitZone(MistSDK.Mist.Zone)
    ```
    ><center> Mist.Zone </center>
    | Property       | Datatype              | Description                                                   |
    |----------------|-----------------------|---------------------------------------------------------------|
    | name           | String?               | The name of the zone.                                         |
    | timestamp      | String?               | The timestamp associated with the zone.                       |
    | origin         | String?               | The origin of the zone.                                       |
    | level          | Int?                  | The level of the zone.                                        |
    | recipient      | String?               | The recipient of the zone.                                    |
    | recipientId    | String?               | The ID of the recipient of the zone.                           |
    | userId         | String?               | The ID of the user associated with the zone.                  |
    | zoneId         | String?               | The ID of the zone.                                           |
    | mapId          | String?               | The ID of the map associated with the zone.                   |
    | siteId         | String?               | The site ID associated with the zone.                         |
    | orgId          | String?               | The organization ID associated with the zone.                 |
    | clientType     | String?               | The type of client associated with the zone.                  |
    | isRandom       | Bool?                 | Indicates if the zone is random.                              |
    | zonePosition   | MistSDK.ZonePosition? | The position information of the zone.                         |
    | trigger        | MistSDK.Mist.ZoneTrigger? | The trigger associated with the zone.                      |
    | time           | String?               | The time associated with the zone.                             |
    | sinceTime      | String?               | The since time associated with the zone.                      |
    | timeEpoch      | Int64?                | The epoch time associated with the zone.                      |
    | sinceTimeEpoch | Int64?                | The epoch time since associated with the zone.                |
    | sessionId      | String?               | The session ID associated with the zone.                      |
    | duration       | Int64?                | The duration associated with the zone.                        |



-------------------------
-------------------------
-------------------------
-------------------------
# old documentation below 

<u>didUpdateRelativeLocation</u>
1. ```swift
    func didUpdateRelativeLocation(_ relativeLocation: MistPoint!) -> Void
    ```
1. ```swift
    func didReceivedOrgInfo(withTokenName tokenName: String!, andOrgID orgID: String!) -> Void 
    ```



This delegate method is a part of __IndoorLocationDelegate__ which is very important to be overridden. So, the class whichever is trying to confirm __IndoorLocationDelegate__ should override this method.

The main purpose of this method is to provide __relativeLocation__ of type __MistPoint__, which gives us detailed information about the current location of the device.

><center> MistPoint </center>
| Property | Datatype | What it infers| 
|--|---|-----|
| x| double | It is a spatial coordinate x|
| y | double | It is a spatial coordinate y.
| lat | NSNumber | It mentions the current latitude of the device.
| lon | NSNumber | It mentions the current longitude of the device.
| hasMotion | bool | It tells us whether the device has a motion or not.
| latency | double | It provides the delay in getting the data.
| heading | NSNumber | It provides direction towards AP in degrees.
| map | MistMap | It provides all the information related to MistMap.

<u>Example:</u>

_Swift_:
```swift 
func didUpdateRelativeLocation(_ relativeLocation: MistPoint!){ 
    self.mistPoint = relativeLocation
}
```
_Obj-c_:
```objc
 (void)didUpdateRelativeLocation:(MistPoint *)relativeLocation { 
    _mistPoint = relativeLocation;
}
```

<u>didUpdate(_ map: MistMap).</u>

It will be called once the map gets updated – which means that, if the device moves from one floor to another floor, there will be a change in the floor map. In such a scenario this callback will be triggered with an updated floor map.
It is also the required delegate method and should be overridden without fail.

><center> MistMap </center>
| Property | Datatype | What it infers| 
|--|---|-----|
name | String | It is the name of the map (Floor plan).
Width, height | long | Width and height of the map.
mapID | String | Unique identifier for the map.
siteID | String | Site is a venue where our MistSDK is running – and this id refers to the unique identifier for the site.
orgID | String | Organization ID.
url | String | This is the url which contains the actual map.
wayfindingPath | MistPath | It provides the path towards our goal destination.

><center> MistPath </center>
| Property | Datatype | What it infers| 
|--|---|-----|
coordinate | String | It is spatial coordinate.
name | String | Name of the route which is visible on the map
mapID | String | Unique identifier for the map.
nodes | list : PathNode | List of nodes to be traversed.
pathID | String | Unique id for the path.

<u>Example:</u>

_Swift_:
```swift 
 func didUpdate(_ map: MistMap!) { 
    self.mistMap = map
}
```
_Obj-c_:
```objc
 (void)didUpdateMap:(MistMap *)map { 
    _mistMap = map;
}
```

<u>didErrorOccur(_ withError:,withErrorMessgage).</u>

This method will get called if the SDK faces any error rather than getting any positive response. And it will help us by giving two values
1. errorMessage of type NSString,
2. errorType of type ErrorType Enum.

><center> ErrorType </center>
| Property | What it infers| 
|----|----|
.noResponse | This will be thrown when Mist cloud cannot send a response to the device.
.responseParseError| This will be thrown, if we could not be able to parse the response from Mist server.
.noBeaconsDetected | It will be thrown when there is no beacon available to scan.
.noConnectionToCloud | It will be thrown when we cannot be able to establish a connection to our __Mist cloud__.
.noDataConnection | It will be thrown when our device has no internet connection.
.authFailure | If SDK could not authorize, we will get this issue.
.serverOverloaded | This error will be thrown if our server is already overloaded, and it cannot process our request.
.sdkNotStarted | This will be thrown when the SDK itself is not running.

<br>

<u>Example:</u>

_Swift_:
```swift 
    func didErrorOccur(with errorType: ErrorType, andMessage errorMessage: String!) {
        switch errorType{
            case .noResponse:
            // Handle noResponse Error
            case .responseParseError:
            // Handle responseParseError Error
            case .noBeaconsDetected:
            // Handle noBeaconsDetected Error
            case .noConnectionToCloud:
            // Handle noConnectionToCloud Error
            case .noDataConnection:
            // Handle noDataConnection Error
            case .authFailure:
            // Handle authFailure Error
            case .serverOverloaded:
            // Handle serverOverloaded Error
            case .sdkNotStarted:
            // Handle sdkNotStarted Error
        } 
}
```
_Obj-c_:
```objc
    (void)didErrorOccurWithType:(ErrorType)errorType andMessage:(NSString *)errorMessage{
        switch (errorType) {
            case ErrorTypeNoResponse:
            // handle ErrorTypeNoResponse break;
            case ErrorTypeResponseParseError:
            // handle ErrorTypeResponseParseError break;
            case ErrorTypeNoBeaconsDetected:
            // handle ErrorTypeNoBeaconsDetected break;
            case ErrorTypeNoConnectionToCloud:
            // handle ErrorTypeNoConnectionToCloud break;
            case ErrorTypeNoDataConnection:
            // handle ErrorTypeNoDataConnection break;
            case ErrorTypeAuthFailure:
            // handle ErrorTypeAuthFailure break;
            case ErrorTypeServerOverloaded:
            // handle ErrorTypeServerOverloaded break;
            case ErrorTypeSDKNotStarted:
            // handle ErrorTypeSDKNotStarted break;
        } 
    }
```

<u>didReceivedOrgInfo</u>

This method will get called if the SDK can successfully authenticate our organization with an access token.
It will give is the following values
1. __TokenName__ of type __String__.
2. __OrgID__ of type __String__.

<u>Example:</u>

_Swift_:
```swift 
    func didReceivedOrgInfo(withTokenName tokenName: String!, andOrgID orgID: String!) { 
        self.tokenName = tokenName
        self.orgID = orgID
    }
```
_Obj-c_:
```objc
    (void)didReceivedOrgInfoWithTokenName:(NSString *)tokenName andOrgID:(NSString *)orgID{
        _orgID = orgID;
        _tokenName = tokenName; 
    }

```

##### <u>ZonesDelegate.</u>

As we already gone through, a zone is a custom area defined on the Mist map to get the notification to provide vicinity related information to your customer. And this delegate will get called when a device enters the defined zone area in the Mist portal.
- ```swift 
    func didEnter(_ mistZone: MistZone!) -> Void
    ```
- ```swift 
    func didExitZone(_ mistZone: MistZone!) -> Void
    ```

<u>didEnter(_ mistZone: MistZone)</u>

This delegate method will get called when the app enters our monitoring region -
(the area where we implemented our Mist System).

<u>Example:</u>

_Swift_:
```swift 
     func didEnter(_ mistZone: MistZone!) { 
        print("Entered into MistZone \(mistZone)")
    }
```
_Obj-c_:
```objc
    (void)didEnterZone:(MistZone *)mistZone{
        printf("Entered into the zone %s", mistZone.debugDescription);
    }
```

<u>didExitZone(_ mistZone: MistZone)</u>

This delegate method will get called when the app exits from our monitoring region - (the area where we implemented our Mist System).
Both above methods give us a __mistZone__ object of type __MistZone.__

<u>Example:</u>

_Swift_:
```swift 
    func didExitZone(_ mistZone: MistZone!) { 
        print("Exit from MistZone \(mistZone)")
    }
```
_Obj-c_:
```objc
    (void)didExitZone:(MistZone *)mistZone{
        printf("Exit from the zone %s", mistZone.debugDescription);
    }
```

> Note: Both methods are optional in this protocol- means that no need to override if any class conforming to ZonesDelegate but if we want to have a better experience, use this delegate as much effective as possible.

<br>

><center> MistZone </center>
| Property | Datatype | What it infers| 
|--|---|-----|
name | String | Name of the Zone.
userID | String | Name of the Zone.
orgID, siteID | String | They are just an identifier of the organization and a site where mist is implemented.
zoneID | String | Unique Identifier for the zone.
sinceTime | String | It tells from what time user is inside the zone.
sinceTimeEpoch | long | It tells and gives the timer, and it says how long the device is active in a zone.
sessionID | String | SDK Maintains a session in backend. And this id indicates the session of the current device inside the zone.
zonePosition | ZonePosition | It gives the spatial coordinates of Three-dimensional plane

<br>


><center> ZonePosition </center>
| Property | Datatype | What it infers| 
|--|---|-----|
x | double | Spatial coordinate X
y | double | Spatial coordinate Y
z | double | Spatial coordinate Z

<u>MapsListDelegate.</u>

```swift
func didReceiveAllMaps(_ maps: [MistMap]!) -> Void.
```
1. This delegate will give us maps for all sites in the entire organization. It gives us the __array of maps__ which is supposed to be of __MistMap__ type (properties were explained in detail – didUpdate(_map) delegate explanation.).
2. And it is a mandatory method to be overridden, if the class wants to conform this __MapsListDelegate__ protocol.

<u>Example:</u>

_Swift_:
```swift 
    func didReceiveAllMaps(_ maps: [MistMap]!) { 
        self.mapList = maps
    }
```
_Obj-c_:
```objc
    (void)didReceiveAllMaps:(NSArray<MistMap *> *)maps{ 
        _mapList = maps;
    }
```

<u>VirtualBeaconsDelegate.</u>
As we already know, vBeacon can provide proximity related information and the good thing is you can move these vBeacon wherever we want to place them in your floorplan and never have to worry about any battery or relocation problems. And it will work as exactly same as physical beacon.

<u>Required methods to be overridden.</u>

1. didRangeVirtualBeacon
    ```swift
    func didRangeVirtualBeacon(_ mistVirtualBeacon: MistVirtualBeacon!) -> Void
    ```
    Using this delegate method, we can get the current ranging beacon, which is near to us.

<u>Example:</u>

_Swift_:
```swift 
    func didRangeVirtualBeacon(_ mistVirtualBeacon: MistVirtualBeacon!) { 
        self. rangingvBeacon = mistVirtualBeacon
    }
```
_Obj-c_:
```objc
    (void)didRangeVirtualBeacon:(MistVirtualBeacon *)mistVirtualBeacon { _rangingvBeacon = mistVirtualBeacon;
    }
```


<u>Optional methods can be overridden on the need basis.</u>

1.  didUpdateVirtualBeaconList

    It will return all the available virtual beacons placed on the floorplan whenever its position and other details are changed from the Mist portal.

    ```swift
    func didUpdateVirtualBeaconList(_ mistVirtualBeacons: [MistVirtualBeacon]!) -> Void
    ```


<u>Example:</u>

_Swift_:
```swift 
    func didUpdateVirtualBeaconList(_ mistVirtualBeacons: [MistVirtualBeacon]!) { 
        self.vBeaconList = mistVirtualBeacons
    }
```
_Obj-c_:
```objc
    (void)didUpdateVirtualBeaconList:(NSArray<MistVirtualBeacon *> *)mistVirtualBeacons{ 
        _vBeaconList = mistVirtualBeacons;
    }
```


It returns the object/ array of Objects of the type __MistVirtualBeacon.__

<u>Properties of MistVirtualBeacon</u>

><center> MistVirtualBeacon </center>
| Property | Datatype | What it infers| 
|--|---|-----|
name | String | Name of the Virtual beacon
orgID | String | Organization Identifier
siteID | String | Site/ Venue id
mapID | String | Floorplan id
vbID | String | Unique Identifier of the Virtual beacon on Mist portal
vbUUID | long | vBeacon UUID
vbMajor | String | Major value of vBeaconsession of the current device inside the zone.
vbMinor | long | Minor value of vBeacon
position | BeaconPosition | It Mentions where exactly the vBeacon is positioned in floor plan.
additionalInf | AdditionalInfo | Additional information on the vBeacons
message | String | Any custom messages entered on mist portal.it will be Used for Notification purpose.
power | long | It’s the power of the vBeacon, to increase the sensing strength. So that the device can sense the vBeacon in a specified distance.


<br>

><center> BeaconPosition </center>
| Property | Datatype | What it infers| 
|--|---|-----|
x | double | Spatial coordinate X of vBeacon.
y | double | Spatial coordinate Y of vBeacon.
z | double | Spatial coordinate Z of vBeacon.

<br>

><center> AdditionalInfo </center>
| Property | Datatype | What it infers| 
|--|---|-----|
| proximity | String | Proximity of the device with vBeacon, like:<br>1. immediate <br>2. near<br>3. far<br>4. unknown  
| distance | double | It tells how much distance a device from the vBeacon.
rssi | double | Signal strength
orientation | long | Orientation of device towards the vBeacon
direction | String | Direction towards the vBeacon


<br>


7. And finally, we simply call the stop function in the indoor Location manager to stop the SDK.
    a. Swift: IndoorLocationManager?.stop ()
    b. Obj-c:[_indoorLocationManager stop];




<br>

<br>


# Methods Available in IndoorLocationManager
1. -(void) startWithIndoorLocationDelegate: (id <IndoorLocationDelegate>) delegate;
2. -(void) stop;
3. -(NSArray*) getWayfindingPathTo:(Position*) destination;
4. +(NSUUID *)getMistUUID;
5. +(void)clearMistUUID;
6. -(void)saveClientInformation:(NSString *)clientName withDelegate:
(id<ClientInformationDelegate>)clientInformationDelegate;
7. -(void)getClientInformationwithDelegate:(id<ClientInformationDelegate>)clientInformationDelegate;

## 1 - StartWithIndoorLocationDelegate

<ol type="a">
    <li> This method will start the MistSDK.</li>
    <li>This is the instance method, means that it should only be invoked via object of the class</li> 
    <li> It will take an indoorLocationDelegate as a parameter. So that we can be able to override the callbacks defined in indoorLocationDelegate protocol. </li> 
    <li> And it returns void.</li>
    <li>Usage:</li>
</ol>
<br> 


- Swift:
    ```swift
    let indoorLocationManager = IndoorLocationManager.sharedInstance(MistConstants.mistAccessToken) indoorLocationManager?.start (with: self)
    ```
- Obj-C:
    ```obj-c
    self.indoorLocationManager = [IndoorLocationManager sharedInstance: @"<AccessToken>"];
    [_indoorLocationManager startWithIndoorLocationDelegate:self];
    ```

## 2 - Stop

<ol type="a">
    <li>This method will stop MistSDK.</li>
    <li>This is also the instance method, so invocation should be via instance of class only.</li> 
    <li> It doesn’t take any arguments and won’t return anything. </li> 
    <li>Usage:</li>
</ol>
<br> 


- Swift:
    ```swift
    indoorLocationManager?.stop()
    ```
- Obj-C:
    ```obj-c
    [_indoorLocationManager stop];
    ```

## 3 - getWayfindingPathTo.

<ol type="a">
    <li>This method will return a list of nodes to be traversed when we want to draw the path towards the destination.</li>
    <li>This is also the instance method, so invocation should be via instance of class only.</li> 
    <li>We should pass the destination - (an object of type Position) as an argument.</li> 
    <li>It will return a sequence of nodes to be traversed in an array format of type <b>[Node]</b>.</li>
    <li>Usage:</li>
</ol>
<br> 


- Swift:
    ```swift
    let nodes = indoorLocationManager?.getWayfindingPath(to: Position.init(makeX: 10, y: 10))
    ```
- Obj-C:
    ```obj-c
    NSArray *nodes = [_indoorLocationManager getWayfindingPathTo:[Position positionMakeX:10 Y: 10]];
    ```

## 4 - GetMistUUID.

<ol type="a">
    <li><b>MistUUID</b>is the unique user id provided to the user who is in the session actively.</li>
    <li>This method will give us the current Mist User UUID from the server.</li> 
    <li>This is a class method, so invocation should be via Name of the class. It's like invoking a static method with a reference of a class Name.</li> 
    <li>It doesn’t take any arguments. <b>[Node]</b>.</li>
    <li>It returns the unique identifier of type <b>NSUUID</b> to the caller function.
    <li>Usage:</li>
</ol>
<br> 


- Swift:
    ```swift
    let userUniqueID = IndoorLocationManager.getMistUUID()
    ```
- Obj-C:
    ```obj-c
    NSUUID *userUniqueID = [IndoorLocationManager getMistUUID];
    ```

## 5 - clearMistUUID.

<ol type="a">
    <li>If the user wants to remove the user reference id for some reason, they can use <b>clearMistUUI()</b> method.</li>
    <li>It will clear the current session of the user from backend. And the user can call <b>getMistUUID()</b>again if he wishes to create and get a new id for the current user.</li> 
    <li>It doesn’t take any arguments.</li> 
    <li>Usage:</li>
</ol>
<br> 


- Swift:
    ```swift
    IndoorLocationManager.clearMistUUID()
    ```
- Obj-C:
    ```obj-c
    [IndoorLocationManager clearMistUUID];
    ```

## 6 - saveClientInformation.

<ol type="a">
    <li>This method saves the client information to the mist server - (client is the current user in the session).</li>
    <li>The user can send his name to server via this method, SDK will invoke the API update the username to server. After this process we can easily find the user in the map via this name.</li> 
    <li>It takes the username and the <b>ClientInformationDelegate</b> as a parameter. And it doesn’t return anything. Since this is an asynchronous function, we deal with delegate pattern to get a response from this method.</li> 
    <li>Usage:</li>
</ol>
<br> 


- Swift:
    ```swift
    indoorLocationManager?.saveClientInformation(" <username> ", with: self)
    ```
- Obj-C:
    ```obj-c
    [_indoorLocationManager saveClientInformation:"<username>" withDelegate:self]
    ```
<ol type="a" start="6">
<li> And the result will be returned in the call backs of ClientInformationDelegate.</li>
</ol>

## 7 - getClientInformationwithDelegate.

<ol type="a">
    <li>It fetches the client information from the server and returns to the application in the form of <b>ClientInformationDelegate</b></li>
    <li>Usage:</li>
</ol>
<br> 

- Swift:
    ```swift
    indoorLocationManager?.getClientInformationwithDelegate(self)
    ```
- Obj-C:
    ```obj-c
    [self.indoorLocationManager getClientInformationwithDelegate:self];
    ```

## ClientInformationDelegate.

1. -(void) onSuccess:(NSString*) clientName;
2. -(void) onError:(ErrorType )errorType andMessage:(NSString*) errorMessage;

<u>OnSuccess</u>
<br>

1. It will be triggered upon the success of the Updating process.
2. It will give us the updated clientName. 

- Swift:
    ```swift
    func onSuccess(_ clientName: String!) { 
        print(clientName ?? "")
    }
    ```
- Obj-C:
    ```obj-c
     -(void)onSuccess:(NSString *)clientName { 
        NSLog(@"client name %@", clientName);
    }
    ```

<u>OnError</u>
<br>

1. This will be triggered if there is a failure in the updating process.
2. and it will help us by giving two values
    - errorMessage of type __NSString__,
    - errorType of type __ErrorType__ Enum.
    - (For error type, please go through the list of errors mentioned in the table at the __didErrorOccur__ section in IndoorLocationDelegate.)

- Swift:
    ```swift
    func onError(_ errorType: ErrorType, andMessage errorMessage: String!) { 
        //handleErrorCases, for more go through didErrorOccur section in IndoorLocationDelegate
    }   
    ```
- Obj-C:
    ```obj-c
    -(void)onError:(ErrorType)errorType andMessage:(NSString *)errorMessage { 
        //handleErrorCases, for more go through didErrorOccur section in IndoorLocationDelegate
    }
    ```






<br> 


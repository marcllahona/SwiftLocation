![SwiftLocation](https://raw.githubusercontent.com/malcommac/SwiftLocation/master/logo.png)


SwiftLocation
=============
####Easy Location Services and Beacon Monitoring for Swift

[![CocoaPods Compatible](https://img.shields.io/cocoapods/v/SwiftLocation.svg)](https://img.shields.io/cocoapods/v/SwiftLocation.svg)
[![Carthage Compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)
[![Platform](https://img.shields.io/cocoapods/p/SwiftLocation.svg?style=flat)](http://cocoadocs.org/docsets/SwiftLocation)
[![Twitter](https://img.shields.io/badge/twitter-@danielemargutti-blue.svg?style=flat)](http://twitter.com/danielemargutti)

SwiftLocation is a lightweight library you can use to monitor locations, make reverse geocoding (both with Apple and Google's services) monitor beacons and do beacon advertising.
It's really easy to use and it's compatible both with Swift 2.2, 2.3 and 3.0.

Pick the right version:

- Official **Swift 2.2** is in master (and develop)
- **Swift 2.3** branch is [here](https://github.com/malcommac/SwiftLocation/tree/feature/swift2.3).
- **Swift 3.0** branch is [here](https://github.com/malcommac/SwiftLocation/tree/feature/swift3).
- Old unsupported **Swift 2.0** branch is [here](https://github.com/malcommac/SwiftLocation/tree/swift-2.0)

Main features includes:

- **Auto Management of hardware resources**: SwiftLocation turns off hardware if not used by our observers. Don't worry, we take care of your user's battery usage!
- **Complete location monitoring:** you can easily monitor for your desired accuracy and frequency (continous monitoring, background monitoring, monitor by distance intervals, interesting places or significant locations).
- **Device's heading observer**: you can observe or get current device's heading easily
- **Reverse geocoding** (from address string/coordinates to placemark) using both Apple and Google services (with support for API key)
- **GPS-less location fetching** using network IP address
- **Geographic region** monitoring (enter/exit from regions)
- **Beacon Family and Beacon** monitoring
- **Set a device to act like a Beacon** (only in foreground)

###Pre-requisites

Before using SwiftLocation you must configure your project to use location services. First of all you need to specify a value for ```NSLocationAlwaysUsageDescription``` or ```NSLocationWhenInUseUsageDescription``` into your application's Info.plist file. The string value you add will be shown along with the authorization request the first time your app will try to use location services hardware.

If you need background monitoring you should specify ```NSLocationAlwaysUsageDescription``` and specify the correct value in ```UIBackgroundModes``` key (you can learn more [here](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html))

###SwiftLocation in your next big project? Tell it to me!

I'm collecting all the apps which uses SwiftLocation to manage beacon or location. If you are using SwiftLocation in your project please fill a PR to this file or send an email to hello@danielemargutti.com.

From SwiftLocation 0.x to 1.0
-------
Several changes are made from 0.x branch to 1.0 especially from the side of the location manager.
It's pretty easy to align your project with this news version.
Since 1.0 we will keep the API stable and any change will use @available metatag of Swift to keep you in track.

Changes are:

#### Renamed Methods
- ```LocationManager.shared.``` is now replaced by ```Location.```
- ```BeaconManager.shared.``` is now replaced by ```Beacon.```
- Each request is conform to ```Request``` protocol. Where allowed you can use ```start()```, ```pause()``` or ```cancel()``` a running request.
- ```observeLocations()``` is now replaced with ```getLocation()``` (and it allows you to specify a custom timeout)
- ```observeInterestingPlaces()``` is now replaced with ```getInterestingPlaces()```
- Reverse geocoding services are now under the ```reverse``` function umbrella (```reverse(location:...), reverse(address:... and reverse(coordinates:...)```)

#### Other Changes
- ```Accuracy``` now include IP Address Scan (```.IPScan```) to get the current location (```locateByIPAddress()``` was removed). It works as usual, without asking sensor authorization to the user.
- ```observeHeading()``` is now replaced with ```getHeading```. Heading services now works correctly and allow you to specify a frequency (```HeadingFrequency```: ```.Continous(interval)``` to receive new heading at specified time intervals; ```.TrueNorth(minDegree)``` and ```.MagneticNorth(minDegree)``` allows you to receive events only when a specified deviation from the last catched heading is reported).
- ```HeadingRequest``` has now a ```allowsCalibration``` property instead of a ```onCalibrationRequired()``` function.
- ```onSuccess``` handler in ```HeadingRequest``` is now ```onReceiveUpdates```

Documentation
-------

* **Monitor Current User Location (one shout, continous delivery etc.)**
* **Obtain Current Location without GPS**
* **Monitor Device Heading**
* **Reverse Address/Coordinates to CLPlacemark**
* **Monitor Interesting Visits**
* **Monitor Geographic Regions**
* **Monitor Beacons & Beacon Families**
* **Act like a Beacon**

## Monitor Current User Location (one shot, continous delivery etc.)

Getting current user's location is pretty easy; all location related services are provided by the ```LocationManager``` singleton class.

```swift
Location.getLocation(withAccuracy: .Block, onSuccess: { foundLocation in
	// Your desidered location is here
}) { (lastValidLocation, error) in
	// something bad has occurred
	// - error contains the error occurred
	// - lastValidLocation is the last found location (if any) regardless specified accuracy level			
}
```

When you create a new observer you will get a request object ```LocationRequest``` you can use to change on the fly the current observer configuration or stop it.

This is an example:

```swift
Location.getLocation(withAccuracy: .Block, frequency: .OneShot, timeout: 50, onSuccess: { (location) in
	// You will receive at max one event if desidered accuracy can be achieved; this because you have set .OneShot as frequency.
}) { (lastValidLocation, error) in
}
// Sometimes in the future
request.stop() // Stop receiving updates
request.pause() // Temporary pause events
request.start() // Restart a paused request
```

```LocationRequest``` also specify a ```timeout``` property you can set to abort the request itself if no valid data is received in a certain amount of time. By default it's set to ```30 seconds``` (you can change directly at init time in ```getLocation()``` functions or by changing the ```.timeout``` property of the request itself)

```Location.getLocation()``` lets you to specify two parameters: the ```accuracy``` you need to get and the ```frequency``` intervals you want to use to get updated locations.

**ACCURACY**:

* ```IPScan```: (Network connection is required). Get an approximate location by using device's IP addres. It does not require GPS sensor or user authorizations.
* ```Any```: First available location is accepted, no matter the accuracy
* ```Country```: Only locations accurate to the nearest 100 kilometers are dispatched
* ```City```: Only locations accurate to the nearest three kilometers are dispatched
* ```Neighborhood```: Only locations accurate to the nearest kilometer are dispatched
* ```Block```: Only locations accurate to the nearest one hundred meters are dispatched
* ```House```: Only locations accurate to the nearest ten meters are dispatched
* ```Room```: Use the highest-level of accuracy, may use high energy
* ```Navigation```: Use the highest possible accuracy and combine it with additional sensor data



**FREQUENCY:**

* ```Continuous```: receive each new valid location, never stop (you must stop it manually)
* ```OneShot```: the first valid location data is received, then the request will be invalidated
* ```ByDistanceintervals(meters)```: receive a new update each time a new distance interval is travelled. Useful to keep battery usage low
* ```Significant```: receive only valid significant location updates. This capability provides tremendous power savings for apps that want to track a user’s approximate location and do not need highly accurate position information



## Obtain Current Location without GPS

Sometimes you could need to get the current approximate location and you may not need to turn on GPS hardware and waste user's battery. When accuracy is not required you can locate the user by it's public network IP address (obviously this require an internet connection).

```swift
Location.getLocation(withAccuracy: .IPScan, onSuccess: { (location) in
	// approximate location is here
}) { (lastValidLocation, error) in
	// something wrong has occurred; error will tell you what
}
```

## Monitor Device Heading

You can get data about current device's heading using observeHeading() function.

```swift
Location.getHeading(HeadingFrequency.Continuous(interval: 5), accuracy: 1.5, allowsCalibration: true, didUpdate: { newHeading in
	// each changes of at least 1.5 degree and 5 seconds after the last measurement is reported here
}) { error in
	// something bad occurred
}
```

## All the other features related to location manager

- SwiftLocation works even in background; just use ```Location.allowsBackgroundEvents = true``` (don't forget to define UIBackgroundModes array).
- SwiftLocation can optimize power usage by reducing location updates via ```Location.pausesLocationUpdatesWhenPossible = true```
- SwiftLocation can defer location updates: use it to optimize power usage when you want location data with GPS accuracy but do not need to process that data right away; you can get the complete location points after certain time or distance (use ```Location..stopDeferredLocationUpdates()``` to stop deferred updates)
- Shortcuts to pause or resume requests: ```Location.startAllLocationRequests() ``` (start pending requests), ```Location.stopAllLocationRequests``` (pause or stop all requests), ```Location.stopSignificantLocationRequests()``` (pause or stop only significant requests)
- Use ```Location.minimumDistance = ...``` to define the minimum distance (measured in meters) a device must move horizontally before an update event is generated (by default is ignored, all events are generated)


## Reverse Address Strings or Coordinates

You can do a reverse geocoding from a given pair of coordinates or a readable address string. You can use both Google or Apple's services to perform these request.

Swift location provides three different methods:

* ```reverse(address:using:onSuccess:onError)```: allows you to get a ```CLPlacemark``` object from a source address string. It require ```service``` (```.Apple``` or ```.Google```) and an ```address``` string.
* ```reverse(coordinates:using:onSuccess:onError:)```: allows you to get a ```CLPlacemark``` object from a source coordinates expressed as ```CLLocationCoordinate2D```.
* ```reverse(location:using:onSuccess:onError:)``` the same of the previous method but accept a ```CLLocation``` as source object.

Some examples:

```swift
let addString = "1 Infinite Loop, Cupertino"
Location.reverse(address: addString, onSuccess: { foundPlacemark in
	// foundPlacemark is a CLPlacemark object
}) { error in
	// failed to reverse geocoding due to an error
}
```
```swift
let coordinates = CLLocationCoordinate2DMake(41.890198, 12.492204)
Location.reverse(coordinates: coordinates, onSuccess: { foundPlacemark in
	// foundPlacemark is a CLPlacemark object
}) { error in
	// failed to reverse geocoding due to an error
}
```

## Monitor Interesting Places

```CoreLocation``` allows you to get notified when user visits an interesting place by returning a ```CLVisit``` object: it encapsulates information about interesting places that the user has been. Visit objects are created by the system. The visit includes the location where the visit occurred and information about the arrival and departure times as relevant. You do not create visit objects directly, nor should you subclass ```CLVisit```.

You can add a new handler to get notification about visits via ```getInterestingPlaces(onDidVisit:)``` function.

```swift
Location.getInterestingPlaces { newVisit in
	// a new CLVisit object is returned
}
```

## Monitor Geographic Regions
You can easily to be notified when the user crosses a region based boundary.
You use region monitoring to detect boundary crossings of the specified region and you use those boundary crossings to perform related tasks. For example, upon approaching a dry cleaners, an app could notify the user to pick up any clothes that had been dropped off and are now ready.

SwiftLocation offers to you a simple method called ```monitor()``` to get notified about these kind of events: 

```swift
// Define a geographic circle region
let centerPoint = CLLocationCoordinate2DMake(0, 0)
let radius = CLLocationDistance(100)
do {
   // Attempt to monitor the region
	let request = try Beacons.monitor(geographicRegion: centerPoint, radius: radius, onStateDidChange: { newState in
		// newState is .Entered if user entered into the region defined by the center point and the radius or .Exited if it move away from the region.
	}) { error in
		// something bad has happened
	}
} catch let err {
	// Failed to initialize region (bad region, monitor is not supported by the hardware etc.)
	print("Cannot monitor region due to an error: \(err)")
}
```

Usually you can ```pause()```/```start()``` or ```cancel()``` the request itself; just keep a reference to it.

## Monitor Beacons

You can easily to be notified when the user crosses a region defined by a beacon or get notified when users did found one or more beacons nearby.

Just use ```monitor()``` function:

```swift
let b_proximity = "00194D5B-0A08-4697-B81C-C9BDE117412E"
// You can omit major and minor to get notified about the entire beacon family defined by b_proximity
let b_major = CLBeaconMajorValue(64224)
let b_minor = CLBeaconMinorValue(43514)
		
do {
	// Just create a Beacon structure which represent our beacon
	let beacon = Beacon(proximity: proximity, major: major, minor: minor)
	// Attempt to monitor beacon
	try Beacons.monitor(beacon: beacon, events: Event.RegionBoundary, onStateDidChange: { state in
		// events called when user crosses the boundary the region defined by passed beacon
	}, onRangingBeacons: { visibleBeacons in
		// events is fired countinously to get the list of visible beacons (with observed beacon) nearby
	}, onError: { error in
		// something went wrong. request is cancelled.
	})
} catch let err {
	// failed to monitor beacon
}
```

## Act like a Beacon

You can set your device to act like a beacon (this feature works only in foreground due to some limitations of Apple's own methods).

Keep in mind: advertising not works in background.

```swift
let request = Beacons.advertise(beaconName: "name", UUID: proximity, major: major, minor: minor, powerRSSI: 4, serviceUUIDs: [])
```

Use ```stop()``` on ```request``` to stop beacon advertise.

Changes
-------

### Version 1.0.3 (2016/08/26):
- [FIX #61](https://github.com/malcommac/SwiftLocation/issues/61) Fixed several issues with location services ```start()``` func.

### Version 1.0.2 (2016/08/26):
- [FIX #60](https://github.com/malcommac/SwiftLocation/issues/60) A new rewritten Beacon Monitoring service class
- Added support for deferred location updates
- Better handle not supported hardware errors
- Fixed an issue which does not reset or start unwanted timeout timer in location services
- Better support to ```pause(),start()``` and ```cancel()``` requests

### Version 1.0.1 (2016/08/22):
- [FIX #48](https://github.com/malcommac/SwiftLocation/issues/48) Fixed an issue which blocks the library to report correct values for ```.Room``` and ```.Navigation``` accuracy levels
- [FIX #55](https://github.com/malcommac/SwiftLocation/issues/55) Fixed an issue with timeout timer of location manager which is called even when the location was found correctly.
- [NEW] ```CLLocationAuthorizationStatus``` is now conform to CustomStringConvertible protocol.

### Version 1.0.0 (2016/08/15):
- First stable version


Installation
-------

This library require iOS 8+ and Swift 2.2.

## CocoaPods
[CocoaPods](<http://cocoapods.org>) is a dependency manager for Cocoa projects. You can install it with the following command:

```
$ gem install cocoapods
```

CocoaPods 0.39.0+ is required to build SwiftLocation 1.0.0+.
To integrate SwiftLocation into your Xcode project using CocoaPods, specify it in your Podfile:

```
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '9.0'
use_frameworks!

target '<Your Target Name>' do
    pod 'SwiftLocation', '~> 1.0'
end
```

Then, run the following command:

```
$ pod install
```
## Carthage

[Carthage](<https://github.com/Carthage/Carthage>) is a decentralized dependency manager that builds your dependencies and provides you with binary frameworks.

You can install Carthage with [Homebrew](<http://brew.sh/>) using the following command:

```
$ brew update
$ brew install carthage
```

To integrate SwiftLocation into your Xcode project using Carthage, specify it in your Cartfile:

```
github "malcommac/SwiftLocation" ~> 1.0
```

Run ```carthage update``` to build the framework and drag the built ```SwiftLocation.framework``` into your Xcode project.

Author & License
-------

SwiftLocation was created and mantained by Daniele Margutti.

- Email: [hello@danielemargutti.com](<mailto:hello@danielemargutti.com>)
- Website: [danielemargutti.com](<http://www.danielemargutti.com>)
- Twitter: [@danielemargutti](<http://www.twitter.com/danielemargutti>)

Your app here!
-------
While SwiftLocation is free to use and change (I'm happy to discuss any PR with you) if you plan to use it in your project please consider to add:

```"Location Services provided by SwiftLocation by Daniele Margutti"```

and a link to this GitHub page.

Drop me an email so I'll add your creation here!

SwiftLocation is available under the MIT license.

See the LICENSE file for more info.

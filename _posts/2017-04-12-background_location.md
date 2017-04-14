---
title: iOS location
description: I'm develpoing a littile app where background location was used.So I put basic knowledgement and some confusion in there.
header: iOS location
---

Happiness takes no account of time.

locationManager property

1. delegate //necessary
2. activityType //why you need location
3. desiredAccuracy //more higher, more power used
4. pausesLocationUpdatesAutomatically //meaning is it's name
5. allowsBackgroundLocationUpdates //must set if you want background location
6. distanceFilter //minimum distance to update in meters

Method:
1. requestAlwaysAuthorization() //Allow background location
2. requestWhenInUseAuthorization() //also can run in back if you set allowsBackgroundLocationUpdates true, when alow location in background, there is a blue header in top of screen when we press home button.
3. requestLocation() //didUpdateLocations callback only one time
4. startUpdatingLocation() //
5. startMonitoringVisits() //
6. startMonitoringSignificantLocationChanges() //
7. allowDeferredLocationUpdates() //

callback:
1. didUpdateLocations
2. didFailWithError
3. didFinishDeferredUpdatesWithError
4. didVisit
5. didChangeAuthorization
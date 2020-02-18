# HEALTHKIT PLUGIN

Capacitor plugin to retrieve data from HealthKit

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

Add HealthKit to your Xcode project (section signing & capabilities)
ADD Privacy - Health Records Usage Description
ADD Privacy - Health Share Usage Description to your Xcode project
ADD Privacy - Health Update Usage Description to your Xcode project


### Installing

```
npm i --save capacitor-healthkit
```

Then

```
npx cap update
```


## Methods

### isAvailable()

Tells if HealthKit are available.

```
isAvailable(successCallback, errorCallback)
```
* successCallback: {type: function(available)}, if available a true is passed as argument, false otherwise
* errorCallback: {type: function(err)}, called if something went wrong, err contains a textual description of the problem

### requestAuthorization()

Requests read and write access to a set of data types. It is recommendable to always explain why the app needs access to the data before asking the user to authorize it.

Important: this method must be called before using the query and store methods, even if the authorization has already been given at some point in the past. Failure to do so may cause your app to crash.

```
requestAuthorization(datatypes, successCallback, errorCallback)
```

* datatypes: {type: Mixed array}, a list of data types you want to be granted access to. You can also specify read or write only permissions.
  {
    all : [‘’]                    // Read & Write permission
       read : ['steps'],                   // Read only permission
        write : ['height', 'weight']        // Write only permission
  }

Example :
let result = await CapacitorHealthkit.requestAuthorization({
      all: ["calories", "stairs", "activity"], // ask for Read & Write permission
      read: ["steps", "distance", "duration"], // ask for Read Only permission
      write: [""] // ask for Write Only permission
    });


* successCallback: {type: function}, called if all OK
* errorCallback: {type: function(err)}, called if something went wrong, err contains a textual description of the problem

The datatype activity also includes sleep. If you want to get authorization only for workouts, you can specify workouts as datatype, but be aware that this is only availabe in iOS.


Data type for requestAuthorization
| Data type | Unit | HealthKit equivalent |
| --- | --- | --- |
| steps | count | HKQuantityTypeIdentifierStepCount |
| stairs | count| HKQuantityTypeIdentifierFlightsClimbed |
| distance | m | HKQuantityTypeIdentifierDistanceWalkingRunning + HKQuantityTypeIdentifierDistanceCycling |
| appleExerciseTime | min | HKQuantityTypeIdentifierAppleExerciseTime |
| calories | kcal | HKQuantityTypeIdentifierActiveEnergyBurned + HKQuantityTypeIdentifierBasalEnergyBurned |
| activity | activityType | HKWorkoutTypeIdentifier + HKCategoryTypeIdentifierSleepAnalysis |


### queryHKitSampleType()

Gets all the data points of a certain data type within a certain time window.

Warning: if the time span is big, it can generate long arrays!

```
queryHKitSampleType(queryOptions, successCallback, errorCallback)
```

queryOption example :
const endDate = new Date();
{
      _sampleName: ’stepCount’, // String
      _startDate: '2019/07/01', // String
      _endDate: endDate, // Date
      _limit: 0 // Int
}

Sample name available for queries
| Sample name for query | Request Auth Needed  | HealthKit equivalent |
| --- | --- | --- |
| stepCount | steps | HKQuantityTypeIdentifierStepCount |
| flightsClimbed | stairs | HKQuantityTypeIdentifierFlightsClimbed |
| distanceWalkingRunning | distance | HKQuantityTypeIdentifierDistanceWalkingRunning |
| distanceCycling | distance | HKQuantityTypeIdentifierDistanceCycling |
| appleExerciseTime | appleExerciseTime | HKQuantityTypeIdentifierAppleExerciseTime |
| activeEnergyBurned | calories | HKQuantityTypeIdentifierActiveEnergyBurned |
| basalEnergyBurned | calories | HKQuantityTypeIdentifierBasalEnergyBurned |

| sleepAnalysis | activity | HKCategoryTypeIdentifierSleepAnalysis |
| workoutType | activity | HKWorkoutTypeIdentifier |


EXAMPLE FUNCTION in Angular :
```
async queryHKitSampleType(sampleName: string) {
    // sample name, start date (string), end Date (date), limit (0 to infinite)
    // let start = "2019/07/01" // YY/MM/DD
    this.dataName = sampleName;
    const endDate = new Date();
    this.data = await CapacitorHealthkit.queryHKitSampleType({
      _sampleName: sampleName,
      _startDate: '2019/07/01',
      _endDate: endDate,
      _limit: 0
    });
  }
```


* startDate: {type: Date}, start date from which to get data
* endDate: {type: Date}, end data to which to get the data
* dataType: {type: String}, the data type to be queried (see above)
* limit: {type: integer}, optional, sets a maximum number of returned values
* successCallback: {type: function(data) }, called if all OK, data contains the result of the query in the form of an array of: { startDate: Date, endDate: Date, value: xxx, unit: 'xxx', sourceName: 'aaaa', sourceBundleId: 'bbbb' }
* errorCallback: {type: function(err)}, called if something went wrong, err contains a textual description of the problem



iOS quirks
* Limit is set to unlimited by default (if you insert 0)
* Datapoints are ordered in an descending fashion (from newer to older). You can revert this behaviour by adding ascending: true to your query object.
* HealthKit does not calculate active and basal calories - these must be inputted from an app
* HealthKit does not detect specific activities - these must be inputted from an app
* When querying for activities, only events whose startDate and endDate are both in the query range will be returned.

RETURN DATA ::
```
{
    ‘’countReturn’’: result.count, // number of results
    ‘resultDate’: output // output data in result of query
}
```

Returned objects (output) contain a set of fixed fields:
* startDate: {type: Date} a date indicating when the data point starts
* endDate: {type: Date} a date indicating when the data point ends
* sourceBundleId: {type: String} the identifier of the app that produced the data
* sourceName: {type: String} the name of the app that produced the data (as it appears to the user)
* unit: {type: String} the unit of measurement
* value: the actual value
* uuid: (string) the unique identifier of that measurement


And output :
If quantity type output contains :
```
- uuid (string)
- value (double)
- unitName (string)
- startDate (ISO8601 String)
- endDate (ISO8601 String)
- duration (double ou INT jsais pas)
- source (string)
- sourceBundleId (string)
```

If Workout type output contains :
```
- uuid (string)
- startDate (ISO8601 String)
- endDate (ISO8601 String)
- duration (double ou INT jsais pas)
- source (string)
- sourceBundleId (string)
- "workoutActivityId (Int)
- "totalEnergyBurned (kilocalorie)
- "totalDistance (meter)
- "totalFlightsClimbed (count)
- "totalSwimmingStrokeCount (count)
```
If data = -1 => no data collected
 
If Sleep type output contains :
```
- uuid (string)
- startDate (ISO8601 String)
- endDate (ISO8601 String)
- duration (double ou INT jsais pas)
- source (string)
- sourceBundleId (string)
```

## Built With

* Capacitor
* VSCode
* XCode

## Contributing

Theo Creach for Ad Scientiam

## Versioning

Version 0.0.2

## Authors

* **Theo Creach** - *Developer* - [Twitter](https://twitter.com/crcht)

## License

This project is licensed under the MIT License


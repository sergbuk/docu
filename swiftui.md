Question 1: 
Will be value for a field added in version 2 moved/synced on a device which has app version 1 and later will be updated to version 2?

[Mirroring a Core Data Store with CloudKit](https://developer.apple.com/documentation/coredata/mirroring_a_core_data_store_with_cloudkit)

[Deploy the Development Schema to Production](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/CloudKitQuickStart/DeployingYourCloudKitApp/DeployingYourCloudKitApp.html#//apple_ref/doc/uid/TP40014987-CH10)

[Synchronizing a Local Store to the Cloud (WWDC Sample App Download Link)](https://developer.apple.com/documentation/coredata/synchronizing_a_local_store_to_the_cloud)

[SwiftUI by Example (1000)](https://www.hackingwithswift.com/quick-start/swiftui)

[Core Data and SwiftUI - Saving, retrieving, updating and deleting persistent data](https://www.blckbirds.com/post/core-data-and-swiftui)
[code sample](https://github.com/BLCKBIRDS/Core-Data-in-SwiftUI---Pizza-Restaurant-App)

Swift Coredata Cheatsheets:

https://learn.co/lessons/swift-coredata-cheatsheet
https://www.andrewcbancroft.com/2015/02/18/core-data-cheat-sheet-for-swift-ios-developers/

[Core Data в деталях](https://habr.com/ru/post/436510/)

[Мультиконтекстность в Core Data](https://habr.com/ru/post/238901/)

[How To Observe A Managed Object Context](https://cocoacasts.com/how-to-observe-a-managed-object-context)
source: https://github.com/bartjacobs/ObservingManagedObjectContext

[Filter store transactions for changes relevant to the current view.](https://developer.apple.com/documentation/coredata/consuming_relevant_store_changes)

[Core Data and Swift: Core Data Stack](https://code.tutsplus.com/tutorials/core-data-and-swift-core-data-stack--cms-25065)

[Core Data and Aggregate Fetches In Swift](http://www.cimgf.com/2015/06/25/core-data-and-aggregate-fetches-in-swift/)

[Group by, Count and Sum in CoreData](https://www.cocoanetics.com/2017/04/group-by-count-and-sum-in-coredata/)

Book: "Core data in swift" by Marcus Zarra pdf

[Syncing a Core Data Store with CloudKit](https://developer.apple.com/documentation/coredata/mirroring_a_core_data_store_with_cloudkit/syncing_a_core_data_store_with_cloudkit)

[Consuming Relevant Store Changes](https://developer.apple.com/documentation/coredata/consuming_relevant_store_changes)

(also NSManagedObjectContextDidSaveNotification vs NSPersistentStoreRemoteChangeNotification)

Toggle sync with NSPersistentCloudKitContainer (see post Nov 3, 2019 1:34 PM):

https://forums.developer.apple.com/thread/118924
https://stackoverflow.com/questions/58179862/cloudkit-sync-using-nspersistentcloudkitcontainer-in-ios13

[Predicate Format String Syntax](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Predicates/Articles/pSyntax.html)

[Observing Core Data Changes with a Custom Combine Publisher](https://www.mattmoriarity.com/observing-core-data-changes-with-combine/custom-publisher/)

[Persistent History Tracking in Core Data](https://mjtsai.com/blog/2019/08/21/persistent-history-tracking-in-core-data/)

[Using Combine (BOOK)](https://heckj.github.io/swiftui-notes/)

Code Sample:
```swift
NotificationCenter.default.addObserver(
    self,
    selector: #selector(fetchChanges),
    name: NSNotification.Name(
        rawValue: "NSPersistentStoreRemoteChangeNotification"), 
        object: persistentContainer.persistentStoreCoordinator
)
```

Code Sample (NSExpressionDescription, source https://stackoverflow.com/questions/27344756/how-to-sum-a-double-attribute-from-coredata-in-swift):
But there is a better way: You create an "expression description" for the sum of all totalWorkTimeInHours values:

```swift
let expressionDesc = NSExpressionDescription()
expressionDesc.name = "sumOftotalWorkTimeInHours"
expressionDesc.expression = NSExpression(forFunction: "sum:",
         arguments:[NSExpression(forKeyPath: "totalWorkTimeInHours")])
expressionDesc.expressionResultType = .DoubleAttributeType
```
and then a fetch request which fetches only this sum:
```swift
let fetchRequest = NSFetchRequest(entityName: "Log")
fetchRequest.propertiesToFetch = [expressionDesc]
fetchRequest.resultType = .DictionaryResultType

var error : NSError?
if let results  = managedContext.executeFetchRequest(fetchRequest, error: &error) {
    let dict = results[0] as [String:Double]
    let totalHoursWorkedSum = dict["sumOftotalWorkTimeInHours"]!
    println(totalHoursWorkedSum)
} else {
    println("fetch failed: \(error?.localizedDescription)")
}
```
The advantage is that the sum is calculated on the SQLite level, you don't have to fetch all the objects into memory.

Code Sample (NSCompoundPredicate):
```swift
        let predicateIsNumber = NSPredicate(format: "isStringOrNumber == %@", NSNumber(value: false))
        let predicateIsEnabled = NSPredicate(format: "isEnabled == %@", NSNumber(value: true))
        let andPredicate = NSCompoundPredicate(type: .and, subpredicates: [predicateIsNumber, predicateIsEnabled])

        //check here for the sender of the message
        let fetchRequestSender = NSFetchRequest<NSFetchRequestResult>(entityName: "Keyword")
        fetchRequestSender.predicate = andPredicate
```

Code Sample (How to translate this SQL Query into a NSFetchRequest):
source: https://stackoverflow.com/questions/26809587/how-to-translate-this-sql-query-into-a-nsfetchrequest
select max(ZSENTAT), * from ZMESSAGE where ZISINCOMING == 1 group by ZROOTMESSAGEID;

```objective-c
NSExpression *maxExpression = [NSExpression expressionWithFormat:@"max:(sentAt)"];
NSExpressionDescription *maxDesc = [[NSExpressionDescription alloc] init];
maxDesc.name = @"maxSentAt";
maxDesc.expression = maxExpression;
maxDesc.expressionResultType = NSDateAttributeType; // I assume sentAt is a NSDate
request.propertiesToFetch = @[@"rootMessageId", maxDesc];
request.propertiesToGroupBy = @[@"rootMessageId"];
request.resultType = NSDictionaryResultType;
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"isIncoming == %@",[NSNumber numberWithBool:YES]];
request.predicate = predicate;
NSArray *results = [self.managedObjectContext executeFetchRequest:request error:&error];
etc
```

Code Sample (How to use Combine to assign the number of elements returned from a Core Data fetch request?)
source: https://stackoverflow.com/questions/57766270/how-to-use-combine-to-assign-the-number-of-elements-returned-from-a-core-data-fe

```swift
extension NotificationCenter.Publisher {
    func context<T>(fetchRequest: NSFetchRequest<T>) -> Publishers.CompactMap<NotificationCenter.Publisher, [T]> {
        return compactMap { notification -> [T]? in
            let context = notification.object as! NSManagedObjectContext
            var results: [T]?
            context.performAndWait {
                results = try? context.fetch(fetchRequest)
            }
            return results
        }
    }
}

let playFetchRequest: NSFetchRequest<ReplayRecord> = ReplayRecord.fetchRequest()
let replayVideoFetchRequest: NSFetchRequest<ReplayVideo> = ReplayVideo.fetchRequest()

let playsPublisher = contextDidSavePublisher.context(fetchRequest: playFetchRequest).map(\.count)
let replayVideoPublisher = contextDidSavePublisher.context(fetchRequest: replayVideoFetchRequest).map(\.count)

playsSubscription = playsPublisher.zip(replayVideoPublisher).map {
    String(format: Constants.Strings.playsText, $0, $1)

}.receive(on: RunLoop.main).assign(to: \.text, on: self.playsLabel)
```


[Observing Core Data Changes with a Custom Combine Publisher](https://www.mattmoriarity.com/observing-core-data-changes-with-combine/custom-publisher/)

code sample [How to fix slow List updates in SwiftUI"](https://www.hackingwithswift.com/articles/210/how-to-fix-slow-list-updates-in-swiftui)

```swift
List(items, id: \.self) {
    Text("Item \($0)")
}
.id(UUID())
```

TODO: Display all environment values:
struct ShowLineLimit: View {
  
  @Environment(\.lineLimit) var limit
  
  var body: some View {
    Text(verbatim: "The line limit is: \(limit)")
  }
}


[Environment Values](https://developer.apple.com/documentation/swiftui/environmentvalues)


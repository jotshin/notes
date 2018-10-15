### Identify the problem and start debugging
When you run into Core Data threading crash bug, it will look like this

<img src="https://oleb.net/media/xcode-core-data-multithreading-violation.png" width="50%">

The debugger halts at `+[NSManagedObjectContext __Multithreading_Violation_AllThatIsLeftToUsIsHonor__]:`.

Turn on the core data concurrency debugging flag, otherwise you won't run into this when debugging everytime.
`-com.apple.CoreData.ConcurrencyDebug 1`

<img src="https://oleb.net/media/xcode-scheme-core-data-concurrency-debug.png" width="50%">

### Understand Core Data threading
In Apple [official document](https://developer.apple.com/documentation/coredata/using_core_data_in_the_background?language=objc):

> To use Core Data in a multithreaded environment, ensure that:
> - Managed object contexts are bound to the thread (queue) that they are associated with upon initialization
> - Managed objects retrieved from a context are bound to the same queue that the context is bound to

Also, to avoid threading problem for Core Data:
> - **In general, avoid doing data processing on the main queue that is not user-related.** 
Data processing can be CPU-intensive, and if it is performed on the main queue, it can result in unresponsiveness in the user interface. If your application will be processing data, like importing data into Core Data from JSON, create a private queue context and perform the import on the private context.
> - **Do not pass NSManagedObject instances between queues.** 
Doing so can result in corruption of the data and termination of the app. When it is necessary to hand off a managed object reference from one queue to another, use NSManagedObjectID instances.

### The solution
Now we know how the problem is caused - the object's threading is different from the the current one, which violates Core Data’s threading contract. In addition, we need to avoid performing the polling operation on the main queue, which isn't user-related.

As a result, We need to access the Core Data objects in the background queue by asking a managed object context for the managed object that corresponds with objectID, and according to Apple:

> You retrieve the managed object ID of a managed object by calling the objectID accessor on the NSManagedObject instance.

### Here's the check list for how the problem can be solved:
- [ ] Verify which queue your current operation is in
- [ ] Verify which `NSManagedObjectContext` your object is fetched from
- [ ] If above two aren't consistent, try to use objectID to fetch the object in the correct context

References:
[Core Data Concurrency Debugging](https://oleb.net/blog/2014/06/core-data-concurrency-debugging/)
[Core Data from Scratch: Concurrency](https://code.tutsplus.com/tutorials/core-data-from-scratch-concurrency--cms-22131)

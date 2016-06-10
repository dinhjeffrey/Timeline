# Timeline

### Level 3

Timeline is a simple photo sharing service. Students will bring in many concepts that they have learned, and add more complex data modeling, Image Picker, CloudKit, and protocol-oriented programming to make a Capstone Level project spanning multiple days and concepts.

Most concepts will be covered during class, others are introduced during the project. Not every instruction will outline each line of code to write, but lead the student to the solution. 

Students who complete this project independently are able to:

#### Part One - Project Planning, Model Objects, and Controllers

* follow a project planning framework to build a development plan
* follow a project planning framework to prioritize and manage project progress
* implement relationships in a Core Data model architecture
* use staged data to prototype features

#### Part Two - Apple View Controllers, Search Controller, Container Views

* implement search using the system search controller
* use the image picker controller and activity controller
* use container views to abstract shared functionality into a single view controller

#### Part Three - Basic CloudKit: CloudKitManager, CloudKitManagedObject, Manual Sync

* check CloudKit availability
* save data to CloudKit
* fetch data from CloudKit
* sync pulled CloudKit data to a local Core Data persistent store

#### Part Four - Intermediate CloudKit: Subscriptions, Push Notifications, Automatic Sync

* query data from CloudKit
* use subscriptions to generate push notifications
* use push notifications to run a push based sync engine


## Part One - Project Planning, Model Objects, and Controllers

* follow a project planning framework to build a development plan
* follow a project planning framework to prioritize and manage project progress
* implement relationships in a Core Data model architecture
* use staged data to prototype features

Follow the development plan included with the project to build out the basic view hierarchy, basic implementation of local model objects, model object controllers, and helper classes. Build staged data to lay a strong foundation for the rest of the app.

### View Hierarchy

Implement the view hierarchy in Storyboards. The app will have a Timeline tableview that will also use a Search Controller to display search results. Both the Timeline view and the Search Results view will display a list of `Post` objects and segue to a `Post` detail view.

The Navigation Controller should have a Plus (+) button that presents a modal Add Post scene that will allow the user to select a photo, add a caption, and submit the photo.

1. Add a `UITableViewController` Timeline scene, embed it in a `UINavigationController`, add a Plus (+) button as the right bar button. 
2. Add a `PostListTableViewController` subclass of `UITableViewController` and assign it to the Timeline scene
3. Add a `UITableViewController` Post Detail scene, add a segue to it from the Timeline scene
4. Add a `PostDetailTableViewController` subclass of `UITableViewController` and assign it to the Post Detail scene
5. Add a `UITableViewController` Add Post scene, embed it into a `UINavigationController`, and add a modal presentation segue to it from the Plus (+) button on the Timeline scene
    * note: Because this scene will use a modal presentation, it will not inherit the `UINavigationBar` from the Timeline scene
6. Add a `AddPostTableViewController` subclass of `UITableViewcontroller` and assign it to the Add Post scene.
7. Add a `UITableViewcontroller` Search Results scene. It does not need a relationship to any other view controller.
    * note: You will implement this scene in Part 2 when setting up the `UISearchController` on the Search scene
8. Add a `SearchResultsTableViewController` subclass of `UITableViewController` and assign it to the Search Results scene.

### Implement Model

Timeline will use a Core Data local persistent storage with a basic CloudKit based sync engine. To begin, add Core Data to the project by creating a Core Data Model xcdatamodel file, and adding the `Stack` file that will be used to access Core Data. Then use the Core Data Model file to create your local Core Data model objects.

You will want to save `Post` objects that hold the image data, and `Comment` objects that hold text. A `Post` should own a set of `Comment` objects.

While you will not implement sync in this portion of the project, it is important to recognize the need for syncing when designing your model.

#### SyncableObject

Create a `SyncableObject` model object that will hold shared data for both `Post` and `Comment` objects. `Post` and `Comment` objects will need to share a set of properties that make it easier for syncing in a future step. The shared properties will include a timestamp, a unique identifier, and additional data that will help determine if the object has been synced to the server.

1. Add a new `SyncableObject` Core Data entity to your managed object model. Add a `timestamp` Date attribute, a `recordName` String attribute, and a `recordIDData` NSData attribute. Set the `recordName` and `timestamp` attributes as required.
2. Select the `recordName` and `timestamp` attributes and deselect the 'Optional' option in the Attribute Inspector panel.

#### Post

Create a `Post` model object that will hold image data. Most often, Core Data is backed by a SQLite persistent store. SQLite databases are not built for managing large data blobs like photos. Use the 'Allows external storage' option to have Core Data manage storing the data to disk instead of the database itself. Core Data will automatically store the data to disk and include a reference to that data in the database instead.

1. Add a new `Post` Core Data entity to your managed object model. Add a `photoData` Binary Data attribute. 
2. Set the `SyncableObject` as the Parent Entity. 
    * note: This will make the `Post` entity a subclass of the `SyncableObject` class, inheriting all of it's properties.
3. Select the `photoData` attribute and choose the 'Allows External Storage' option in the Attribute Inspector panel.
4. Create the `NSManagedObject` subclass files.
5. Add a convenience initializer that accepts a `photo` parameter as `NSData`, and a timestamp parameter as an `NSDate`.
6. Implement the convenience initializer, calling the `init(entity: insertIntoManagedObjectContext:)` function that normally is called by the `NSEntityDescription.entityForName(name:, inManagedObjectContext:)` function.

#### Comment

Create a `Comment` model object that will hold user-submitted text comments for a specific `Post`. 

1. Add a new `Comment` Core Data entity to your managed object model. Add a `text` String attribute.
2. Set the `SyncableObject` as the Parent Entity. 
    * note: This will make the `Comment` entity a subclass of the `SyncableObject` class, inheriting all of it's properties.
3. Add a relationship that supports adding multiple `Comment` object relationships to a single `Post` object, set the inverse relationship on the `Post` object.
4. Create the `NSManagedObject` subclass files.
5. Add a convenience initializer that accepts a `Post` parameter, a text parameter, and a timestamp parameter as an `NSDate`.
    * note: Include the `Post` parameter because a `Comment` cannot exist without a parent `Post`. This is a common pattern for doing [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection). 
6. Implement the convenience initializer, calling the `init(entity: insertIntoManagedObjectContext:)` function that normally is called by the `NSEntityDescription.entityForName(name:, inManagedObjectContext:) function.


### Model Object Controller

Add and implement the `PostController` class that will be used for CRUD operations. 

1. Add a new `PostController` class file.
2. Add a `sharedController` singleton property.
3. Add a `saveContext` function that saves the `Stack`'s `managedObjectContext`.
4. Add a `createPost` function that takes an image parameter as a `UIImage` and a caption as a `String`.
5. Implement the `createPost` function to initialize a `Post` with the image and a `Comment` with the caption text and save the context.
6. Add a `addCommentToPost` function that takes a `text` parameter as a `String`, and a `Post` parameter.
7. Implement the `addCommentToPost` function to call the appropriate `Comment` initializer and save the context.


### Wire Up Views

#### Timeline Scene - Post List Table View Controller

Implement the Post List Table View Controller. You will use a similar cell to display posts in multiple scenes in your application. Create a custom `PostTableViewCell` that can be reused in different scenes.

1. Implement the scene in Interface Builder by creating a custom cell with an image view that fills the cell. 
2. Create a `PostTableViewCell` class, add and implement an `updateWithPost` to the `PostTableViewCell` to update the image view with the `Post`'s photo.
3. Choose a height that will be used for your image cells. To avoid worrying about resizing images or dynamic cell heights, you may want to use a consistent height for all of the image views in the app.
4. Overwrite the `viewDidLoad()` function to initialize an `NSFetchedResultsController` that pulls `Post` objects ordered by the `timestamp` property in reverse chronological order (latest post on top).
5. Implement the `UITableViewDataSource` functions using the `NSFetchedResultsController`.
    * note: The final app does not need to support any editing styles, but you may want to include support for editing while developing early stages of the app.
6. Implement the `NSFetchedResultsControllerDelegate` functions to begin and end table view updates.
7. Implement the `prepareForSegue` function to check the segue identifier, capture the detail view controller, index path, selected post, and assign the selected post to the detail view controller.
    * note: You may need to quickly add a `post` property to the `PostDetailTableViewController`.

#### Post Detail Scene

Implement the Post Detail View Controller. This scene will be used for viewing post images and comments. Users will also have the option to add a comment, share the image, or follow the user that created the post.

Use the table view's header view to display the photo and a toolbar that allows the user to comment, share, or follow. Use the table view cells to display comments.

1. Add a vertical `UIStackView` to the Header of the table view. Add a `UIImageView` and a `UIToolbar` to the stack view. Add 'Comment', 'Share', and 'Follow Post' `UIBarButtonItem`s to the toolbar. Set up your constraints so that the image view is the height you chose previously for displaying images within your app.
2. Update the cell to support comments that span multiple lines without truncating them. Set the `UITableViewCell` to the subtitle style. Set the number of lines to zero. Implement dynamic heights by setting the `tableView.rowHeight` and `tableView.estimatedRowHeight` in the `viewDidLoad`.
3. Add an `updateWithPost` function that will update the scene with the details of the post. Implement the function by setting the `imageView.image` and reloading the table view if needed.
4. Overwrite the `viewDidLoad()` function to initialize an `NSFetchedResultsController` that pulls `Comment` objects matching the `self.post` property, ordered by the `timestamp` property in chronological order (latest comment on bottom).
    * note: You will need to set a predicate on the `NSFetchRequest` (hint: `"post == $@"`)
5. Implement the `UITableViewDataSource` functions using the `NSFetchedResultsController`.
    * note: The final app does not need to support any editing styles, but you may want to include support for editing while developing early stages of the app.
5. Add an IBAction for the 'Comment' button. Implement the IBAction by presenting a `UIAlertController` with a text field, a Cancel action, and an 'OK' action. Implement the 'OK' action to initialize a new `Comment` via the `PostController` and reload the table view to display it.
    * note: Do not create a new `Comment` if the user has not added text.
6. Add an IBAction for the 'Share' and 'Follow' buttons. You will implement these two actions in future steps.

#### Add Post Scene

Implement the Add Post Table View Controller. You will use a static table view to create a simple form for adding a new post. Use three sections for the form:

Section 1: Large button to select an image, and a `UIImageView` to display the selected image
Section 2: Caption text field
Section 3: Add Post button

Until you implement the `UIImagePickerController`, you will use a staged static image to add new posts.

1. Assign the table view to use static cells. Adopt the 'Grouped' cell style. Add three sections.
2. Build the first section by creating a tall image selection/preview cell. Add a 'Select Image' `UIButton` that fills the cell. Add an empty `UIImageView` that also fills the cell. Make sure that the button is on top of the image view so it can properly recognize tap events.
3. Build the second section by adding a `UITextField` that fills the cell. Assign placeholder text so the user recognizes what the text field is for.
4. Build the third section by adding a 'Add Post' `UIButton` that fills the cell. 
5. Add an IBAction to the 'Select Image' `UIButton` that assigns a static image to the image view (add a sample image to the Assets.xcassets that you can use for prototyping this feature), and removes the title text from the button.
    * note: It is important to remove the title text so that the user no longer sees that a button is there, but do not remove the entire button, that way the user can tap again to select a different image.
6. Add an IBAction to the 'Add Post' `UIButton` that checks for an `image` and `caption`. If there is an `image` and a `caption`, use the `PostController` to create a new `Post` and dismiss the view controller. If either the image or a caption is missing, present an alert directing the user to check their information and try again.
7. Add a 'Cancel' `UIBarButtonItem` as the left bar button item. Implement the IBAction to dismiss the view.

#### A Note on Reusable Code

Consider that this Photo Selection functionality could be useful in different views and in different applications. New developers will be tempted to copy and paste the functionality wherever it is needed. That amount of repetition should give you pause. `Don't repeat yourself` (DRY) is a shared value among skilled software developers.

Avoiding repetition is an important way to become a better developer and maintain sanity when building larger applications.

Imagine a scenario where you have three classes with similar functionality. Each time you fix a bug or add a feature to any of those classes, you must go and repeat that in all three places. This commonly leads to differences, which leads to bugs.

You will refactor the Photo Selection functionality (selecting and assigning an image) into a reusable child view controller in Part 2. 

### Polish Rough Edges

At this point you should be able view added post images in the Timeline Post List scene, add new `Post` objects from the Add Post Scene, add new `Comment` objects from the Post Detail Scene, and persist and use user profile information provided by the current user. 

Use the app and polish any rough edges. Check table view cell selection. Check text fields. Check proper view hierarchy and navigation models.

### Black Diamonds

* Review the README instructions and solution code for clarity and functionality, submit a GitHub pull request with suggested changes.
* Provide feedback on the expectations for Part One to a mentor or instructor.

## Part Two - Search Controller, Container Views, Apple View Controllers

* implement search using the system search controller
* use the image picker controller and activity controller
* use container views to abstract shared functionality into a single view controller

Add and implement search functionality to the search view. Implement the Image Picker Controller on the Account Setup scene and Add Post scene. Decrease the amount of repeated code by refactoring the similar functionality in the Account Setup and Add Post scenes into a child view controller that is used in both classes.

### Search Controller

Build functionality that will allow the user to search for posts with comments that have specific text in them. For example, if a user creates a `Post` with a photo of a waterfall, and there are comments that mention the waterfall, the user should be able to search the Timeline view for the term 'water' and filter down to that post (and any others with water in the comments).

#### Update the Model

Add a `SearchableRecord` protocol that requires a `matchesSearchTerm` function. Update the `Post` and `Comment` objects to conform to the protocol.

1. Add a new `SearchableRecord.swift` file.
2. Define a `SearchableRecord` protocol with a required `matchesSearchTerm` function that takes a `searchTerm` parameter as a `String` and returns a `Bool`.
    * note: Because this protocol will be used on `NSManagedObject`s, add the `@objc` keyword to the protocol.

Consider how each model object will match to a specific search term. What searchable text is there on a `Comment`? What searchable text is there on a `Post`?

3. Update the `Comment` class to conform to the `SearchableRecord` protocol. Return `true` if `text` contains the search term, otherwise return `false`.
4. Update the `Post` class to conform to the `SearchableRecord` protocol. Return `true` if any of the `Post` `comments` match, otherwise return `false`.

Use a Playground to test your `SearchableRecord` and `matchesSearchTerm` functionality and understand what you are implementing.

#### Search Results Controller

Search controllers typically have two views: a list view, and a search result view that displays the filtered results. The list view holds the search bar. When the user begins typing in the search bar, the `UISSearchController` presents a search results view. Your list view must conform to the `SearchResultsUpdating` protocol function, which implements updates to the results view.

Understanding Search Controllers requires you to understand that the main view controller can (and must) implement methods that handle what is being displayed on another view controller. The results controller must also implement a way to communicate back to the main list view controller to notify it of events. This is a two way relationship with communication happening in both directions.

1. Create a `SearchResultsTableViewController` subclass of `UITableViewController` and assign it to the scene in Interface Builder.
2. Add a `resultsArray` property that contains a list of `SearchableRecords`
3. Implement the `UITableViewDatasource` functions to display the search results.   
    * note: For now you will only display `Post` objects as a result of a search. Use the `PostTableViewCell` to do so.

#### Update Timeline Scene

1. Add a function `setUpSearchController` that captures the `resultsController` from the Storyboard, instantiates the `UISearchController`, sets the `searchResultsUpdater` to self, and adds the `searchController`'s `searchBar` as the table's header view.
2. Implement the `UISearchResultsUpdating` protocol `updateSearchResultsforSearchController` function. The function should capture the `resultsViewController` and the search text from the `searchController`'s `searchBar`, filter the local `posts` array for posts that match, assign the filtered results to the `resultsViewController`'s `resultsArray`, and reload the `resultsViewController`'s `tableView`.
    * note: Consider the communication that is happening here between two separate view controllers. Be sure that you understand this relationship.

##### Segue to Post Detail View

Remember that even though the Timeline view and the Search Results view are displaying similar cells and model objects, you are working with separate view controllers with separate cells and instances of table views. 

The segue from a `Post` should take the user to the Post Detail scene, regardless of whether that is from the Timeline view or the Search Results view.

To do so, implement the `UITableViewDelegate` `didSelectRow` function on the Search Results scene to manually call the `toPostDetail` segue _from the Search scene_.

1. Adopt the `UITableViewDelegate` on the Search Results scene and add the `didSelectRowAtIndexPath` function. Implement the function by capturing the sending cell and telling the Search Result scene's `presentingViewController` to `performSegueWithIdentifier` and send the selected cell so that the Search scene can get the selected `Post`.
    * note: Every view controller class has an optional `presentingViewController` reference to the view controller that presented it. In this case, the presenting view controller of the Search Results scene is the Timeline scene. So this step will manually call the `performSegueWithIdentifier` on the Search scene.
2. Update the `prepareForSegue` function on the Search Scene to capture and segue to the Post Detail scene with the correct post. Try to do so without looking at the solution code.
    * note: You must check if the `tableView` can get an `indexPath` for the sender. If it can, that means that the cell was from the Search scene's `tableView`. If it can't, that means the cell is from the Search Result scene's `tableView` and that the user tapped a search result. If that is the case, capture the `Post` from the `resultsArray` on the `searchResultscontroller`.
    * note: You can access the `searchResultsController` by calling `(searchController.searchResultsController as? SearchResultsTableViewController)`

Try to work through the Search segue without looking at the solution code. Understanding this pattern will solidify your understanding of many object-oriented programming patterns.


### Image Picker Controller

#### Add Post Scene

Implement the Image Picker Controller in place of the prototype functionality you built previously.

1. Update the 'Select Image' IBAction to present a `UIImagePickerController`. Give the user the option to select from their Photo Library or from the device's camera if their device has one. 
2. Implement the `UIImagePickerControllerDelegate` function to capture the selected image and assign it to the image view.

### Reduce Code Repetition

Refactor the photo selection functionality from the Account Setup and Add Post scenes into a child view controller. 

Child view controllers control views that are a subview of another view controller. It is a great way to encapsulate functionality into one class that can be reused in multiple places. This is a great tool for any time you want a similar view to be present in multiple places.

In this instance, you will put 'Select Photo' button, the image view, and the code that presents and handles the `UIImagePickerController` into a `PhotoSelectorViewController` class. You will also define a protocol for the `PhotoSelectorViewController` class to communicate with it's parent view controller.

#### Container View and Embed Segues

Use a container view to embed a child view controller into the Account Setup scene and Add Post scene.

>Container View defines a region within a view controller's view subgraph that can include a child view controller. Create an embed segue from the container view to the child view controller in the storyboard.

1. Open `Main.storyboard` to your Account Setup scene.
2. Add a new section to the static table view to build the Container View to embed the child view controller.
3. Search for Container View in the Object Library and add it to the newly created table view cell.
    * note: The Container View object will come with a view controller scene. You can use the included scene, or replace it with another scene. For now, use the included scene.
4. Set up contraints so that the Container View fills the entire cell.
5. Move or copy the Image View and 'Select Photo' button to the container view controller.
6. Create a new `PhotoSelectViewController` file as a subclass of `UIViewController` and assign the class to the scene in Interface Builder.
7. Create the necessary IBOutlets and IBActions, and migrate your Photo Picker code from the Account Setup view controller class. Delete the old code from the Account Setup view controller class.
8. Repeat the above steps for the Add Post scene. Instead of keeping the included child view controller from the Container View object, delete it, and add an 'Embed' segue from the container view to the scene you set up for the Account Setup scene.

You now have two views that reference the same scene as a child view controller. This scene and accompanying class can now be used in both places, eliminating the need for code duplication.

#### Child View Controller Delegate

Your child view controller needs a way to communicate events to it's parent view controller. This is most commonly done through delegation. Define a child view controller delegate, adopt it in the parent view controller, and set up the relationship via the embed segue.

1. Define a new `PhotoSelectViewControllerDelegate` protocol in the `PhotoSelectViewController` file with a required `photoSelectViewControllerSelectedImage` function that takes a `UIImage` parameter to pass the image that was selected.
    * note: This function will tell the assigned delegate (the parent view controller, in this example) what image the user selected.
2. Add a weak optional delegate property.
3. Call the delegate function in the `didFinishPickingMediaWithInfo` function, passing the selected media to the delegate.
4. Adopt the `PhotoSelectViewControllerDelegate` protocol in the Account Setup class file, implement the `photoSelectViewControllerSelectedImage` function to capture a reference to the selected image.
    * note: In the Account Setup scene, you will use that captured reference to update the user.
5. Adopt the `PhotoSelectViewControllerDelegate` protocol in the Add Post class file, implement the `photoSelectViewControllerSelectedImage` function to capture a reference to the selected image.
    * note: In the Add Post scene, you will use that captured reference to create a new post.

Note the use of the delegate pattern. You have encapsulated the Photo Selection workflow in one class, but by implementing the delegate pattern,  each parent view controller can implement it's own response to when a photo was selected. 

You have declared a protocol, adopted the protocol, but you now must assign the delegate property on the instance of the child view controller so that the `PhotoSelectViewController` can communicate with it's parent view controller. This is done by using the embed segue, which is called when the Container View is initialized from the Storyboard, which occurs when the view loads.

1. Assign segue identifiers to the embed segues in the Storyboard file
2. Update the `prepareForSegue` function in the Account Setup scene to check for the segue identifier, capture the `destinationViewController` as a `PhotoSelectViewController`, and assign `self` as the child view controller's delegate.

### Post Detail View Controller Share Sheet

Use the `UIActivityController` class to present a share sheet from the Post Detail view. Share the image and the text of the first comment.

1. Add an IBAction from the Share button in your `PostDetailTableViewController`.
2. Initialize a `UIActivityController` with the `Post`'s image and the text of the first comment as the shareable objects.
3. Present the `UIActivityController`.

### Black Diamonds:

* Some apps will save photos taken or processed in their app in a custom Album in the user's Camera Roll. Add this feature.
* Review the README instructions and solution code for clarity and functionality, submit a GitHub pull request with suggested changes.
* Provide feedback on the expectations for Part One to a mentor or instructor.


## Part Three - Basic CloudKit: CloudKitManager, CloudKitManagedObject, Manual Sync

* check CloudKit availability
* save data to CloudKit
* fetch data from CloudKit
* sync pulled CloudKit data to a local Core Data persistent store

Following some of the best practices in the documentation, add CloudKit to your project as a backend syncing engine for photos. Check for CloudKit availability, save new posts and comments to CloudKit, fetch posts and comments from CloudKit and save them to Core Data.

At this stage you are simply syncing photos, posts, and comments from the device to CloudKit, and pulling new photos, posts, and comments from CloudKit. You will implement user discoverability and search in a future part of the project.

### CloudKit Manager

Add a CloudKit Manager that abstracts your CloudKit code into a single helper class, and fulfills the basic required CloudKit functions. 

You will add more CloudKit functionality to the `CloudKitManager` in future steps.

1. Add a `CloudKitManager` helper class.
2. Add function signatures that perform basic CloudKit functionality. 

``swift

    internal func fetchRecordsWithType(type: String, completion: ((records: [CKRecord]?, error: NSError?) -> Void)?)

    internal func fetchRecordWithID(recordID: CKRecordID, completion: ((record: CKRecord?, error: NSError?) -> Void)?)

    internal func fetchRecentRecords(recordType: String, fromDate: NSDate, toDate: NSDate, completion: ((records: [CKRecord]?, error: NSError?) -> Void)?)

    internal func deleteRecordWithID(recordID: CKRecordID, completion: ((recordID: CKRecordID?, error: NSError?) -> Void)?)

    internal func deleteRecordsWithID(recordIDs: [CKRecordID], completion: ((records: [CKRecord]?, recordIDs: [CKRecordID]?, error: NSError?) -> Void)?)

    internal func saveRecord(record: CKRecord, completion: ((record: CKRecord?, error: NSError?) -> Void)?)

    internal func modifyRecord(record: CKRecord, completion: ((record: CKRecord?, error: NSError?) -> Void)?)

    internal func handleCloudKitUnavailable(accountStatus: CKAccountStatus, error: NSError?)

    internal func displayCloudKitNotAvailableError(errorText: String)
```

3. Using the documentation for CloudKit, fulfill the contract of each function signature. Using the data passed in as a paremeter, write code that will return the requested information. When it makes sense to do so using the NSOperation subclasses, try to use them over the convenience functions.

### CloudKitManagedObject

Write a protocol that will define how the app will work with CloudKit and Core Data objects. Add a protocol extension that adds predefined convenience functionality to each object that adopts that protocol.

Our `CloudKitManagedObject` types will need to have a timestamp, a way to persist the data from a `CKRecord` object into the persistent store, a unique record name, a record type that matches a `CKRecord` type, and a way to represent the managed object as a `CKRecord` for when we want to push the data to CloudKit.

1. Create a new `CloudKitManagedObject` file that defines a new protocol named `CloudKitManagedObject`.
2. Add required gettable and settable variables for the `timestamp` as an NSDate, `recordData` as optional NSData.
    * `CKRecord` conforms to the `NSCoding` protocol, which allows it to be easily serialized to and from `NSData`. Core Data does not natively support saving `CKRecord` objects, but we can set the `NSData` of a `CKRecord` to the `recordData` to persist the `CKRecord`.
3. Add required gettable computed properties for the `recordType` as a String, and `cloudKitRecord` as an optional CKRecord.
4. Add a required `updateWithRecord` function that accepts a `CKRecord` as a parameter. This function will be used to update the Core Data object with `CKRecord` data received from CloudKit.


### Post Controller Manual Sync

#### Update Post for Sync Functionality

#### Update Comment for Sync Functionality

#### Update the Post Controller for Manual Sync
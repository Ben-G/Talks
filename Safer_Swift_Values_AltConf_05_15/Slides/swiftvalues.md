footer: Safer Swift Code with Value Types | @benjaminencz | AltConf, June 2015 
slidenumbers: true

# Safer Swift Code with Value Types

^
- My name is Benjamin Encz
- iOS Developer at Make School

---

#What do I mean by safety?

![inline](images/safety.png)

^
- Have you heard about Peter? Got fired from Medium for shipping buggy product
- Probably not true ;)
- This talks is not about Software Security
- It's about reducing the potential for programming error
- So it's essentially about "Safety" as in "Job Safety" ;)

---

## Agenda

1. **What** are Value Types vs. Reference Types
2. **Why** is this topic relevant now?
3. **How:** A Practical Example of a "Value Oriented" Architecture

^
- 3 Main Parts
- What: make sure we are on the same page
- Why now?
- How is the main point of the presentation! Listend great talk by Andy Matushak that inspired this talk
- Ideas of Value Orientation appealed to me, wanted to find practical application for my iOS code
 
---

#[fit] Values vs. References

---

# Reference Types

```swift

class PersonRefType {
  let name:String
  var age:Int
  
  // ..
}
// 1
let peter = PersonRefType(name: "Peter", age: 36)
// 2
let peter2 = peter
// 3
peter2.age = 25
// peter {"Peter", 25}
// peter2 {"Peter", 25}
```

^ 
- Defining very simple class - that's one of the reference types in Swift
- Create an instance assigned to a var
- Create a second var
- Modify the second var
- Age changed for both vars!

---
# Value Types

```swift

struct Person {
  let name:String
  var age:Int
}
// 1
let petra = Person(name:"Petra", age:25)
// 2
var petra2 = petra
// 3
petra2.age = 20
// petra  {"Petra", 25}
// petra2 {"Petra", 20}
```

^
- Define struct that looks almost the same
- Create an instance, assign it to var
- Assign it to a second var
- Change age
- Change is only reflected in first var
- "Copy on assignment" behavior

---


#[fit]Why now?

---

#Foundation / C Types

**Reference Types:**
- `NSArray`
- `NSSet`
- `NSData`

**Value Types:**
- `NSInteger`
- `Struct`

^
- Most types we work with in Obj-C are Reference Types
- Value types are limited to C types: Int, Bool, C structs

---

#Swift Standard Library

**Reference Types:**
- `ManagedBuffer`(?)
- `NonObjectiveCBase`(??)

**Value Types:**
- `Array`
- `String`
- `Optional`

^
- Swift Standard Library has shifted towards value types
- The only reference types are pretty esotheric (mostly useful to build your own data structures)
- Array, String, Optional, etc. are value types
- Indicator that we should make more use of value types in our code?
- Next: Why did this shift happen?

---

#Enums and Structs in Swift are Powerful

- Can have variables
- Can have functions
- Can implement protocols

^ 
- Swift promotes the use of enums/structs by giving them almost all the capabilities of classes

---
#So What Can't They Do? 

> “Indeed, in contrast to structs, Swift classes support  implementation inheritance, (limited) reflection,  deinitializers, and multiple owners.” - Andy Matushak (http://www.objc.io/issue-16/swift-classes-vs-structs.html)

^ 
- Inheritance and multiple ownership are the most important to us
- Maybe value types can mitigate these disadvantages?
- I will try to show: multiple owners is seldom desirable
- Code reuse can happen outside of inheritance
- Just because it's possible doesn't mean we should do it. What's the motivation? 

---

#We've already been doing this!

```objectivec
@property (copy) NSString *userName;
```

^
- This is a common pattern in Cocoa + Objective-C. We don’t want the value of a string that we are passing on to a view to be changed from outside. We don’t want multiple owners.
- We already were aware of the problem, now we have a solution

---

#[fit]Case Study
#[fit]A Twitter Client Built on Immutable Value Types

^
- Why **immutable** value types?
- Mutable value types are only useful in very rare cases!
- Mostly not, because you only update one copy, not any of the other copies around
(http://stackoverflow.com/questions/441309/why-are-mutable-structs-evil)

---
#Twitter Client

1. Download the latest 200 tweets and display them
2. Allow to filter tweets (RT only, favorited tweets only, etc.)
3. Allow user to favorite tweets (should be synced with server)

---

![inline 120%](images/OOP_Twitter.png)

^
- Objects encapsulate data and behavior
- E.g Tweet Object has a "favorite" method
- TwitterClient encapsulates most functionality: Auth, Parsing, API Requests -> Not uncommon to have a such a Client class in our apps
- Next let's take a look at a value oriented architecture

---

![inline 120%](images/fetchtweets.png)

^
- Let's leave the favoriting of tweets out of the model for now, it's a little more complex, we'll discuss in a second
- Classes/Structs in rectangles
- Circles are functions
- Simple Value and User structs
- ViewController and Tweet Cell
- Rest are functions that are composed together
- Instead of bulky *Client* class we have isolated functions with well defined inputs and outputs
- **Encourages reuse!** e.g. Image Download function
- I think we can agree that this is a fine architecture
- What about favoriting? Why is it complicated enough to leave out of this diagram?

---

#[fit]How can we favorite Tweets?

^
- We still need to discuss the most important part: how can we favorite tweets using immutable value types?
- And.. why is it so complicated that we needed to defer discussion until now?
- After all, it is really simple in OOP

---

#It Is Very Simple with OOP

```swift
tweet.favorited = true
```
^
- Simply modify the value and we're done

---

#It Is Simple with OOP

```swift
let lockQueue = dispatch_queue_create("com.happylocking", nil)
dispatch_sync(lockQueue) {
	tweet.favorited = true
}
```
^
- Except we need to lock to make sure that the same reference is not accessed from multiple threads at the same time

---

#Is It Simple with OOP?

```swift
let lockQueue = dispatch_queue_create("com.happylocking", nil)
dispatch_sync(lockQueue) {
	tweet.favorited = true
	NSNotificationCenter.defaultCenter().
		postNotificationName("Tweet Changed", object: tweet)
}
```
^
- And we need to propagate the updated info to parts of the app that don't have a reference to this tweet object
- We could have multiple tweet objects that represent the same tweet on the server, e.g. because we performed multiple API requests to retrieve them


---

#Modelling Change is Hard!

```swift
let lockQueue = dispatch_queue_create("com.happylocking", nil)
dispatch_sync(lockQueue) {
	tweet.favorited = true
	NSNotificationCenter.defaultCenter().
		postNotificationName("Tweet Changed", object: tweet)
	tweetAPIClient.markFavorited(tweet.identifier)
}
```
^
- Finally we need to know which data we need to update *outside* of our application and perform the necessary change there

---

#Modelling Change is Hard!

- Protect against unwanted updates
- Distribute new value throughout application
- Understand what the underlying *identity* of an object is and perform update accordingly

---

#Modelling Change is Hard!

- Protect against unwanted updates
- Distribute new value
- Understand what the underlying *identity* of an object is and perform update accordingly

-> **We need to this in all places where we mutate values!**

---
#[fit]Modelling Change
#[fit]is Hard!

---
#How Can We Model Change With Immutable Value Types?

**Model change to Values as Values**

- Create a new Tweet for every change
- Save these changes in a *Store*
- *Store* saves local changes and server state
- *Store* provides a merged view on list of tweets
- *Store* can trigger sync of local state to server

^
- Every change yields a new Tweet instance
- We keep local state and server state separated
- Store provides a view by combining server and local state
- The store can use local state list to trigger API requests that syncs that local state


---

![inline](images/MergingState.png)

^
- Merging is simple in this app
- Latest local Tweet is "truth", if we don't have local changes we use server tweet
- Merging could be much more sophisticated

---

#Favoriting a Tweet

```swift
  let currentTweet = tweetTableViewCell.tweet!

  let newTweet = Tweet(
    content: currentTweet.content,
    identifier: currentTweet.identifier,
    user: currentTweet.user,
    type: currentTweet.type,
    favoriteCount: currentTweet.favoriteCount,
    isFavorited: !currentTweet.isFavorited
  )
  
  store.addTweetChangeToLocalState(newTweet)
```

^
- Create a new Tweet and add it to the local state of the store
- What does the store look like?

---

#Modelling Change

```swift
class TweetStore {
 
  var tweets: [Tweet]? {
    get {
      var mergedList: [Tweet]? = nil

      dispatch_sync(storeQueue) {
        mergedList = mergeListIntoListLeftPriority(self.localState, self.serverTweets)
      }
      
      return mergedList
    }
  }
  
  func addTweetChangeToLocalState(tweet: Tweet) {
    dispatch_sync(storeQueue) {
      let index = find(self.localState, tweet)
      if let index = index {
        self.localState[index] = tweet
      } else {
        self.localState.append(tweet)
      }
    }
  }
  
  //...
```

^
- We lock here to get thread safety
- Two important pieces of functionality:
- tweets is computed property, merges local and server state (could also not be computed, many options here)
- Additional method allows caller to add tweet to change set. As simplification we only keep newest modified local tweet - also here could be more sophisticated and we could resolve conflicts between multiple local versions
- That is how we model change
- What does our complete architecture diagram look like?

---

![inline](images/ValueType_Twitter_Marked.png)

^
- Callers can pull data from the store, that data runs through the merge function
- Callers can add local changes to the store
- Callers can trigger store to fetch new data
- How do we sync changes?

---

#Syncing Change

1. Iterate over each local change
2. Generate API request that syncs that local change to server
3. Upon each API response:
	- If success: remove tweet from local change set
	- If failure: leave tweet in local change set
	
^
- Changes that couldn't be synced successfully stay in local change set and get synced again later

---

#Syncing Change


```swift
struct StateMerge <T> {
  let originalList: [T]
  let localState: [T]
}

enum SyncResult <T> {
  case Success(StateMerge<T>)
  case Error(StateMerge<T>)
}

protocol StoreSync {
  typealias StoreType
  
  static func syncLocalState(merge: StateMerge<StoreType>) 
  		-> Promise<SyncResult<StoreType>>
}
```
^
- No need to dive into detail of code
- Takes server list and local state
- Returns success/error and remainder of local list

---

#Benefits of "Value Oriented" Architecture

- Confidence that no one will change our data under the covers
- Change propagation needs to be handled explicitly
- No false sense of *identity*
- Modelling change as data opens opportunities:
	- Undo Functionality
	- Sophisticated conflict resolution
	- ... 	

---
#[fit]Thank you!

**Example project:** 
https://github.com/Ben-G/TwitterSwift

**Get in touch:** 
@benjaminencz

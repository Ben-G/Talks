footer: Safer Swift Code with Value Types | @benjaminencz | 360iDev, August 2015 
slidenumbers: true

#[fit]Safer Swift Code 
#[fit]with Value Types

^
- My name is Benjamin Encz
- iOS Developer at Make School
- What does Safety mean in context of my talk?
- Not Software Security!
- Broadly spoken 2 different goals with new techniques: Increase efficiency and reduce possible errors

---

#What do I mean by safety?

*Safe*, adjective [^*]:
- involving little or no risk of mishap, error, etc.
- dependable or trustworthy

[^*]: Thanks to Rich Hickey for pointing out this great way to open a talk


^
- TODO

---

#Should I use a struct or a class?

It depends! [^4] [^5] [^6] [^7] [^8] [^9]

[^4]: GitHub Styleguide: "Unless you require functionality that can only be provided by a class (like identity or deinitializers), implement a struct instead"

[^5]: Apple Developer Guide: "In practice, this means that most custom data constructs should be classes, not structures"

[^6]: Apple Swift Blog: "One of the primary reasons to choose value types over reference types is the ability to more easily reason about your code."

[^7]: http://faq.sealedabstract.com/structs_or_classes/: "when it comes time to actually save any of those models to disk, or push them over a network, or draw them to the screen, or whatever, you need a class. Classes everywhere. Not "immutable valuable types everywhere" like in the Struct Philosophy™"

[^8]: http://owensd.io/2015/07/05/re-struct-or-class.html

[^9]: https://www.mikeash.com/pyblog/friday-qa-2015-07-17-when-to-use-swift-structs-and-classes.html

---

# Agenda

1. **What** are Value Types vs. Reference Types
2. **Why** is this topic relevant now?
3. **How:** A Practical Example of a Value Oriented Architecture

^
- 3 Main Parts
- What: make sure we are on the same page
- Why now?
- How is the main point of the presentation! Listened great talk by Andy Matushak that inspired this talk
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
- The only reference types are pretty esoteric (mostly useful to build your own data structures)
- Array, String, Optional, etc. are value types
- Indicator that we should make more use of value types in our code?
- Next: Why did this shift happen?

---

#Enums and Structs in Swift are Powerful

- Can have properties
- Can have method
- Can conform to protocols

^ 
- Swift promotes the use of enums/structs by giving them almost all the capabilities of classes
- You might wonder, which capabilities are missing from value types that are available in reference types?

---
#So What Can't They Do? 

> “Indeed, in contrast to structs, Swift classes support  **implementation inheritance**, (limited) reflection,  deinitializers, and **multiple owners**.” 

Andy Matushak[^1]

[^1]:http://www.objc.io/issue-16/swift-classes-vs-structs.html

^ 
- Inheritance and multiple ownership are the most important to us
- Maybe value types can mitigate these disadvantages?
- I will try to show: multiple owners is seldom desirable
- Code reuse can happen outside of inheritance
- Just because it's possible doesn't mean we should do it. What's the motivation? 

---

#We've Already Been Doing This!

```objectivec
@property (copy) NSString *userName;
```

^
- This is a common pattern in Cocoa + Objective-C. We don’t want the value of a string that we are passing on to a view to be changed from outside. We don’t want multiple owners.
- We use `copy` attribute
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
#Twitter Clients

1. Download the latest 50 tweets and display them
2. Allow to filter tweets (RT only, favorited tweets only, etc.)
3. *Allow user to favorite tweets (should be synced with server)*

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

#Modeling Change is Hard!

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

#Modeling Change is Hard!

- Protect against unwanted updates
- Distribute new value throughout application
- Understand what the underlying *identity* of an object is and perform update accordingly

---

#Modeling Change is Hard!

- Protect against unwanted updates
- Distribute new value throughout application
- Understand what the underlying *identity* of an object is and perform update accordingly

-> **We need to this in all places where we mutate values!**

---

#Traditional Data Flow

![inline](images/InformationFlowOOP.png) 

---

![inline](images/InformationFlowOOP.png) 

*Safe*, adjective:
- involving little or no risk of mishap, error, etc.
- dependable or trustworthy

---

#[fit]Modelling Change
#[fit]is Hard!

---

#Should I use a struct or a class?

It depends! [^4] [^5] [^6] [^7] [^8] [^9]

[^4]: GitHub Styleguide: "Unless you require functionality that can only be provided by a class (like identity or deinitializers), implement a struct instead"

[^5]: Apple Developer Guide: "In practice, this means that most custom data constructs should be classes, not structures"

[^6]: Apple Swift Blog: "One of the primary reasons to choose value types over reference types is the ability to more easily reason about your code."

[^7]: http://faq.sealedabstract.com/structs_or_classes/: "when it comes time to actually save any of those models to disk, or push them over a network, or draw them to the screen, or whatever, you need a class. Classes everywhere. Not "immutable valuable types everywhere" like in the Struct Philosophy™"

[^8]: http://owensd.io/2015/07/05/re-struct-or-class.html

[^9]: https://www.mikeash.com/pyblog/friday-qa-2015-07-17-when-to-use-swift-structs-and-classes.html

---

#It's an architectural question

- Shared state vs. isolated state
- Mutable state vs. immutable state

[^10]: https://developer.apple.com/videos/wwdc/2014/

---

#How Can We Model Change With Immutable Value Types?

>

---

#How Can We Model Change With Immutable Value Types?

**Model change to values as values!**

---

#How Can We Model Change With Immutable Value Types?

**Model change to values as values:**

* Create a new Tweet for every change
* Save these mutations in a local change set
* Save the server state in a separate data set
* Provide a merged view on list of tweets
* Provide method to sync local state to server

^
- Every change yields a new Tweet instance
- We keep local state and server state separated
- Store provides a view by combining server and local state
- The store can use local state list to trigger API requests that syncs that local state

---

![inline 180%](images/MergingState.png)

^
- Merging is simple in this app
- Latest local Tweet is "truth", if we don't have local changes we use server tweet
- Merging could be much more sophisticated

---

#How is new truth distributed?

---

#How is new truth distributed?

Drawing inspiration from the JavaScript world. Unidirectional data flow with flux/redux [^10] [^11]:

**Unidirectional Data Flow**

[^10]: https://facebook.github.io/flux/

[^11]: https://github.com/rackt/redux

---

#Unidirectional data flow

![inline](images/InformationFlow.png)

---

![left 55%](images/InformationFlowNoCaption.png)

#Unidirectional data flow

- **ActionCreator**: Provides interface for business logic
- **Action**: Represents an individual state mutation
- **Dispatcher**: Stores state, invokes reducers, publishes new state
- **Reducers**: Combine current state and action to produce a new state


---

#Implementing unidirectional data flow

---

#Representing State

```swift
typealias TimelineState = (serverState: [Tweet], localState: [Tweet])

typealias TimelineMergedState = (serverState: [Tweet], localState: [Tweet], mergedState: [Tweet])
```

---

#Favoriting a Tweet

Within the `TimelineViewController` we trigger the state change by dispatching an Action Creator:

```swift
  func didFavorite(tweetTableViewCell:TweetTableViewCell) {
    let currentTweet = tweetTableViewCell.tweet!
    
    if (currentTweet.isFavorited) {
      timelineDispatcher.dispatch { TimelineActionCreator.unfavoriteTweet(currentTweet) }
    } else {
      timelineDispatcher.dispatch { TimelineActionCreator.favoriteTweet(currentTweet) }
    }
    
    timelineDispatcher.dispatch { TimelineActionCreator.syncFavorites() }
  }
```

---

#Favoriting a Tweet

Within the `ActionCreator` we generate an according action:

```swift
  static func favoriteTweet(tweet: Tweet) -> ActionCreator {
    return { state, dispatcher in
      return .FavoriteTweet(tweet)
    }
  }
```

^
- This indirection is helpful, because the ActionCreator receives a reference to the current state and the dispatcher
- This means the ActionCreator can decide which action to create based on the current state and it can also dispatch delayed actions

---


#Favoriting a Tweet

Within the `Reducer` we generate a new state based on the old one:

```swift

struct TimelineReducers {
  //...
  static func favoriteTweet(var state: TimelineState, tweet: Tweet) -> TimelineState {
  	//...
  }
	
}
```

---

#Favoriting a Tweet

```swift
  static func favoriteTweet(var state: TimelineState, tweet: Tweet) -> TimelineState {
    let newTweet = Tweet(
      content: tweet.content,
      identifier: tweet.identifier,
      user: tweet.user,
      type: tweet.type,
      favoriteCount: tweet.favoriteCount,
      isFavorited: true
    )
    
    let tweetIndex = find(state.localState, tweet)
    
    if let tweetIndex = tweetIndex {
      // if we have stored local state for this tweet previously, override here
      state.localState[tweetIndex] = newTweet
    } else {
      // else append new state
      state.localState.append(newTweet)
    }
    
    return state
  }
```

^
- Create a new Tweet and add it to the local state of the store
- What does the store look like?

---

#Favoriting a Tweet

Then the `Dispatcher` forwards the new state to its subscribers, which inlcudes the `TimelineViewController`:

```swift
extension TimelineViewController: TimelineSubscriber {
  
  func newState(state: TimelineMergedState) {
    tweets = state.mergedState
  }
  
}
```

---

#Syncing Change

>

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


#Summary

![inline fill 150%](images/MergingState.png)  ![inline fill 50%](images/InformationFlowNoCaption.png)

---

#Benefits of a Value Oriented Architecture

- Confidence that no one will change our data under the covers
- Change propagation needs to be handled explicitly
- Modeling change as data opens opportunities:
	- Undo Functionality
	- Simpler client server sync

---
#[fit]The Value Mindset

**Twitter**: @benjaminencz

**Project & Slides:** https://github.com/Ben-G/TwitterSwift

**Related, great talks:** Controlling Complexity [^20], The Value of Values [^21]

[^20]: https://realm.io/news/andy-matuschak-controlling-complexity/

[^21]: http://www.infoq.com/presentations/Value-Values

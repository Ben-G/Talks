footer: Safer Swift Code with Value Types | @benjaminencz | 360iDev, August 2015 
slidenumbers: true

#[fit]Safer Swift Code 
#[fit]with Value Types

^
- My name is Benjamin Encz
- iOS Developer at Make School
- Today I want to talk about a practical way of implementing a value oriented architecture
- Want to focus on potential benefits that come from it

---

#What do I mean by safety?

*Safe*, adjective [^*]:
- involving little or no risk of mishap, error, etc.
- dependable or trustworthy

[^*]: Thanks to Rich Hickey for pointing out this great way to open a talk


^
- Thanks to Rich Hickey, creator of Clojure, for this great inspiration on how to open a talk
- Take the dictionary definiton of words you use in title
- Two definitions of safety fit this context especially well
- Hopefully I can show that an architecture based on value types can often be more reliable and predictable

---

#Should I use a struct or a class?

It depends! [^4] [^5] [^6] [^7] [^8] [^9]

[^4]: GitHub Styleguide: "Unless you require functionality that can only be provided by a class (like identity or deinitializers), implement a struct instead"

[^5]: Apple Developer Guide: "In practice, this means that most custom data constructs should be classes, not structures"

[^6]: Apple Swift Blog: "One of the primary reasons to choose value types over reference types is the ability to more easily reason about your code."

[^7]: http://faq.sealedabstract.com/structs_or_classes/: "when it comes time to actually save any of those models to disk, or push them over a network, or draw them to the screen, or whatever, you need a class. Classes everywhere. Not "immutable valuable types everywhere" like in the Struct Philosophy™"

[^8]: http://owensd.io/2015/07/05/re-struct-or-class.html

[^9]: https://www.mikeash.com/pyblog/friday-qa-2015-07-17-when-to-use-swift-structs-and-classes.html

^
- I gave a very first version of this talk at the beginning of this year, since then discussion about struct vs. class has been an ongoing source of discussion
- A lot of people have given advice on this, Apple itself has contradicting opinions on their blog
- Today I want to show an example in which the entire model layer is implemented with value types
- We are all very familiar with reference type based, mutable applications, today I hope to provide insight into a potential solution for a value oriented architecture 

---

# Agenda

1. **What** are Value Types vs. Reference Types
2. **Why** is this topic relevant now?
3. **How:** A Practical Example of a Value Oriented Architecture

^
- 3 Main Parts
- What: make sure we are on the same page
- Why now?
- How is the main point of the presentation! Listened great talk by Andy Matuschak that inspired this talk
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
- Can have methods
- Can conform to protocols

^ 
- Swift promotes the use of enums/structs by giving them almost all the capabilities of classes
- You might wonder, which capabilities are missing from value types that are available in reference types?

---
#So What Can't They Do? 

> “Indeed, in contrast to structs, Swift classes support  **implementation inheritance**, (limited) reflection,  deinitializers, and **multiple owners**.” 

Andy Matuschak[^1]

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
#Twitter Client

1. Download the latest 50 tweets and display them
2. Allow to filter tweets (RT only, favorited tweets only, etc.)
3. *Allow user to favorite tweets (should be synced with server)*

---

#Twitter Client

![inline image](images/twitterapp.png)

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
let lockQueue = dispatch_queue_create("com.happysyncing", nil)
dispatch_sync(lockQueue) {
	tweet.favorited = true
}
```
^
- Except we need to lock to make sure that the same reference is not accessed from multiple threads at the same time

---

#Is It Simple with OOP?

```swift
let lockQueue = dispatch_queue_create("com.happysyncing", nil)
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
let lockQueue = dispatch_queue_create("com.happysyncing", nil)
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

#OOP Data Flow

![inline](images/InformationFlowOOP.png) 

^
- Here's visualization of what discussed so far
- Information flow from model to view and view to model
- Multiple components share references to mutable objects
- However, we might have different objects that represent the same tweet (e.g. multiple queries to core data)
- Updates are propagated manually and information can flow between all components involved

---

#Is the OOP Data Flow Safe?


![left 50%](images/InformationFlowOOPNoCaption.png) 

*Safe*, adjective:
- involving little or no risk of mishap, error, etc.
- dependable or trustworthy

^
- Is this programming model safe?
- Not always!
- A lot of thought needs to go into any part of application that manipulates state
- We need to ensure correct state isolation and propagation -> a lot of room for error


---

#Is the OOP Data Flow Safe?


![left 50%](images/InformationFlowOOPNoCaption.png) 

*Safe*, adjective:
- involving little or no risk of mishap, error, etc.
- dependable or trustworthy

**-> A lot of room for error**


^
- Is this programming model safe?
- Not always!
- A lot of thought needs to go into any part of application that manipulates state
- We need to ensure correct state isolation and propagation -> a lot of room for error


---

#[fit]Modelling Change
#[fit]is Hard!

^
- This is true for object oriented approach and the value oriented approach

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

^
- Every change yields a new Tweet instance
- We keep local state and server state separated
- Store provides a view by combining server and local state
- The store can use local state list to trigger API requests that syncs that local state

---

#How Can We Model Change With Immutable Value Types?

**Model change to values as values:**

* Create a new Tweet for every change
* Save these mutations in a local change set

^
- Every change yields a new Tweet instance
- We keep local state and server state separated
- Store provides a view by combining server and local state
- The store can use local state list to trigger API requests that syncs that local state

---

#How Can We Model Change With Immutable Value Types?

**Model change to values as values:**

* Create a new Tweet for every change
* Save these mutations in a local change set
* Save the server state in a separate data set

^
- Every change yields a new Tweet instance
- We keep local state and server state separated
- Store provides a view by combining server and local state
- The store can use local state list to trigger API requests that syncs that local state

---

#How Can We Model Change With Immutable Value Types?

**Model change to values as values:**

* Create a new Tweet for every change
* Save these mutations in a local change set
* Save the server state in a separate data set
* Provide a merged view on list of tweets

^
- Every change yields a new Tweet instance
- We keep local state and server state separated
- Store provides a view by combining server and local state
- The store can use local state list to trigger API requests that syncs that local state

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

#Where do we store state, how is new state distributed?

^
- We can no longer mutate values in place and rely on implicitly shared state
- We need to think about a *central authority* that can handle that for us 
- The first time around I came up with a custom solution to this problem
- Since then explored some existing ones
- Decided to try out the *Flux* architecture that facebook is using for their web frontend

---

#Where do we store state, how is new state distributed?

Drawing inspiration from the JavaScript world [^11] [^12]:

-> **Unidirectional Data Flow**

[^11]: https://facebook.github.io/flux/

[^12]: https://github.com/rackt/redux

---

#Unidirectional data flow

![inline 66%](images/InformationFlow.png)

---

![left 55%](images/InformationFlowNoCaption.png)

#Unidirectional data flow

- **ActionCreator**: Provides interface for business logic
- **Action**: Represents an individual state mutation
- **Dispatcher**: Stores state, invokes reducers, publishes new state
- **Reducers**: Pure functions, combine current state and action to produce a new state

^
- Views and View Controllers trigger actions through ActionCreators
- ActionCreators are sent to Dispatcher
- Dispatcher invokes action creator, Action creator can create action
- Dispatcher sends created action (if any) to a reducer
- Dispatcher distributes new state to all subscribers
- State only changes through actions; there's no other way

---

#Implementing unidirectional data flow

---

#Representing State

- State will be stored in the Dispatcher in the following form:

```swift
typealias TimelineState = (serverState: [Tweet], localState: [Tweet])

typealias TimelineMergedState = (serverState: [Tweet], 
                                  localState: [Tweet], 
                                 mergedState: [Tweet])
```
^
- State is represented as tuple of serverState and localState
- TimelineMergedState is used to provide the UI with a list of tweets that is computed by serverState and localState

---
#Defining Actions

- Each action describes an atomic mutation to the application state
- Action Creators vend these Actions Reducers implement them

```swift
enum Action {
  case Mount
  case FavoriteTweet(Tweet)
  case UnfavoriteTweet(Tweet)
  case SetServerState([Tweet])
  case SetLocalState([Tweet])
}
```

^
- Actions describe individual operations that can change the state of the app
- Mount populates the initial state
- Favoriting & Unfavoriting modify individual tweets
- Methods to set entire local and server state are used for server sync
- E.g. when server sync completes, entire server state is reset

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

^
- Dispatcher and Action Creators serve as interface to the application logic
- When we want to modify state, we dispatch an action creator
- Here we dispatch favorite/unfavorite
- Then we dispatch synchronization

---

#Favoriting a Tweet

Within the `ActionCreator` we generate an according action:

```swift
  static func favoriteTweet(tweet: Tweet) -> ActionCreator {
    return { state, dispatcher -> Action? in
      return .FavoriteTweet(tweet)
    }
  }
```

^
- This indirection is helpful, because the ActionCreator receives a reference to the current state and the dispatcher
- This means the ActionCreator can decide which action to create based on the current state and it can also dispatch delayed actions
- This is the simplest possible example of an action creator
- Synchronization action creator is more complex, dispatches after callback instead of returning an action

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

^
- The reducer is where the state modification happens
- it receives current state and relevant data for the action and returns the new state

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
- Here's the interesting part, we derive new state from existing one
- First create new favorited tweet
- Then check if tweet is already in local change list (in this app we only remember the latest local change)
- If not, append to local state
- Then return new state

---

#Favoriting a Tweet

Then the `Dispatcher` forwards the new state to its subscribers, which inlcudes the `TimelineViewController`:

```swift
extension TimelineViewController: TimelineSubscriber {
  
  func newState(state: TimelineMergedState) {
    // table view is reloaded when 'tweets' is set
    tweets = state.mergedState
  }
  
}
```
^
- Dispatcher invokes subscribers with new state
- Subscriber updates UI
- In this case table view is reloaded whenever `tweets` is set to a value

---

#Favoriting a Tweet: Recap

![inline](images/InformationFlowNoCaption.png)

^
- We saw a lot of code
- In case you lost track, here's again the overview of the information flow
- UI -> Dispatcher -> Reducer -> UI


---

#Syncing Change

>

^ 
- another interesting feature with this architecture is server sync
- Want to discuss briefly conceptually, but not dive into code 

---

#Syncing Change

1. Iterate over each mutation in the local change set
	
^
- Changes that couldn't be synced successfully stay in local change set and get synced again later 

---

#Syncing Change

1. Iterate over each mutation in the local change set
2. Generate API request that syncs that local change to server
	
^
- Changes that couldn't be synced successfully stay in local change set and get synced again later 

---

#Syncing Change

1. Iterate over each mutation in the local change set
2. Generate API request that syncs that local change to server
3. Upon each API response:
	- If success: remove tweet from local change set
	- If failure: leave tweet in local change set
	
^
- Changes that couldn't be synced successfully stay in local change set and get synced again later 

---


#A Twitter Client built on Immutable Value Types

![inline fill 150%](images/MergingState.png)  ![inline fill 50%](images/InformationFlowNoCaptionNoArrow.png)

^
- Two main ingredients for this architecture: Store local modifications separately from server state; generate merged view + flux unidirectional data flow

---

#Should I use a struct or a class?


#It's an architectural question!

It's not about `struct` vs. `class`

It's about:
- Shared state vs. isolated state
- Mutable state vs. immutable state [^10]

[^10]: WWDC 2014, Session 229, Advanced iOS Application Architecture and Patterns

^
- For me this is primarily an architectural question
- Do you want implicitly shared, mutable state in your data model? Sometimes can be easier!
- Or do you prefer isolated immutable state?
- The latter can be accomplished with immutable reference types as well

---

#Downsides of a Value Oriented Architecture

- Unconventional: can be an issue when working on larger teams
- Can be difficult to integrate with frameworks that rely on reference semantics, e.g. Core Data

---

#Benefits of a Value Oriented Architecture

- Confidence that no one will change our data under the covers
- State modification & propagation needs to be handled explicitly and can be implemented in one place
- Modeling change as data opens opportunities, e.g. simpler client server sync
- Easier to test

^
- Implementing state modification & propagation in one place means that we can reuse the code for multiple types, this is alternative to inhertiance


---
#[fit]The Value Mindset

**Twitter**: @benjaminencz

**Project & Slides:** https://github.com/Ben-G/TwitterSwift

**Related, great talks:** Controlling Complexity [^20], The Value of Values [^21]

[^20]: https://realm.io/news/andy-matuschak-controlling-complexity/

[^21]: http://www.infoq.com/presentations/Value-Values

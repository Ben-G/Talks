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

---

#Swift Standard Library

**Reference Types:**
- `ManagedBuffer`(?)
- `NonObjectiveCBase`(??)

**Value Types:**
- `Array`
- `String`
- `Optional`

---

#Enums and Structs in Swift are Powerful

- Can have variables
- Can have functions
- Can implement protocols

^ Swift promotes the use of enums/structs by giving them almost all the capabilities of classes

---
#So What Can't They Do? 

> “Indeed, in contrast to structs, Swift classes support  implementation inheritance, (limited) reflection,  deinitializers, and multiple owners.” - Andy Matushak (http://www.objc.io/issue-16/swift-classes-vs-structs.html)

---

#We've already been doing this!

```objectivec
@property (copy) NSString *userName;
```

---

#[fit]Case Study
#[fit]A Twitter Client Built on Immutable Value Types

---
#Twitter Client

1. Download the latest 200 tweets and display them
2. Allow to filter tweets (RT only, favorited tweets only, etc.)
3. Allow user to favorite tweets (should be synced with server)

---

![inline 120%](images/OOP_Twitter.png)

---

![inline 120%](images/fetchtweets.png)

---

#[fit]How can we favorite Tweets?

---

#It Is Very Simple with OOP

```swift
tweet.favorited = true
```

---

#It Is Simple with OOP

```swift
let lockQueue = dispatch_queue_create("com.happylocking", nil)
dispatch_sync(lockQueue) {
	tweet.favorited = true
}
```

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

---

#Modelling Change is Hard!

- Protect against unwanted updates
- Distribute new value
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

---

![inline](images/MergingState.png)

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

---

#Modelling Change

```swift
class TweetStore {
 
  var tweets: [Tweet]? {
    get {
      return mergeListIntoListLeftPriority(localState, serverTweets!)
    }
  }
  
  func addTweetChangeToLocalState(tweet: Tweet) {
    let index = find(localState, tweet)
    if let index = index {
      localState[index] = tweet
    } else {
      localState.append(tweet)
    }
  }
  
  //...
```

---

![inline](images/ValueType_Twitter_Marked.png)

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


---

#Benefits of "Value Oriented" Architecture

- Seperation of behavior and data encourages abstraction
- Change propagation needs to be handled explicitly
- No false sense of *identity*
- Treating change as data opens opportunities:
	- Undo Functionality
	- Sophisticated conflict resolution
	- ... 	

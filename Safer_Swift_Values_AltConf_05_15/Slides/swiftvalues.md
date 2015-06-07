footer: Safer Swift Code with Value Types | @benjaminencz | AltConf, June 2015 
slidenumbers: true

# Safer Swift Code with Value Types

---

#What do I mean by safety?

![image](images/safety.png)

---

## Agenda

1. **What** are Value Types vs. Reference Types
2. **Why** is this topic relevant now?
3. **How:** A Practical Example of a "Value Oriented" Architecture

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

#Enums and Structs in Swift

- Can have variables
- Can have functions
- Can implement protocols

---

> “Indeed, in contrast to structs, Swift classes support  implementation inheritance, (limited) reflection,  deinitializers, and multiple owners.” - Andy Matushak

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

![inline](images/ValueType_Twitter.png) 

---

![inline](images/ValueType_Twitter_Marked.png)

---

#Why the **** so complicated??

```swift
tweet.favorited = true
```

---

#Why the **** so complicated??

```swift
tweet.favorited = true
```

---

#Why so complicated??

```swift
let lockQueue = dispatch_queue_create("com.happylocking", nil)
dispatch_sync(lockQueue) {
	tweet.favorited = true
}
```

---

#Why so complicated??

```swift
let lockQueue = dispatch_queue_create("com.happylocking", nil)
dispatch_sync(lockQueue) {
	tweet.favorited = true
	NSNotificationCenter.defaultCenter().
		postNotificationName("Tweet Changed", object: tweet)
}
```

---

#Why so complicated??

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

#Why so complicated??

- Protect against unwanted updates
- Distribute new value
- Understand what the underlying *identity* of an object is and perform update accordingly

---

#Why so complicated??

- Protect against unwanted updates
- Distribute new value
- Understand what the underlying *identity* of an object is and perform update accordingly

-> **We need to this in all places where we mutate values!**


---
#[fit]Modelling Change
#[fit]is Hard!
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
  tweets = store.tweets
```

---

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
    
    postFavorites()
  }
  
  //...
```

---



---

#Benefits of "Value Oriented" Architecture

- Seperation of behavior and data encourages abstraction
- Change propagation needs to be handled explicitly
- No false sense of *identity*

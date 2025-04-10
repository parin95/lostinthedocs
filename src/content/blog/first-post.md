---
title: '🗑️ Garbage Collection vs ARC'
description: 'I thought I was being clever with weak references. Turns out, Swift had other plans.'
pubDate: 'Apr 10 2025'
heroImage: '/gc-arc-meme.png'
---

I was reading the [Swift ARC docs](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting#Weak-References) and came across this line:

> In systems that use garbage collection, weak pointers are sometimes used to implement a simple caching mechanism [...] However, with ARC, values are deallocated as soon as their last strong reference is removed, making weak references unsuitable for such a purpose.

I had to read it twice. Then a third time.

Wait… so weak references aren’t great for caching in Swift? Why not?

Let’s rewind a bit.

---

### 🧠 What I thought I understood

I knew that:

* **Strong references** keep an object alive.
* **Weak references** don’t.
* Weak is useful to avoid retain cycles (hello, `[weak self]`).

So I thought: *"If I store something in a cache using a weak reference, I’m not bloating memory, but I might still get a hit if no one’s deleted it yet. Nice."*

That’s how it works in languages like Java or Python, which use Garbage Collection (GC). But Swift doesn’t use GC. It uses Automatic Reference Counting (ARC).

And that difference matters a *lot*.

---

### 🚫 What actually happens in Swift

In GC systems, objects stick around in memory even after the last strong reference is gone — until the garbage collector gets around to cleaning up. So if your cache holds a weak reference, there's a decent chance the object is still sitting in memory when you check.

But Swift’s ARC is instant. The moment the last strong reference goes away?

💥 Deallocated. Gone. Poof.

---

### 🧪 A tiny experiment

Here’s a quick demo that made it all click:

```swift
class Image {}

class Cache {
    weak var image: Image?
}

func testCache() {
    let cache = Cache()

    do {
        let img = Image()
        cache.image = img // stored weakly
        print("Image assigned to cache")
    }

    // img goes out of scope here
    print("Cached image: \(String(describing: cache.image))") // nil
}

testCache()
```

Output:

```
Image assigned to cache
Cached image: nil
```

Even though I just assigned the image to the cache, it was immediately deallocated once img went out of scope — because the cache only held a weak reference.

So yeah, that cache? Basically empty unless something else keeps your object alive.

---

### 💡 Takeaways

* Don’t use `weak` for caching in Swift.
* If you want an actual cache, use **strong references** and **evict items yourself** when needed.
* `NSCache` is a great option — it manages memory for you, works like a dictionary, and automatically purges under memory pressure.

---

### 🧭 Final Thought

This was one of those “Wait... what?” moments for me. It looked familiar because of my past experience in other languages, but Swift’s memory model is just fundamentally different.

And once I understood why it works that way, it made total sense.
Just took me a minute (okay, a few hours) to get there.

---

*✏️ Got stuck on something else in the Swift book? You’re not alone. I’ll be back with another post soon. Probably about unowned, because… yeah.*

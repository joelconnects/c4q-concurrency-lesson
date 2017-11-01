# Concurrency

## Objective

* To understand concurrency and its essential role in iOS development
* To understand how queues and asynchronous operations facilitate a smooth user experience
* Introduce Apple's `Dispatch` API

## Vocabulary

- **concurrency** - the execution of more than one task at a time.
- **thread** - a construct where tasks are executed. Tasks can start, stop, and finish on one thread or across multiple threads.
- **queue** - a sequenced collection of tasks where the first task is added in the back (enqueue) and removed from the front (dequeue), also referred to as First-In-First-Out (FIFO).
- **asynchronous** - the ability to start a task and move on to another task without waiting for the initial task to complete.
- **callback** - the mechanism used to signal the completion of a task (usually a long running task).

## Readings

[Concurrency](https://en.wikipedia.org/wiki/Concurrency_(computer_science)) <br />
[Queue](https://en.wikipedia.org/wiki/Queue_(abstract_data_type)) <br />
[Asynchronous](https://en.wikipedia.org/wiki/Asynchronous_I%2FO) <br />
[Thread](https://en.wikipedia.org/wiki/Thread_(computing))

## Resources

[Apple/Documentation/Dispatch](https://developer.apple.com/documentation/dispatch) 

## Main Lesson

### Before We Get Started

It's important to point out that concurrency will probably make your head hurt at first. Therefore, the goal of this lesson is not to master concurrency and its associated terms but to have a general understanding of the concept.

### Introduction

One big reason that concurrency matters is its positive affect on the user experience. To illustrate this, imagine scrolling through a feed on any app, be it Facebook, Instagram, etc. Consider the moment when the feed stops loading media and/or scrolling feels "choppy". As a user this experience is irritating and prevents the consumption of adorable pet videos. Hopefully your next question is, "how does concurrency help me watch pet videos?"

The answer is juggling. Concurrency allows tasks to be performed (i.e., juggled) at the same time. This means the processor in your device can make a network call to get the latest video while also making sure your screen refreshes 60 times per second. Performance stalls when tasks are not efficiently executed and the app slows or crashes as a result.

It's important to mention that modern computers and devices have multi-core processors. This allows for tasks to be carried out on different cores at the same time. This is called [parallelism](https://en.wikipedia.org/wiki/Parallel_computing). If concurrency is like juggling then parallelism is like having more than one juggler. 

### Concurrency in iOS

Now that you've been introduced to concurrency, the next step is understanding its use in iOS development. Apple handles the nitty gritty of managing tasks but you will need (and want) to take ownership of some aspects of the process. Apple provides two different APIs to manage tasks, `Dispatch` and `Operation`. This lesson introduces `Dispatch`. 

**Multithreading** 

Apple performs the complex coordination and execution of tasks using threads. A thread is simply a pathway to execute tasks within an application. While the concept of a thread is straightforward, managing threads is not. This is where `Dispatch` plays an important role by providing access to threads at a high-level via queues.

A task can either run on the main thread or a background thread. The main thread is where UI-related tasks are performed. As mentioned previously, the screen is refreshing constantly to maintain a seamless appearance. This is possible because the main thread is always churning through tasks without blockage or interruption. A task can become an issue on the main thread when it takes too long (picture a slow car driving in the fast lane). If the car slows down enough it will create traffic or deadlock and the same goes for a task on the main thread.

A background thread is there to accommodate slower tasks. A common example is a network request for an image. Whether the file size is small or large, it will take more time to complete the task than what needs to happen immediately on the screen. 

**Queue**

If you've every stood in line somewhere (e.g., maybe to get an iPhone?), you've been in a queue. As with any line, you enter the line at the back and exit at the front. This is referred to as first-in-first-out. `Dispatch` provides queues to line up tasks for execution but still hides the complexities of thread management. A queue can be designated as serial or concurrent. Serial means tasks run in order (i.e., a task cannot start until the previous task is completed). Concurrent means tasks can run simultaneously. It also means one task may start first but finish before or after another task in the same queue.

**Asynchronous**

Next up is the option to run a queue asynchronously. Have you ever heard someone say, "Don't wait for me, I'll catch up"? That's exactly how the queue behaves running asynchronously. The UI is like your friend that's always in a hurry and can't be bothered to wait for anyone. By running the queue asynchronously, the main thread does not have to wait on a time intensive task to be completed. To revisit the image fetching example, the ability to interact and see the user interface should always be the top priority. Asynchronous calls allow the surrounding code to continue executing without interruption.

**Callback**

Callbacks are a means of communicating when a task has been completed. As you will see in the examples below, closures in Swift afford you the opportunity to communicate back and forth between parts of your code. This is useful when you call a function to perform a task that will take some time. The function can include a parameter to act as a callback when the task is completed. For example, if you order a food delivery (call a function), you can still do things around the house (make other calls). When the delivery person rings the bell (callback), you can pause what you're doing and eat (perform task resulting from original function call).

## Exercise

The exercise is a truncated example of the image request flow. Notice the queue that is used when the image is passed into the callback and received back in the `ViewController`. `Dispatch` has a `main` type property that gives you direct access to add tasks to the main queue to be run on the main thread. `.async` indicates the tasks will run asynchronously allowing the view controller to continue doing other work. 

Earlier you learned **UI tasks run on the main thread**. What may not be obvious is the `networkRequestForImage(callback:)` method will most likely get bumped to a background thread even though a background queue is not specified. Remember Apple does a lot of the heavy lifting in relation to task management. It's your job as a developer to recognize tasks that take more time so you can respond accordingly in your code. In this example "jumping" on the main queue is necessary because the image is UI-related. If you don't jump on the main queue, the UI can become unresponsive and in the worst case, the app will crash.

Copy/paste the code below into an empty playground file in Xcode. Pay attention to the order of statements printed to the console. Try commenting out the `DispatchQueue.main.async` call but still running the print statement inside the closure. What do you notice? Did the order of statements in the console change? Why?

```swift
import UIKit
import PlaygroundSupport

// Needed for async calls in Playground
PlaygroundPage.current.needsIndefiniteExecution = true

// Main
class ViewController {
  static func performTasks() {
    print("UI Task One")
    APIClient.networkRequestForImage { image in
      DispatchQueue.main.async {
        print("\(image) ready for display")
      }
    }
    print("UI Task Two")
  }
}

// Background
class APIClient {
  static func networkRequestForImage(callback: (UIImage) -> ()) {
    
    // Perform network request to get image
    let imageFromServer = UIImage()
    
    callback(imageFromServer)
  }
}

ViewController.performTasks()
```

## Summary

Concurrency in iOS is a big topic. This lesson only scratched the surface by introducing you to `Dispatch`, the main queue, executing tasks asynchronously, and why the main thread is always focused on UI tasks. Your challenge is to go deeper. With a more challenging topic, try watching/reading various videos/articles from multiple sources. Every person will explain the topic a bit differently and that can be very helpful. You can also create a project in Xcode and play around with making a working app from the example above. Check out the [OMDb API](http://www.omdbapi.com/) or another API to make a request, get some info/images, and put something on screen. Have fun!

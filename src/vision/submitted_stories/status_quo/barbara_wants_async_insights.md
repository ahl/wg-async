# 😱 Status quo stories: Barbara wants Async Insights

## 🚧 Warning: Draft status 🚧

This is a draft "status quo" story submitted as part of the brainstorming period. It is derived from real-life experiences of actual Rust users and is meant to reflect some of the challenges that Async Rust programmers face today.

If you would like to expand on this story, or adjust the answers to the FAQ, feel free to open a PR making edits (but keep in mind that, as they reflect peoples' experiences, status quo stories [cannot be wrong], only inaccurate). Alternatively, you may wish to [add your own status quo story][htvsq]!

## The story

[Barbara] has an initial prototype of a new service she wrote in sync Rust. She then decides, since the service is extremely I/O bound, to port it to async Rust and her benchmarks have led her to believe that performance is being left on the table.

She does this by sprinkling `async/.await` everywhere, picking an executor, and moving dependencies from sync to async.

Once she has the program compiling, she thinks "oh that was easy". She runs it for the first time and surprisingly she finds out that when hitting an endpoint, nothing happens.

Barbara, always prepared, has already added logging to her service and she checks the logs. As she expected, she sees here that the endpoint handler has been invoked but then... nothing. Barbara exclaims, "Oh no! This was not what I was expecting, but let's dig deeper."

She checks the code and sees that the endpoint spawns several tasks, but unfortunately those tasks don't have much logging in them.

Barbara knows that debugging with a traditional debugger is not very fruitful in async Rust. She does a deep dive into the source code and doesn't find anything. Then she adds much more logging, but to her dismay she finds that a particular task seems stuck, but she has no idea why.

She really wishes that there was a way to get more insight into why the task is stuck. These were the thoughts inside her head at that moment:
* Is it waiting on I/O?
* Is there a deadlock?
* Did she miss some sync code that might still be there and messing with the executor?

For the I/O question she knows to use some tools on her operating system (lsof). This reveals some open sockets but she's not sure how to act on this.

She scans the code for any std lib imports that might be blocking, but doesn't find anything.

After hours of crawling through the code, she notices that her task is receiving a message from a bounded async channel. She changes this to be an unbounded channel and then things start working.

She wants to know why the code was not working, but unfortunately she has no way to gain insight into this issue. She fears that her task might use too much memory knowing that the channel is unbounded, but she can't really tell.

She thinks, "Anyhow it is working now, let's see if we got some performance gains." After thorough benchmarking she finds out that she didn't quite get the performance gain she was expecting. "Something is not working, as intended", she thinks.

## 🤔 Frequently Asked Questions

### **What are the morals of the story?**
* There are very few ways to get insights into running systems. Tracing is state of the art. `console.log` #ftw
* Tracing is a static activity and there's no way to dynamically gain insights.
* While it's possible to find solutions to these issues, often you don't have insight into if those solutions bring new problems.
* Debugging process for non-trivial issues is almost guaranteed to be painful and expensive.

### **What are the sources for this story?**
[Issue 75](https://github.com/rust-lang/wg-async/issues/75)

### **What are examples of the kinds of things a user might want to have insight into?**
* Custom Events - logging/tracing (Per task?)
* Memory consumption per task.
* I/O handles in waiting state per task.
* Number of tasks and their states over time.
* Wake and drop specific tasks.
* **Denoised** stack traces and/or stack traces that are task aware.
* Who spawned the task?
* Worker threads that are blocked from progressing tasks forward.
* Tasks that are not progressing.

### **Why did you choose [Barbara] to tell this story?**
Barbara knows what she's doing, but still there is little way to get insights.

### **How would this story have played out differently for the other characters?**
Depending on what languages he was using before, [Alan] would likely have had experience with a stronger tooling story:
* The highly [debuggable](https://youtu.be/JvBT4XBdoUE) BEAM (a VM), for Erlang.
* [Delve](https://github.com/go-delve/delve), the debugging tool for Go.
* Using [Visual Studio](https://devblogs.microsoft.com/visualstudio/how-do-i-debug-async-code-in-visual-studio/) to debug C#.
* Debugging async Java using [IntelliJ](https://www.jetbrains.com/help/idea/debug-asynchronous-code.html).

[character]: ../../characters.md
[status quo stories]: ../status_quo.md
[Alan]: ../../characters/alan.md
[Grace]: ../../characters/grace.md
[Niklaus]: ../../characters/niklaus.md
[Barbara]: ../../characters/barbara.md
[htvsq]: ../status_quo.md
[cannot be wrong]: ../../how_to_vision/comment.md#comment-to-understand-or-improve-not-to-negate-or-dissuade

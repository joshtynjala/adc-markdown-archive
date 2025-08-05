# Performance-tuning Adobe AIR applications

by Oliver Goldman

![Oliver Goldman](./img/oliver_goldman_bio.jpg.adimg.mw.160.png)

## Requirements

### User level

All

### Required products

- [Adobe AIR SDK](https://airsdk.dev/) or
  [Apache Flex SDK](https://flex.apache.org)

Application performance is perennial. It's in its nature. In order for an
application to perform well, every part of the application has to perform well.
Lapse in one area and it brings your entire application down. It's difﬁcult to
write a large application without letting your guard down once in a while.

Questions about performance often indicate a failure to understand this
weakest-link-in-the-chain aspect of the problem. Here are some of my favorite
lousy questions about performance and AIR applications:

- Will my AIR application be fast?
- Is AIR fast enough to do X?
- Isn't AIR too slow to do Y?

(Here's proof also that no matter what your kindergarten teacher told you, there
_is_ such a thing as a lousy question.)

AIR almost never makes it impossible to achieve good performance in your
application. On the other hand, AIR can't do it for you, either. Like I said,
it's the nature of the problem.

Fortunately, standard tuning techniques apply to AIR as much as they'd apply to
writing any piece of desktop software.

### Asking good questions

Achieving good performance starts, like most engineering problems, with
understanding the problem you're trying to solve. Here are some good questions
to ask about your application:

- Which operations in my application are performance sensitive?
- What metric can I use to measure this sensitivity?
- How can I optimize my application to that metric?

Most applications contain a lot of code that runs well enough. Don't spend your
time on that stuff, especially if any gains would be below the threshold at
which users could notice them. Make sure you're focused on things that matter.

Common examples of operations worth optimizing are:

- Image, sound, and video processing
- Rendering large data sets or 3D models
- Searching
- Responding to user input

### Defining metrics

Performance is often equated with speed, but don't be lulled into thinking
that's the only metric that matters. You may ﬁnd that you need to tune for
memory use or battery life. Applications that minimize use of these may also be
considered better performing than those that don't. Sometimes optimizing for
other metrics also speeds things up, but other times trade-offs are required.

Regardless of what you're measuring, _you must have something to measure._ If
you're not measuring anything, you can't tell whether changes improve
performance or harm it. Good metrics have these three properties:

- They're quantiﬁable. You can measure them and record them as a number.
- They're consistent. You can measure them repeatedly and usefully compare
  measurements.
- They're meaningful. Changes in the measured value correspond to the thing
  you're optimizing for.

To make this concrete, suppose you're writing an application that's going to
perform some image-processing tasks on a large set of images. During the
processing, the application needs to display feedback on its progress to the
user. It also needs to allow the user to cancel an operation, rather than
waiting for it to complete. This is a simple application, but even it has at
least three interesting metrics that we can examine.

#### Example: Throughput

The ﬁrst and most obvious metric is _throughput._ It's meaningful, in this
example, because we know we must process a large number of images. The higher
the throughput, the faster that processing completes.

Throughput is easily quantiﬁed as processing per unit time. Although it could be
measured as the number of images processed, measuring the number of bytes can
produce a more consistent value when image sizes vary. Throughput for this
example is easily measured in bytes per millisecond.

#### Example: Memory use

A less obvious metric for this application is memory use. Memory use is not as
visible a metric to end users as is throughput. Users have to run another
application, such as Activity Monitor, in order to monitor memory use. But
memory use can be a limiting factor: run out of memory, and your application
won't work.

Memory use is of interest in our image-processing example because the images
themselves are large. We'd like to be able to process large images—even those
that exceed available RAM—without running out of memory. Memory use is
straightforward to measure in bytes.

#### Example: Response time

The ﬁnal metric for our sample application is one that's often overlooked:
response time to user input. This metric is immediately visible to all of your
users, even if they rarely stop to measure it. It's also pervasive. Users expect
all operations—from resizing windows, to canceling an operation, to typing text,
to respond immediately.

Whereas some metrics are perceived linearly by users, response time has an
important threshold. Any lag in response to input over approximately 100
milliseconds is perceptible to users as _slow_. If your application consistently
responds below this threshold, no further optimization is necessary. Clearly,
this metric is easily quantiﬁed in milliseconds.

Response time is a particular challenge for the image-processing application
because processing any individual image will take well over 100 milliseconds. In
some programming environments this is addressed by handling user input on a
different thread from long-running calculations. Under the covers, this solution
depends on the operating system switching thread contexts quickly enough such
that the user input thread can respond in time. AIR, however, doesn't offer an
explicit threading model and so this switch must be done explicitly. This is
illustrated in the next section. The following sample demonstrates three
different ways of setting up image processing, optimizing for different metrics:

    <?xml version="1.0" encoding="utf-8"?>
    <mx:WindowedApplication xmlns:mx="http://www.adobe.com/2006/mxml"
        layout="horizontal"
        frameRate='45'>
        <mx:Script>
            <![CDATA[
                private static const DATASET_SIZE_MB:int = 100;

                private function doThroughput():void {
                    var start:Number = new Date().time;
                    var data:ByteArray = new ByteArray();
                    data.length = DATASET_SIZE_MB * 1024 * 1024;
                    filter( data );
                    var end:Number = new Date().time;
                    _throughputLabel.text = ( data.length / ( end - start )) + " bytes/msec";
                }

                private function doMemory():void {
                    var start:Number = new Date().time;
                    var data:ByteArray = new ByteArray();
                    data.length = 1024 * 1024;
                    for( var chunk:int = 0; chunk < DATASET_SIZE_MB; chunk++ ) {
                        filter( data );
                    }
                    var end:Number = new Date().time;
                    _memoryLabel.text = ( DATASET_SIZE_MB * data.length / ( end - start )) + " bytes/msec";
                }

                private function doResponse():void {
                    _chunkStart = new Date().time;
                    _chunkData = new ByteArray();
                    _chunkData.length = 100 * 1024;
                    _chunksRemaining = DATASET_SIZE_MB * 1024 / 100;
                    _chunkTimer = new Timer( 1, 1 );
                    _chunkTimer.addEventListener( TimerEvent.TIMER_COMPLETE, doChunk );
                    _chunkTimer.start();
                }

                private function doChunk( event:TimerEvent ):void {
                    var iterStart:Number = new Date().time;
                    while( _chunksRemaining > 0 ) {
                        filter( _chunkData );
                        _chunksRemaining--;

                        var now:Number = new Date().time;
                        if( now - iterStart > 90 ) break;
                    }

                    if( _chunksRemaining > 0 ) {
                        _chunkTimer.start();
                    } else {
                        var end:Number = new Date().time;
                        _responseLabel.text = ( DATASET_SIZE_MB * 1024 * 1024 / ( end - _chunkStart )) + " bytes/msec";
                    }
                }

                private var _chunkStart:Number;
                private var _chunkData:ByteArray;
                private var _chunksRemaining:int;
                private var _chunkTimer:Timer;

                private function filter( data:ByteArray ):void {
                    for( var i:int = 0; i < data.length; i++ ) {
                        data[i] = data[i] * data[i] + 2;
                    }
                }
                private function onMouseMove( event:MouseEvent ):void {
                    var global:Point = new Point( event.stageX, event.stageY );
                    var local:Point = _canvas.globalToLocal( global );
                    _button.x = local.x;
                    _button.y = local.y;
                }

            ]]>
        </mx:Script>

        <mx:HBox width='100%' height='100%'>
        <mx:VBox width='50%' height='100%'>
        <mx:Button label='Measure throughput' click='doThroughput();'/>
        <mx:Label  id='_throughputLabel'/>
        <mx:Button label='Reduce memory use' click='doMemory();'/>
        <mx:Label  id='_memoryLabel'/>
        <mx:Button label='Maintain responsiveness' click='doResponse();'/>
        <mx:Label  id='_responseLabel'/>
        </mx:VBox>
        <mx:Canvas
            width='50%' height='100%'
            id="_canvas"
            horizontalScrollPolicy="off"
            verticalScrollPolicy="off"
            backgroundColor="white"
            mouseMove='onMouseMove( event );'
        >
        <mx:Label text="Move Me" id="_button"/>
        </mx:Canvas>
        </mx:HBox>
    </mx:WindowedApplication>

### Taking measurements

Once you've identiﬁed and deﬁned your metrics but before you can address them,
you must be able to measure them. Only by measuring and tracking your metrics
before and after can you determine the impact of those changes. If possible,
track all of your metrics together so you can see how changes made to optimize
one metric might impact others.

#### Measuring throughput

Throughput can be conveniently measured programmatically. The basic pattern for
measuring throughput is:

    start_msec = new Date().time
    do_work()
    end_msec = new Date().time
    rate = bytes_processed / ( end_msec - start_msec )

#### Measuring memory

Memory is a more complex subject. Most runtime environments, including AIR,
don't provide good APIs for determining an application's memory use. Memory use
is best monitored using an external tool such as Activity Monitor (Mac OS X),
Task Manager (Windows), BigTop (Mac OS X), and the like. After selecting a
monitoring tool, you need to determine _which_ memory metric you want to track.

_Virtual memory_ is the biggest number reported by tracking tools. As the name
suggests, this does not measure the amount of physical RAM the process is using.
It's better thought of as the amount of memory address space the process is
using. At any given time, some portion of the memory allocated to the process is
typically being stored on disk instead of RAM. The amount of RAM plus space on
disk taken together is often thought of as being equivalent to a process'
virtual memory, but it is possible that portions of the address space are in
neither place. The details depend on the operating system and how it allocates
portions of virtual memory for different purposes.

The absolute size of virtual memory your application is using, given what
virtual memory encompasses, is likely not an interesting metric. Virtual memory
of your application relative to other, similar applications may be of interest,
but is still difﬁcult to usefully compare. The most interesting aspect of
virtual memory is its behavior over time: growth without bound generally
indicates a memory leak. Memory leaks may not show up in other memory metrics
because the leaked memory, if not referenced, gets paged to disk and then simply
stays there.

The best memory metric to monitor is _private bytes_, which measures the amount
of RAM your process is using and which is used only by your process. This metric
speaks directly to the impact your application has on the overall system,
courtesy of its use of a shared resource.

Private bytes will ﬂuctuate as your application allocates and de-allocates
memory. It will also ﬂuctuate as your application is active or idle as, when its
idle, some of its pages may be paged to disk. To track private bytes, I
recommend using a monitoring tool to take periodic samples (that is, one per
second) during the operations you're optimizing.

Other memory metrics you may see in monitoring tools include resident size and
shared bytes. Resident size is the total RAM use of your process, made up of
private and shared bytes. Shared bytes are sections of RAM that are shared with
other processes. Usually these sections contain read-only resources, such as
code, from shared libraries or system frameworks. Although you can track these
metrics, applications have by far the most control over—and problems with—the
private bytes value.

#### Response time

Response time is best measured with a stopwatch. Start when the user takes an
action, for example, clicking a button. Stop when the application responds,
typically by changing the displayed user interface. Subtract the two and you
have your measurement.

### The optimization process

With goals and metrics in place you're ready to optimize. The process itself is
straightforward and should be familiar. Repeat these three steps until done:

1.  Measure
2.  Analyze
3.  Modify

Broadly speaking, analysis can lead you to one of two kinds of changes: design
or code.

#### Design changes

Design changes generally have the largest impact. They can be more difﬁcult to
make later in the game, however, so be sure not to wait too long before deﬁning
and measuring against your performance goals.

For an example, let's return to our image-processing application. A naive
implementation might load each image in its entirety into memory, process it,
and then write the results back to disk. The peak memory use (private bytes) of
this application is then primarily a function of the size of the loaded images.
If the images exceed available RAM, the application will fail.

Few image-processing operations are global; most can be performed on one portion
of an image at a time. By dividing the image into ﬁxed-size chunks and
processing them one at a time you can limit the peak memory use of the
application to a number of your choosing. This also enables processing images
that are larger than available RAM.

After modifying your design, be sure to re-evaluate all of your metrics. There
is always some interplay between them as designs are evolved. Those changes may
not always be what you expect. When I prototyped this sample application,
processing images in ﬁxed-size chunks did _not_ signiﬁcantly alter the
throughput of the application, despite my expectation that it would be slower.

#### Code changes

When no further design enhancements present themselves, turn to tuning your
code. There are many techniques to experiment with in this arena. Some are
unique to ActionScript; some are not.

Be careful not to apply code changes too early. They tend to sacriﬁce
readability and structure in the name of performance. This isn't necessarily
bad, but if applied too early they can reduce your ability to evolve and
maintain your application. As Donald Knuth said, "premature optimization is the
root of all evil."

#### Purpose-built test applications

Real-world applications are often large, complex, and full of code that runs
fast enough. To help focus your optimization on key operations, consider
creating a test application for just that purpose.

Among other advantages, the test application provides a place to include
instrumentation (that is, for measuring throughput) without requiring that you
include that code in your ﬁnal application.

Of course, you need to validate that your optimization results still apply when
your improvements are ported back to your application.

### Chunking work

As mentioned earlier, the AIR runtime does not provide a mechanism for executing
application code on a background thread. This is particularly problematic when
attempting to maintain responsiveness during computationally intensive tasks.

Much like chunking in space can be used to optimize memory use, chunking in time
can be used to break up computations into short-running segments. You can keep
your application responsive by responding to user input between segments.

The following pseudo-code arranges to perform about 90 msec of work at a time
before relinquishing control to the main event loop. The main event loop ensures
that, for example, mouse-clicks are processed. With this timing, most user input
will be processed within 100 msec, keeping the application responsive enough
from the user's point of view.

    var timer:Timer = new Timer( 1, 1 )
    timer.addEventListener( TimerEvent.TIMER, doChunk )
    function doChunk( event:Event ):void {
        var start:Number = new Date().time
        while( workRemaining ) {
            doWork()
            var now:Number = new Date().time
            if( now - start > 90 ) {
                // reschedule more work to occur after input
            if( workRemaining )
                timer.start()
            break
            }
        }
    }

In this example, it's important that `doWork()` runs for signiﬁcantly less time
than the chunk duration in order to maintain responsiveness. To keep under 100
msec worse case, it should run for no longer than 10 msec.

Again, re-measure all metrics after adopting an approach like this. In my
image-processing application, my throughput dropped by about 10% after adopting
this chunking approach. On the other hand, my application was responsive within
100 msec to all user input—instead of only between images. I consider that a
reasonable trade-off.

### Wrapping up

Creating high-performance applications isn't easy, but it is a problem that
responds to disciplined measurement, analysis, and incremental improvement. AIR
applications are not fundamentally different in this regard.

Performance is also an evolving target. Not only does each set of improvements
potentially impact your other metrics, but underlying hardware, operating
system, and other changes can also shift the balance between what's fast and
what's slow. Even what you're optimizing for might change over time.

With good practices in place you'll be able to create high-performance AIR
applications—and keep them that way. Just don't let your guard down. All it
takes is one slow feature to have users asking, "Is your application fast enough
to do X?"

> This work is licensed under a
> [Creative Commons Attribution-Noncommercial-Share Alike 3.0 Unported License](https://creativecommons.org/licenses/by-nc-sa/3.0/)

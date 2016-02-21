
# Big Data 220A

## Assignment: Data Acquisition with Kafka

(50 points possible, with extra credit options)

### Announcement (Sat Feb 13, 7pm)

As I mentioned Feb 10 in class, running your code in a docker container is required.  Instructions for running a container with ubuntu and python are part of the recently posted [sample code documentation](https://gitlab0.bigdata220uw.mooo.com/jhenri/uwbd-examples/blob/master/data-acquisition/README.md#run-in-a-docker-container).  I will be happy to address any docker troubleshooting issues via the canvas discussion feature.

### Goals

- Get hands dirty with Kafka.  Hands-on experience with Kafka will be useful both for data acquisition and also for low-latency streaming analytics.  Streaming analytics require backing of streaming storage from Kafka or something a lot like it.  Learning streaming storage first is easier than learning streaming analytics first.
- Get hands dirty with XML and JSON data formats.  Both JSON and XML can represent data that would be difficult to store in CSV or relational tables.  Proficiency with XML and JSON are essential to understanding the strengths of many "big data" and "nosql" technologies.
- Apply one effective pattern for processing large amounts of data, often useful in cases which could not be processed with a Spark or Spark Streaming job alone.

### Problem Statement

Design and implement a data acquisition program to schedule downloading of RSS feeds and RSS feed items.  In class 4, we cast the data acquisition problem as:

- Work items (a set)
- Completed work items (a set)
- storage of every completed work item

In our program, the work items will be a request to obtain a resource.  The record of work items and their completion will be stored in Apache Kafka.  The storage of our completed work items will be a (local) filesystem.

For students finding the minimum requirements easy, a few extra credit add-ons to the minimum requirements will be added, starting with E1).


## Defined terms

| Term | Definition|
|---|---|
| Work item | A request to obtain a resource |
| Schedule | A state of the data acquisition program running, including all completed and uncompleted work items. |

## Nonfunctional Requirements

1) Store all state in Kafka.  Represent your state as one or more Kafka topics.  The number of topics should have a maximum number no matter how much input is provided to the system.  If you're not sure about how many topics to use, start with one topic.

2) Store each message into each topic in Kafka as one line of text containing one JSON dictionary.

3) Killing some or all processes at any time and restarting should result in steady progress toward a completed schedule.

4) Presume that Kafka will be configured to not delete for a very long time.

5) Presume that when your program starts, it will be allowed enough time to consume the whole of the input.

6) (10 points) Provide a README describing how to operate your data acquisition program.

7) Shutdown/cleanup code for kafka consumers is not required.  Programs consuming from Kafka may run forever until a Control-C interrupt.

## Functional Requirements

(40 points total)

1) Implement a command line program to add new RSS feeds to the schedule.  An example list of RSS feeds is provided.  It should compare its inputs to the schedule and not add items to the schedule which have been scheduled with the same parameters before.

2) Interpret an RSS feed on the schedule as as a request to retrieve its items and add the items to the schedule.  Note that the items of an RSS feed are usually not RSS feeds.

3) Interpret a non-feed on the schedule as a request to retrieve it with an HTTP GET and save it in a directory on the local filesystem.

4) Add data to topics to indicate the completion of work.

5) Provide a parameter called "timeToLive" to requests in the work schedule.  While we say "time", in this context we are not talking about a clock time but a logical time.  This use of the phrase "time to live" is used many places in computing, for example in the specifications for the internet (TCP/IP).

Our definition:

<b>timeToLive</b>: A whole number indicating how many degrees of items will be fetched.  When consuming a work item with a timeToLive of N, write retrieved items to the schedule with a time to live of N-1.

| value | Interpretation |
|---|---|
| timeToLive &gt; 1 | Retrieve resource and find linked resources inside |
| timeToLive = 1 | Retrieve resource and store in filesystem, but do not process resource to find linked resources |
| timeToLive &lt;= 0 | Requests with timeToLive &lt;=0 should not be produced.  If consumed, ignore such requests |

The default timeToLive for scheduling an RSS feed as in 1) should be N=2.  Thus the feeds items will have N=1, and finding sub-items in HTML (as opposed to RSS feed) resources will not be necessary.

For testing, make sure you can add an RSS feed to the schedule with timeToLive=1

6) If retrieval of an item fails, mark it on the schedule as permanently failed, but wait 3 seconds before moving on.  This is a simpler behavior than trying to retry.  See E2 for an extra credit opportunity regarding retry behavior.

## Sample code and Resources

| Sample Code | file name |
|---|---|
| A list of RSS feeds | data-acquisition/rss_feeds.txt |
| Kafka consumer | kafka_python/hello-kafka-consumer.py |
| Kafka producer | kafka_python/hello-kafka-producer.py |
| Kafka python README | [kafka_python/README.md](../../../uwbd-examples/blob/master/kafka_python/README.md) |

All resources are in the [uwbd-examples](https://gitlab0.bigdata220uw.mooo.com/jhenri/uwbd-examples/tree/master) repository.

### External resources

|   | Hyperlink |
|---|---|
| json in python | [json — JSON encoder and decoder](https://docs.python.org/2/library/json.html) |
| xpath in python | [stack overflow](http://stackoverflow.com/questions/8692/how-to-use-xpath-in-python) |

## FAQ

Q: Will this assignment involve peer review?

A: In the interest of time, peer review is not scheduled for this assignment.

Q: What is RSS?

A:
- https://en.wikipedia.org/wiki/RSS
- http://www.npr.org/help/rss.html


Q: how did you scrape the rss links from cnn?

A: Chrome:

    function uniq(a) {
        var seen = {};
        return a.filter(function(item) {
            return seen.hasOwnProperty(item) ? false : (seen[item] = true);
        });
    }

    uniq($x("//a").filter(a => a.href.endsWith(".rss") && (a.href.indexOf("?")<0)).map(a => a.href)).join("\n")


## Extra Credit

Extra credit points will count toward your own score, but will not count toward calibrating the pass/fail curve.

E1) (20 points) Provide a way to perform only a certain fraction of the work on a given worker.  Use md5 to convert URLs to a number, and perform work only when a certain range of MD5 sums result.

This is useful for testing, because it allows the effective work load to be smaller and thus complete faster.  It is also useful for scaling, because it allows a large collection of workers to participate.

E2) (10 points) If retrieval of an item fails, try to distinguish between whether the failure was permanent (e.g. HTTP status 404 not found) or temporary (e.g. TCP error could not reach host).  Mark permanent failures in the schedule as permanently failed so they will not be retried.  Mark temporary failures on the queue as attempted.  For temporarily failed attempts, include a time after which a retry will be allowed.  Retry up to 3 times for each event, and all the retries occur, refrain from future retries.

More extra credit items TBA

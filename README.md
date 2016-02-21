# Assignment2: Data acquisition with kafka

**Author: Wenxiao Jeremy Gu**

**Modified from classnotes**

BigData2 (Winter 2016), University of Washington, Downtown Seattle campus

# Part 1 Setup

## git clone 

## Run docker

Refer to DockerTutorial.md


### Window 1: at UWBD-examples directory
```
    docker-machine start vm0
    ## docker-machine regenerate-certs vm0
    docker-machine env vm0
    eval $(docker-machine env vm0)
    ## tcp://192.168.99.101:2376
    
    
    export DOCKER_IP=192.168.99.101
    bash uwbd-util/tools/restart-kafka-in-docker.sh
  
    docker run -it -v $(pwd):/$(basename $(pwd)) --net=host --name=python1 jlynn/python-virtualenv bash
    
    ## going to docker
	## Kafka code
	export KAFKA_HOME="/uwbd-examples/kafka"
    ## export KAFKA_HOME="/Users/wenxiaogu/Dropbox/2-Bigdata/bigdata220/big_summary_2016/BigData2/0203_kafka_python/kafka"
    ## test 
    ls $KAFKA_HOME -alh
	## create a topic called topicNew
	/venv/bin/python /uwbd-examples/kafka_python/hello-kafka-ensure_topic_exists.py
	
	
	eval $DOCKER_IP
```

### Window 2: Any directory
```
    ## After running window 1, connect your docker container via ssh
    docker-machine ssh vm0
    
    ## install bash
    tce-load -wi bash.tcz
```

### If Window 1 crashes, we want to open Window 3 and want to connect to the container
```
	docker-machine ssh vm0
	docker ps -a
	docker start <container name> ## if the container is already stopped.
	docker attach <container name>
```


### Recent changes

- Added example code: <code>data-acquisition/exampleDataAcqMain.py</code>

- Added relative directory <code>../kafka_python</code> to scripts located there so all could be run from a current directory of <code>uwbd-examples/data-acquisition</code>.

- Incorporated into this page all examples on ../kafka_python/README.md essential to the Data Acquisition assignment.

- Added kafka-consume-and-produce-from-the-same-program.py

- References to "python" and "pip" updated to presume a virtualenv at /venv, as described at [../data-acquisition/README.md](https://gitlab0.bigdata220uw.mooo.com/jhenri/uwbd-examples/blob/master/data-acquisition/README.md).

- hello-kafka-consumer instructions clarified


### See also

The [../kafka_python/README.md](https://gitlab0.bigdata220uw.mooo.com/jhenri/uwbd-examples/blob/master/kafka_python/README.md) is available for reference.  The essential elements have been added to this page.

## Obtain the code

    pushd [dir]/uwbd-examples

dir: where you put git clones

    git pull
    git submodule init
    git submodule update

## Run docker containers

The following method of running a docker container is different from the one shown in class.  The one in class was just one way to get through things, and also a way to explain some perils of msys.  The method following is recommended for the following ergonomic advantages:

- You do not need to apply "docker-machine env"
- You can find your file sharing directory with the aid of tab completion.  Thus problems with your docker-machine sharing are more likely to surface before <code>docker run</code>.
- On windows, the perils of the msys shell (the ones fought with in class) are entirely side stepped

Connect to your docker container via ssh:

    docker-machine ssh vm0

The embedded linux distribution run by docker-machine does not include the bash shell, which is required.  To install bash:

    tce-load -wi bash.tcz

Change to the directory with your git-tracked source.  Pressing tab is your friend.

    cd /. . . ./uwbd-examples

#### Run containers for kafka:

Change to the directory with your git-tracked source.  Pressing tab is your friend.

    cd /. . . ./uwbd-examples

    export DOCKER_IP=<ipForVm>

The <code>&lt;ipForVm&gt;</code> is the IP of your docker-machine vm.  Use <code>docker-machine ip &lt;yourvmname&gt;</code> in another window if needed.

    bash uwbd-util/tools/restart-kafka-in-docker.sh

<b>Note if you get an error with "pushd", check the tce-load command above.</b>

<b>Note: if you are missing the uwbd-util directory of uwbd-examples, check the two "git submodule" commands above</b>

#### Run a container for your application:

Change to the directory with your git-tracked source.  Pressing tab is your friend.

    cd /. . . ./uwbd-examples

Run a container based on an ubuntu python image:

    docker run -it -v $(pwd):/$(basename $(pwd)) --net=host --name=python1 jlynn/python-virtualenv bash
    cd /uwbd-examples/data-acquisition

The reason the container has a <code>/uwbd-examples</code> is that <code>$(basename $(pwd))</code> evaluated to <code>uwbd-examples</code>.

The flag --net=host is a simplified network configuration that links all ports to your docker-machine VM's ports at the same location.  The containers for kafka and zookeeper created by restart-kafka-in-docker.sh have --net=host.

### An important context change just happened

Just for the record (no action required) if you are following these instructions top to bottom:

Note that by doing a docker-run, you have left the environment where your docker-machine env variables are set.  The instructions will continue from here presuming that we are only going to need our application container terminal.  In an ideal circumstance this will be true.  If you need to run docker commands from here on out, you have three choices:

1) Get yourself a new terminal window and run docker-machine ssh vm0.
2) Exit your application container by typing exit at the bash prompt to take you back to your docker-machine VM.
3) Rerun your application container with the docker socket and certificates mapped, and set your docker environment variables, as shown in [How to docker build](https://gitlab0.bigdata220uw.mooo.com/jhenri/uwbd-instructions/blob/master/How%20to%20docker%20build.md).  Then you will be able to issue commands to your parent docker engine without exiting your docker container.  

None of these actions are required if you do not find a need to run docker commands.

## Install virtualenv

    pip install virtualenv

More info at: http://virtualenv.readthedocs.org/en/latest/installation.html

## Create a virtualenv

    virtualenv /venv

Linux python installations are not compatible with storage on a windows filesystem, so the .. is a way to escape to linux-backed filesystem.  Your local git-tracked source will be backed by windows at all times so you will not lose work on this account.

From now on refer to pip and python like so:

    /venv/bin/python
    /venv/bin/pip

## Obtaining your python dependencies

    cd /uwbd-examples/data-acquisition
    /venv/bin/python setup.py install

Our setup.py invokes installation of pip packages automatically:

    . . .
    setup(
        name = "uwbd-util",
        install_requires=[printd(i.strip()) for i in open("requirements.txt").readlines()],
        packages=find_packages(".")
    )

If instead you wanted to install our list of pip packages more manually, the command is:

    /venv/bin/pip install -r requirements.txt

### data acquisition sample code: download RSS xml

    cd /uwbd-examples/data-acquisition
    /venv/bin/python downloadRss.py

Expected output:

    Wrote data/hortonworks_com_blog_feed.rss

### Data acquisition sample code: parse RSS items from RSS (xml)

For this example, you will need native packages from ubuntu.  Run:

    cd /uwbd-examples/data-acquisition
    bash get-apt-requirements.sh

The contents of this file is very simple:

    # Put your OS-distributed requirements in a file like this one.

    sudo apt-get update
    sudo apt-get install -y libxml2 libxml2-dev libxslt1-dev

Python lxml was removed from requirements.txt to allow the first example urllib3 to run without "apt-get install."  So now we have to install lxml:

    /venv/bin/pip install lxml

And now we can run the example:

    /venv/bin/python findRssItems.py

This will parse <code>data/hortonworks_com_blog_feed.rss</code> written by the previous example code.

Excerpt of expected output:

    found item
    found link: http://hortonworks.com/blog/leaving-st-valentine-behind-for-mr-roboto/
    found item
    found link: http://hortonworks.com/blog/join-hortonworks-3rd-annual-insurance-analytics-usa-summit-2016/
    found item
    found link: http://hortonworks.com/blog/how-pioneering-banks-adopt-hadoop-for-enterprise-data-management/
    found item
    . . .

### Data acquisition sample code: object mapping with jsonstruct

Thanks to requirements.txt you do not have to install jsonstruct manually.

Example workJson.py turns each line of json into a python value and back to a line of json:

    cd /uwbd-examples/data-acquisition
    cat workJsonData.json | venv/bin/python workJson.py

Result:

    got {"isa": "ItemRequested", "url": "http://rss.cnn.com/rss/cnn_latest.rss", "offset": "1"}
    parsed {"url": "http://rss.cnn.com/rss/cnn_latest.rss", "offset": "1"}
    got {"isa": "ItemAttempt", "url": "http://rss.cnn.com/rss/cnn_latest.rss", "timeExpiry": "2016-02-10 23:00:00"}
    parsed {"url": "http://rss.cnn.com/rss/cnn_latest.rss", "timeExpiry": "2016-02-10 23:00:00"}
    got {"isa": "ItemCompleted", "url": "http://rss.cnn.com/rss/cnn_latest.rss", "timeCompletion": "2016-02-10 23:00:00"}
    parsed {"url": "http://rss.cnn.com/rss/cnn_latest.rss"}
    got {"isa": "ItemFailed", "url": "http://rss.cnn.com/rss/cnn_latest.rss", "timeCompletion": "2016-02-10 23:00:00"}
    parsed {"url": "http://rss.cnn.com/rss/cnn_latest.rss"}

Note that these python values and classes are just a start at implementing the Data Acquisition assignment.  The classes like ItemRequested etc will probably have to be modified to complete the assignment.

More information on jsonstruct can be found here:

    https://github.com/initialxy/jsonstruct

Object mapping is not required for the assignment.  It is just one way.  If you are more comfortable, you are welcome to manipulate json directly like this part of workJsonData.py:

    if 'isa' not in j:
        return None
    isa = j['isa']
    if(isa == 'ItemRequested'):

More information about the json api can be found here:

    https://docs.python.org/2/library/json.html

## Running the kafka example code

To avoid steps inessential to the data acquisition assignment, here is a recap of the essential kafka_python examples.  The original kafka demo relied on download of kafka_2.10-0.8.2.2.tgz, and some shell scripts within.  To ease synthesis of the examples into a submission for the data cleaning assignment, these examples are considered inessential.  Only the python examples are shown here.

### Required environment

The sample code locates kafka using this application-specific environment variable:

    export DOCKER_IP=<ipForVm>

The <code>&lt;ipForVm&gt;</code> is the IP of your docker-machine vm.  Use <code>docker-machine ip &lt;yourvmname&gt;</code> in another window if needed.

### Create Kafka topic

The way that the kafka-python connector allows creation of a topic with a method called ensure_topic_exists.  In other words, create if it has not been created.

    /venv/bin/python ../kafka_python/hello-kafka-ensure_topic_exists.py

Excerpt:

    client = KafkaClient(['{}:9092'.format(docker_ip)])
    client.ensure_topic_exists("topic2")

### Produce Kafka from python:

    echo 'a message' | /venv/bin/python ../kafka_python/hello-kafka-producer.py

Excerpt:

    client = KafkaClient(['{}:9092'.format(docker_ip)])
    producer = SimpleProducer(client)
    . . .
    producer.send_messages('topicNew', line)

### Consume (aka receive, aka subscribe) from Kafka topic

Note the consumer has to be used together with the producer to see anything happen.

    /venv/bin/python ../kafka_python/hello-kafka-consumer.py &     # Starts the consumer printing in the background and returns a prompt
    echo 'a message' | /venv/bin/python ../kafka_python/hello-kafka-producer.py


### Consume and produce from the same program

Like hello-kafka-consumer.py, this example consumes messages and prints them on the console.  Start as follows:

    /venv/bin/python ../kafka_python/kafka-consume-and-produce-from-the-same-program.py &

(The &amp; puts the job in the background.  Look up "ps", "kill" and "fg" if you are not familiar.)

While no messages are inserted, every 5 seconds the program will send itself a message to print.  The console will show:

    consumed OffsetAndMessage(offset=58, message=Message(magic=0, attributes=0, key=None, value='No messages found in the past 5 seconds'))

Whenever we send a different kind of message:

    echo 'a message' | /venv/bin/python ../kafka_python/hello-kafka-producer.py

we will see it pass through the message:

    consumed OffsetAndMessage(offset=75, message=Message(magic=0, attributes=0, key=None, value='a message'))


### exampleDataAcqMain.py

This sample incorporates the elements of kafka-consume-and-produce-from-the-same-program.py and incorporates a few final ideas/hints for the assignment.  They are, in brief:

- Idea 1: Instead of using an object mapper like jsonstruct, we can simply consolidate our fields and tags in one place.

- Idea 2: Define a class to hold the current state of the world while your program reads the events going by.

- Idea 3: to represent the items that need work, the python built-in set() class is one simple but adequate implementation.

        /venv/bin/python exampleDataAcqMain.py  &

        echo '{"isa":"ItemRequested", "url":"http://yes.net/1"}' | /venv/bin/python ../kafka_python/hello-kafka-producer.py
        echo '{"isa":"ItemRequested", "url":"http://yes.net/2"}' | /venv/bin/python ../kafka_python/hello-kafka-producer.py
        echo '{"isa":"ItemRequested", "url":"http://yes.net/3"}' | /venv/bin/python ../kafka_python/hello-kafka-producer.py
        echo '{"isa":"ItemCompleted", "url":"http://yes.net/2"}' | /venv/bin/python ../kafka_python/hello-kafka-producer.py



## Adding a python dependency

If you have run the lxml example, then by adding pip's lxml, your requirements is now out of date.  Get it up to date like this:

    cd /uwbd-examples/data-acquisition
    /venv/bin/pip freeze > requirements.txt

Make sure to keep your requirements.txt file up to date and checked in.

## Amending requirements.txt

For example, to add the kafka-python connector:

    cd /uwbd-examples/data-acquisition
    /venv/bin/pip install kafka-python && /venv/bin/pip freeze > requirements.txt

Make sure to keep requirements.txt checked in and up to date.

## How to pick dependencies

Pip packages all have a page on pipy.  For example the page for urllib3:

    https://pypi.python.org/pypi/urllib3

Factors to consider when choosing a dependency:

- Beware of very small download counts.
- Check the API.  Does it make sense?
- Does the package have sample code?  Does the sample code run?
- Beware of too many transitive dependencies.
- Beware of dependencies on known bad packages.

# Distributed RSS
Distributed system for **reading RSS/Atom feeds**. The system reads feeds, parses them and saves new entries into database. It also pulls the full content of the entry into database. The system is *horizontally scalable* (workers and multiple threads per worker) and *resiliant to partial outages* (using message broker).

## Purpose
This project was done for a challenge which was organised by [Zemanta](http://www.zemanta.com) and [Faculty of Computer and Information Science](http://www.fri.uni-lj.si/en/), [University of Ljubljana](http://www.uni-lj.si/eng/). More about this challenge on [official Zemanta page](http://www.zemanta.com/blog/zemantas-programming-challenge-2014-zemantin-programerski-izziv-2014/) and on [faculty page](http://www.fri.uni-lj.si/si/raziskave/studentski_izzivi/zemantin_izziv/).

## General Requirements
This solution **requires** the following systems:
* [Java SE Runtime Environment 7](http://www.oracle.com/technetwork/java/javase/downloads/jre7-downloads-1880261.html)
 The logic of the system is written in JAVA programming language.
* [MongoDB](http://www.mongodb.org/)
 The MongoDB database for storing feeds and entries. **Version 2.4.9**
* [Apache ActiveMQ](http://activemq.apache.org/)
 Message broker for distributing workload. **Version 5.9.1**

### Required libraries
Project uses Maven to define dependencies on third party libraries. Nonetheless here is the list of required libraries:

* [ROME](http://rometools.github.io/rome/)
Library for RSS/Atom in Java.
* [Mongo Java Driver](http://docs.mongodb.org/ecosystem/drivers/java/)
Java driver for MongoDB.
* [ActiveMQ](http://activemq.apache.org/)
Java driver for ActiveMQ message broker.
* [JUnit](http://junit.org/)
Library for Unit testing.
* [Joda-Time](http://www.joda.org/joda-time/)
Library for Java DateTime.
* [HttpComponents](http://hc.apache.org/httpcomponents-client-ga/)
Library for efficent HTTP support in Java.
* [Log4J](http://logging.apache.org/log4j/2.x/)
Logging framework.
* [Commons Codec](http://commons.apache.org/proper/commons-codec/)
Support for efficent encoders (e.g. SHA-1 digest util).
* [Commons CLI](http://commons.apache.org/proper/commons-cli/)
API for parsing command line options.

## General solution
The general solution consists of **three** JAR files:
* *InsertResources*: For inserting RSS feeds from CSV file into MongoDB. The CSV file consists of URLs of feeds.
* *RSSDelegateWorker*: For inserting jobs (feeds) into message queue and checking for stalled jobs.
* *RSSMainWorker*: For running thread workers which fetch entries of feeds, fetch the web page and persist it to the MongoDB. The main worker deques the job from message queue and allocates a new thread from thread pool for each feed. The thread worker then does the rest of the job.

### MongoDB schema
TODO

### Running
A quick tutorial for running the solution.

1. First run the InsertResources jar:
   ```java
   java -jar InsertResources.jar
   ```

   The program accepts the following arguments:
   ```
   usage: java -jar InsertResources
    -collName <arg>   the name of collection to use
    -dbName <arg>     the name of the database to use
    -filePath <arg>   the path of the file with RSS feeds
    -help             help for usage
    -host <arg>       database's host address
    -port <arg>       port on which the database is running
   ```
   
   If the user does not pass any arguments then the following default values are used:
   ```
   collName = "feeds"
   dbName = "rssdb"
   filePath = "./10K-RSS-feeds.csv"
   host = "localhost"
   port = 27017
   ```

2. Then run RSSDelegateWorker jar:
   ```java
   java -jar RSSDelegateWorker
   ```

   The program accepts the following arguments:
   ```
   usage: java -jar RSSDelegateWorker
    -checkInterval <arg>   time in seconds for checking stalled feeds
    -collName <arg>        the name of collection to use
    -dbName <arg>          the name of the database to use
    -help                  help for usage
    -hostBroker <arg>      the URL of the broker
    -hostDB <arg>          database's host address
    -portDB <arg>          port on which the database is running
    -subject <arg>         name of the queue
   ```
   
   If the user does not pass any arguments then the following default values are used:
   ```
   checkInterval = 24 * 60 * 60
   collName = "feeds"
   dbName = "rssdb"
   hostBroker = "failover://tcp://localhost:61616"
   hostDB = "localhost"
   port = 27017
   subject = "RSSFEEDSQUEUE"
   ```

3. And finally the main worker RSSMainWorker jar:
   ```java
   java -jar RSSMainWorker
   ```

   The program accepts the following arguments:
   ```
   usage: java -jar RSSMainWorker
    -collNameEntries <arg>   the name of collection to use for entries
    -collNameFeeds <arg>     the name of collection to use for feeds
    -dbName <arg>            the name of the database to use
    -help                    help for usage
    -hostBroker <arg>        the URL of the broker
    -hostDB <arg>            database's host address
    -portDB <arg>            port on which the database is running
    -subject <arg>           name of the queue
    -threadsNum <arg>        number of active threads
   ```
   
   If the user does not pass any arguments then the following default values are used:
   ```
   collNameEntries = "entries"
   collNameFeeds = "feeds"
   dbName = "rssdb"
   hostBroker = "failover://tcp://localhost:61616"
   hostDB = "localhost"
   portDB = 27017
   subject = "RSSFEEDSQUEUE"
   threadsNum = 10
   ```
   
   Of course one can run **multiple main workers**.
   
## TODO
* Implement check for simmilarity between id's of entries of given feed using Levensthein distance.
* If similarity between id's is not found then also check for similarity between full page content using Jaccard distance.


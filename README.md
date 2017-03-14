# Sentiment Analysis on Twitter Wells Fargo topic

## Tools and libraries

* Java
	* Twitter4J - library to interface with Twitter API for batch and streaming ingestion of tweets
	* Stanford NLP - library for text analytics, sentiment analysis
	* Eclipse - IDE environment for coding in Java
	* Maven - build tool to compile and package our Java source code
* Cloudera Quickstart VM
	* Hadoop / HDFS - where we store our ingested tweets
	* Flume - tool in Hadoop stack used for log streaming; used together with Twitter4J
	* Hive - tool in Hadoop stack to impose a structure / schema on our ingested data as a database table
	* Impala - fast SQL on Hadoop where we will connect our visualization
* Tableau - for displaying visualization

### Step 1: Setup Cloudera Quickstart VM

I set up a Cloudera Quickstart VM on my local machine.

### Step 2: Install Maven and Eclipse create a Java project

I created the twitter-ingest-flume-java project in Eclipse and added the following dependencies  in Maven's pom.xml:
	* Twitter4J
	* Flume
	* Stanford NLP
	
	><dependency>
  	>	<groupId>org.twitter4j</groupId>
  	>	<artifactId>twitter4j-core</artifactId>
  	>	<version>4.0.3</version>
  	></dependency>
  	><dependency>
  	>	<groupId>org.twitter4j</groupId>
  	>	<artifactId>twitter4j-stream</artifactId>
  	>	<version>4.0.3</version>
  	></dependency>
  	><dependency>
  	>	<groupId>org.apache.flume</groupId>
  	>	<artifactId>flume-ng-core</artifactId>
  	>	<version>1.5.2</version>
  	></dependency>
	><dependency>
	>	<groupId>edu.stanford.nlp</groupId>
	>	<artifactId>stanford-corenlp</artifactId>
	>	<version>3.6.0</version>
	></dependency>

### Step 3: 
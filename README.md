# Sentiment Analysis on Twitter Wells Fargo topic

## SUMMARY:
Wells Fargo went through a very publicized scandal that started gaining traction sometime in the fall of 2016. The CEO had to answer questions during a congressional probe and the overall outlook of the company was very bad on all fronts. A series of malpractices had been going on with the knowledge of the upper level management. The stock tanked, and all news concerning the bank was outright bad. Given this, one would think the bank would take a while to recover. This sentimental analysis was done in early March 2017. The number of tweets assessed were 2,502. The total users were 1,647. There were a total of 606 negative tweets and 1055 positive tweets based on certain word choices used when referring to "Wells Fargo". The neutral tweets were 841. This is a sample of only a week and for what it's worth this is a remarkable recovery given the seriousness of the issues the company had to deal with - and still are in many ways. The sum total is that though the scandal was damaging it has not had a long term effect on the outlook and future of Wells Fargo. The analysis says people will complain about practices of their banks but will not necessarily be easily moved as long as they provide the services they require. This is still a subjective analysis given the brevity of the research parameters or timeframe.

## Methodology

### Tools and libraries

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

I set up a Cloudera Quickstart VM 5.8 on my local machine.

### Step 2: Install Maven and Eclipse create a Java project

I created the `twitter-ingest-flume-java` project in Eclipse and added the following dependencies in Maven's pom.xml:
	
* Twitter4J
* Flume
* Stanford NLP
	
	```
	<dependency>
  		<groupId>org.twitter4j</groupId>
  		<artifactId>twitter4j-core</artifactId>
  		<version>4.0.3</version>
  	</dependency>
  	<dependency>
  		<groupId>org.twitter4j</groupId>
  		<artifactId>twitter4j-stream</artifactId>
  		<version>4.0.3</version>
  	</dependency>
  	<dependency>
  		<groupId>org.apache.flume</groupId>
  		<artifactId>flume-ng-core</artifactId>
  		<version>1.6.0</version>
  	</dependency>
	<dependency>
		<groupId>edu.stanford.nlp</groupId>
		<artifactId>stanford-corenlp</artifactId>
		<version>3.6.0</version>
	</dependency>
	```
	
### Step 3: Code and test

In addition to interfacing with Twitter, I embedded a sentiment analysis processing as the tweets are being ingested, evaluating it as "Positive", "Negative" or "Neutral". I used Stanford NLP for the task.

### Step 4: Package and create jar file and deploy

I compiled and packaged the project using Maven. The jar file created was later deployed to /usr/lib/flume-ng/lib. This is needed by Flume to interface with Twitter4J.

Flume has Twitter4J bundled in its library (also /usr/lib/flume-ng/lib) already: but we replaced it with more recent version 4.0.3. (We obtain the jars from Maven repo folder)

### Step 5: Configure and run Flume agent

There are two ways of ingesting data from Twitter: batch using Search API call or streaming using Streaming API. The Search API allows tweets to be pulled from Twitter up to 7 days ago. The Streaming API enables live-streaming of tweets (but only sampling ans restricted to filtered topics or hashtags, not the entire firehose).

I configured Flume to use either one of those. I only selected a few fields / columns as I do not want to store the entire JSON data of each tweet (`csv-limited` in configuration). Here is a snippet of Flume config for Search API (batch):

	batchTwitter.sources.twitter.type = com.whatever.TwitterSearchSource
	batchTwitter.sources.twitter.channels = tchannel
	batchTwitter.sources.twitter.consumerKey = <twitter-consumer-key>
	batchTwitter.sources.twitter.consumerSecret = <twitter-consumer-secret>
	batchTwitter.sources.twitter.accessToken = <twitter-access-token>
	batchTwitter.sources.twitter.accessTokenSecret = <twitter-access-token-secret>
	batchTwitter.sources.twitter.keywords = @wellsfargo, #wellsfargo
	batchTwitter.sources.twitter.writeMode = csv-limited
	
Template of configuration is located in `flume-configs`.

A sample of tweets we used is located in `data` folder.
	
### Step 6: Create a Hive table based on ingested data and expose this table to Impala

Based on the fields I selected and on the structure of ingested tab-delimited data by Flume, I impose a Hive schema on top.

	CREATE EXTERNAL TABLE IF NOT EXISTS tweets (
		tweet_id STRING,
		created_at STRING,
		user_name STRING,
		nick_name STRING,
		user_desc STRING,
		user_location STRING,
		tweet_content STRING,
		favorites BIGINT,
		retweets BIGINT,
		latitude DOUBLE,
		longitude DOUBLE,
		place_name STRING,
		place_full_name STRING,
		place_country STRING,
		place_country_code STRING,
		sentiment STRING
	)
	ROW FORMAT SERDE 'com.bizo.hive.serde.csv.CSVSerde'
	WITH SERDEPROPERTIES("escapeChar" = "\\", "quoteChar" = "\"")
	STORED AS TEXTFILE
	LOCATION '/tmp/tweets';
	
We assume that the same location is where Flume is configured to dump data.

According to the tutorial which I used as guide, if the tweets are ingested as raw JSON, JSONSerde is required to project a table-like structure in Hive and we use more complex data types in table definition.

Then I used `INVALIDATE METADATA` in order for this table to be available to Impala also. If Hive is used where Tableau will connect, it will be very slow.

### Step 7: Install and configure ODBC Connectors for Hive and Impala

I downloaded these from Cloudera website. This is needed to connect Tableau to our table.

### Step 8: Connect Tableau 

Then I connected Tableau to Impala on port 21000 and created our visualization.
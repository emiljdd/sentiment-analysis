# Sentiment Analysis on Twitter Wells Fargo topic

## Tools and libraries

* Java
	* Twitter4J - library to interface with Twitter API for batch and streaming ingestion of tweets
	* Stanford NLP - library for text analytics, sentiment analysis
* Cloudera Quickstart VM
	* Hadoop / HDFS - where we store our ingested tweets
	* Flume - tool in Hadoop stack used for log streaming; used together with Twitter4J
* Tableau - for displaying visualization
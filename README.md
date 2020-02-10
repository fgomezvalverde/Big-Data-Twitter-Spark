# Streaming Processing using Twitter - Spark Streaming - Kafka

This project is for begginer in the Big Data , you will find a very simple project for testing porpuses.
Let looks what we are going to talk about:

1. Objective of the project.
2. Architecture
3. Sofware Installation
4. Implementation
5. Results
6. Conclusions

All the source code is in the next URL: https://github.com/fgomezvalverde/BigDataCourse/tree/master/Twitter%20-%20Data%20Streaming

## 1.Objective of the project
Create a streaming processing data stack using Twitter as a datasource. Get know of the use of tecnologies like: Kafka, Zookeeper, Streaming, Brockers and others. And have a small view of what can be done using this tecnologies in real life scenarios.

## 2. Architecture
<img src="Twitter - Data Streaming/Diagram.png">

## 3.Sofware Installation

1. Installing Zookepeer
https://medium.com/@shaaslam/installing-apache-zookeeper-on-windows-45eda303e835

This a beautiful explanation of what is Zookepeer for Kafka: https://www.cloudkarafka.com/blog/2018-07-04-cloudkarafka_what_is_zookeeper.html

2. Installing Kafka
https://medium.com/@shaaslam/installing-apache-kafka-on-windows-495f6f2fd3c8

3. Test Zooker and kafka. Making a producer and consumer test from Command Line.
https://medium.com/@shaaslam/installing-apache-kafka-on-windows-495f6f2fd3c8 ***This one is an exellente post, just have one issue the consumer should listen to the broker, and not to zookeeper: localhost:9092***

Theres other considerations that you need to have on mind, like; installed JRE, 7-ZIP for tar unziping, Python, Conda with PySpark, the spark streaming kafka assembly. Theres is lots of resources out there to do this things.

## 4.Implementation


Afther the installation of Zookepeer and Kafka, and did a very simple example of the Producer - Consumer in the console. We are ready to start for a more real case scenario, like Twitter as Producer and a Spark Streaming Context as Consumer. There is two approches for the Spark Kafka Streaming, for this example we will use Reciever-based. For more information you can follow this linK: https://spark.apache.org/docs/latest/streaming-kafka-0-8-integration.html

Another thing, I created a topic called **raw-twitter** and parting from that you already have this. You did this already in the Producer - Consumer test from Command with **sql-insert** topic.

### 4.1.Producer

The consumer contains 6 Subject:
- Imports
- Configuration
- Authentication Method
- Get Tweets URL Constructions
- Publish Message to Kafka
- KeepAlive Process

We will focus on the Configuration and the Publish Message to Kafka sections. Still take a look to the others methods in https://github.com/fgomezvalverde/BigDataCourse/blob/master/Twitter%20-%20Data%20Streaming/Producer.ipynb.


#### Configuration
```
consumerKey="XXXXXXXXXXXXXXXXXX"
consumerSecret="XXXXXXXXXXXXXXXXXX"
accessToken="XXXXXXXXXXXXXXXXXX"
accessTokenSecret="XXXXXXXXXXXXXXXXXX"
keyword= "superbowl"
start_date = "2020-01-01" 
end_date = "2020-03-03"
req_count = 0
min_faves=60000
change=10000 
interval = 500
lang="en"

kafka_url="localhost:9092"
topic_name= "raw-twitter"
```
**The Twitter configuration** is mostly API KEYS, the other part is the hashtag that be looking "superbowl", in the specific dates with a very high Faves on it. Soo would find posts like from Shakira. 

For **kafka_url**, Im using the same computer for my kafka server and the producer soo localhost is the right for me , but is you are using another computer or a cloud server you need to change this variable.

And **topic_name**, like I said before Im using **raw-twitter** as topic, you need to changed this to your created topic on kafka.

#### Publish Message to Kafka


```
def connect_kafka_producer():
    producer = None
    try:
        producer = KafkaProducer(bootstrap_servers=[kafka_url], api_version=(0, 10))
    except Exception as ex:
        print('Exception while connecting Kafka')
        print(str(ex))
    finally:
        return producer
```
Using the kafka_url we create the Kafka Producer instance , very basic by the way. Is possible that theres more configuration variables . Take a look the KakfaProducer API.

```
def publish_message(producer_instance, key, value):
    try:
        key_bytes = bytes(key, encoding='utf-8')
        value_bytes = bytes(value, encoding='utf-8')
        producer_instance.send(topic_name, key=key_bytes, value=value_bytes)
        producer_instance.flush()
        print('Message published successfully.')
    except Exception as ex:
        print('Exception in publishing message')
        print(str(ex))
```
This one is in charge of the publish of the message using the Kafka Producer and the **Topic name** setter on the configuration section.


### 4.2.Consumer

The consumer contains 3 Subject:
- Imports
- Configuration
- KeepAlive Process

We will focus on the Configuration and the KeepAlive Process sections. Still take a look to the others methods in https://github.com/fgomezvalverde/BigDataCourse/blob/master/Twitter%20-%20Data%20Streaming/Consumer.ipynb.

#### Configuration

```
zookeeper_url="localhost:2181"
topic_name= "raw-twitter"
```
The consumer need to point to the Zookepeer URL. In my case ***localhost:2181*** . And the topic name is the same that we define before ***raw-twitter***.

```
os.environ['PYSPARK_SUBMIT_ARGS'] = '--jars spark-streaming-kafka-0-8-assembly_2.11-2.4.4.jar pyspark-shell' 
sc = SparkContext(appName="PythonStreamingRecieverKafkaTwitter")
sc.setLogLevel("ERROR")
ssc = StreamingContext(sc, 10) # 10 second window
kvs = KafkaUtils.createStream(ssc,zookeeper_url,"raw",{topic_name:1}) 
lines = kvs.map(lambda x: x[1])
lines.pprint()
ssc.start()
ssc.awaitTermination(30)
ssc.stop()
```

For this part I recommend to this post first, is a very good explanation about the approch https://spark.apache.org/docs/1.4.1/streaming-kafka-integration.html. And also is important to know what is a DStream and how it works, I recommend to read this : https://stackoverflow.com/questions/36421619/whats-the-meaning-of-dstream-foreachrdd-function. 

Basicly we read every 10 seconds ***StreamingContext(sc, 10)***  any message from the topic  one by one ***{topic_name:1}***.

And finally print every message until an error or interruption.

## 5.Demostration

Take a look of the video of a test using the same computer the Producer and Consumer.

You can download the video here: https://github.com/fgomezvalverde/BigDataCourse/raw/master/Twitter%20-%20Data%20Streaming/Localhost%20Demo.mp4

## 6.Conclusions

All the configuration set here was for tests purposes. Using the same computer for Producer and Consumer is not a Production scenario. Also, we work with just 1 Brocker for the Kafka Server, depends of the volumen of the data, replications, disponibility, etc. We can found a better configutation dependes on the sceneario.

What steps can be next from here:
- Using a diferents computer for the Kafka Server, Producer and Consumer.
- Taking more data and use more brockers.
- Analize the twitters messages with sentiment analisys and maybe use it as a Y parameter for a machiner learning project.
- Save the data on a data base, using this stream Processing and maybe can see what to do with batch processing. Databases like Hadoop or Cassandra can be interesenting to integrate.

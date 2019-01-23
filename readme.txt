Install and fire up the Sandbox using the instructions here: http://maprdocs.mapr.com/home/SandboxHadoop/c_sandbox_overview.html. 

Use an SSH client such as Putty (Windows) or Terminal (Mac) to login. See below for an example:
use userid: user01 and password: mapr.

For VMWare use:  $ ssh user01@ipaddress 

For Virtualbox use:  $ ssh user01@127.0.0.1 -p 2222 

You can build this project with Maven using IDEs like Eclipse or NetBeans, and then copy the JAR file to your MapR Sandbox, or you can install Maven on your sandbox and build from the Linux command line, 
for more information on maven, eclipse or netbeans use google search. 

After building the project on your laptop, you can use scp to copy your JAR file from the project target folder to the MapR Sandbox:

use userid: user01 and password: mapr.
For VMWare use:  $ scp  nameoffile.jar  user01@ipaddress:/user/user01/. 

For Virtualbox use:  $ scp -P 2222 nameoffile.jar  user01@127.0.0.1:/user/user01/.  

Copy the data file from the project data folder to the sandbox using scp to this directory /user/user01/data/uber.csv on the sandbox:

For Virtualbox use:  $ scp -P 2222 data/uber.csv  user01@127.0.0.1:/user/user01/data/. 


read more here http://maprdocs.mapr.com/home/Spark/Spark_IntegrateMapRStreams.html

note use:  mapr-sparkml-streaming-uber-1.0-jar-with-dependencies.jar if you did not update the spark classpath

Step 0: Create the topics to read from (ubers) and write to (uberp)  in MapR streams:

maprcli stream create -path /user/user01/stream -produceperm p -consumeperm p -topicperm p
maprcli stream topic create -path /user/user01/stream -topic ubers  
 

to get info on the ubers topic :
maprcli stream topic info -path /user/user01/stream -topic ubers

to delete topics:
maprcli stream topic delete -path /user/user01/stream -topic ubers  



_________________________________________________________________________________  

 Step 1: 



run the Java producer to produce messages with the topic and data file arguments:

java -cp mapr-spark-streaming-hbase-1.0.jar:`mapr classpath` com.streamskafka.uber.MsgProducer /user/user01/stream:ubers /user/user01/data/cluster.txt

To run the MapR Streams Java consumer  with the topic to read from (uberp):

java -cp mapr-spark-streaming-hbase-1.0.jar:`mapr classpath` com.streamskafka.uber.MsgConsumer /user/user01/stream:ubers 

Then you can skip to Step 4 


____________________________________________________________________

Step 2:  Spark Streaming writing to MapR-DB HBase

Configure the HBase connector as documented here
http://maprdocs.mapr.com/home/Spark/ConfigureSparkHBaseConnector.html

start the hbase shell and create a table: 
hbase shell
create '/user/user01/db/uber', {NAME=>'data'}

Run the code to Read from MapR Streams and write to MapR-DB HBAse

spark-submit --class com.sparkkafka.uber.SparkKafkaConsumerWriteHBase --master local[2] \
mapr-sparkml-streaming-uber-1.0-jar-with-dependencies.jar /user/user01/stream:ubers  "/user/user01/db/uber"

start the hbase shell and scan to see results: 
hbase shell
scan '/user/user01/db/uber' , {'LIMIT' => 5}

____________________________________________________________________


Step 3:  Spark Reading from  MapR-DB HBase


Run the code to Read from  MapR-DB HBAse into a spark dataframe

spark-submit --class com.sparkkafka.uber.SparkHBaseReadDF --master local[2] \
mapr-sparkml-streaming-uber-1.0-jar-with-dependencies.jar "/user/user01/db/uber"

_________________________________________________________________________________

to delete the ubers topic after using :
maprcli stream topic delete -path /user/user01/stream -topic ubers

_________________________________________________________________________________






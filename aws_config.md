## Build file
mvn clean compile assembly:assembly -DdescriptorId=jar-with-dependencies

## Create cluster
aws emr create-cluster \
--name "My emr cluster" \
--release-label emr-5.34.0 \
--applications Name=Spark \
--ec2-attributes KeyName=emr2key \
--instance-type m5.xlarge \
--instance-count 3 \
--use-default-roles \
--configurations '[
  {
    "Classification": "yarn-site",
    "Properties": {
      "yarn.nodemanager.aux-services": "mapreduce_shuffle",
      "yarn.nodemanager.aux-services.mapreduce_shuffle.class":"org.apache.hadoop.mapred.ShuffleHandler"

    }
  }
]'


## Set config because giraph needs it
[
  {
    "Classification": "yarn-site",
    "Properties": {
      "yarn.nodemanager.aux-services": "mapreduce_shuffle",
      "yarn.nodemanager.aux-services.mapreduce_shuffle.class":"org.apache.hadoop.mapred.ShuffleHandler"

    }
  }
]

## Add and run step
aws emr add-steps --cluster-id j-3C5FIYN5G7OWM --steps '[{"Type":"CUSTOM_JAR","Name":"GiraphApp2","MainClass":"org.apache.giraph.GiraphRunner","ActionOnFailure":"CONTINUE","Jar":"file:///home/hadoop/book-examples-1.0.0-jar-with-dependencies.jar","Args":["SimpleShortestPathsComputation","-vip", "input/tiny_graph.txt","-vif", "org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat","-vof", "org.apache.giraph.io.formats.IdWithValueTextOutputFormat","-op", "output2", "-w", "1","-ca", "giraph.SplitMasterWorker=false"]}]'

## list clusters:
aws emr list-clusters

## terminate clusters:
aws emr terminate-clusters --cluster-id j-1AT5AD8F9DBAU

## put file
aws emr put --cluster-id j-3EW3FT8IQNNUT --key-pair-file ~/emr2key.pem --src target/SimpleShortestPath_2.10.1_hadoop-1.0.0-jar-with-dependencies.jar

## get state data
aws emr describe-step --cluster-id j-3EW3FT8IQNNUT --step-id s-3SQXBWYJKFF9J

## effeciently copy from s3 to hdfs

s3-dist-cp --src s3://lsc-bucket-v0/myfolder/outputfile.txt --dest hdfs:///user/hadoop/input
  

## timing info

cat /mnt/var/log/hadoop/steps/s-139PZGQGN88XN/syslog | sed -n -e '/Giraph Timers/, /Netty counters/ p'

## get timing info from cli
aws emr get --cluster-id j-3C5FIYN5G7OWM --key-pair-file ~/emr2key.pem --src /mnt/var/log/hadoop/steps/s-139PZGQGN88XN/syslog --dest 
 
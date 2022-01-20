## Build file
`mvn clean compile assembly:assembly -DdescriptorId=jar-with-dependencies`

## Create cluster

`mapred.tasktracker.map.tasks.maximum":4` !Can limit slaves
```
aws emr create-cluster \
--name "My emr cluster" \
--release-label emr-5.34.0 \
--ec2-attributes KeyName=emr2key \
--applications '[
  {
    "Name": "ZooKeeper"
  },
  {
    "Name": "HADOOP"
  }
]' \
--instance-groups \
InstanceGroupType=MASTER,InstanceCount=1,InstanceType=m5.xlarge \
InstanceGroupType=CORE,InstanceCount=2,InstanceType=m5.xlarge \
--use-default-roles \
--configurations '[
  {
    "Classification": "yarn-site",
    "Properties": {
      "yarn.nodemanager.aux-services": "mapreduce_shuffle",
      "yarn.nodemanager.aux-services.mapreduce_shuffle.class":"org.apache.hadoop.mapred.ShuffleHandler"

    }
  },
  {
    "Classification": "mapred-site",
    "Properties": {
      "mapreduce.job.counters.limit": "1000",
      "mapred.tasktracker.map.tasks.maximum":4
    }
  },
  {
    "Classification": "core-site",
    "Properties": {
      "io.file.buffer.size": "65536",
    }
  } 
]'
```
## I ran this but don't know if it's necesarry
`sudo /usr/lib/zookeeper/bin/zkServer.sh start`

## put file
`aws emr put --cluster-id j-3C5FIYN5G7OWM --key-pair-file ~/emr2key.pem --src target/SimpleShortestPath_2.10.1_hadoop-1.0.0-jar-with-dependencies.jar`
## ssh connection
`aws emr ssh --cluster-id j-NG59OU6NM7SV --key-pair-file ~/emr2key.pem`

## effeciently copy from s3 to hdfs
`s3-dist-cp --src s3://lsc-bucket-v0/myfolder/facebook_graph_json_1.txt --dest hdfs:///user/hadoop/input`

## Run command
ec2-54-86-176-6.compute-1.amazonaws.com <= public container ip
Workers <= slaves
`hadoop jar ~/book-examples-1.0.0-jar-with-dependencies.jar org.apache.giraph.GiraphRunner SimpleShortestPathsComputation -vip input/facebook_graph_json_1.txt -vif org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat -vof org.apache.giraph.io.formats.IdWithValueTextOutputFormat -op output15 -w 3 -ca giraph.zkList=ec2-54-86-176-6.compute-1.amazonaws.com:2181,mapred.job.tracker=ec2-54-86-176-6.compute-1.amazonaws.com:8088`


## timing info

cat /mnt/var/log/hadoop/steps/s-139PZGQGN88XN/syslog | sed -n -e '/Giraph Timers/, /Netty counters/ p'



## Add and run step
aws emr add-steps --cluster-id j-3C5FIYN5G7OWM --steps '[{"Type":"CUSTOM_JAR","Name":"GiraphApp2","MainClass":"org.apache.giraph.GiraphRunner","ActionOnFailure":"CONTINUE","Jar":"file:///home/hadoop/book-examples-1.0.0-jar-with-dependencies.jar","Args":["SimpleShortestPathsComputation","-vip", "input/tiny_graph.txt","-vif", "org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat","-vof", "org.apache.giraph.io.formats.IdWithValueTextOutputFormat","-op", "output2", "-w", "1","-ca", "giraph.SplitMasterWorker=false"]}]'

## list clusters:
aws emr list-clusters

## terminate clusters:
aws emr terminate-clusters --cluster-id j-1AT5AD8F9DBAU


## get state data
aws emr describe-step --cluster-id j-3EW3FT8IQNNUT --step-id s-3SQXBWYJKFF9J

  


## get timing info from cli
aws emr get --cluster-id j-3C5FIYN5G7OWM --key-pair-file ~/emr2key.pem --src /mnt/var/log/hadoop/steps/s-139PZGQGN88XN/syslog --dest 
 

## Run Page rank
aws emr add-steps --cluster-id j-3C5FIYN5G7OWM --steps '[{"Type":"CUSTOM_JAR","Name":"GiraphApp2","MainClass":"org.apache.giraph.GiraphRunner","ActionOnFailure":"CONTINUE","Jar":"file:///home/hadoop/book-examples-1.0.0-jar-with-dependencies.jar","Args":["SimplePageRankComputation","-vip", "input/facebook_graph_json_1.txt","-vif", "org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat","-vof", "org.apache.giraph.io.formats.IdWithValueTextOutputFormat","-op", "output3", "-w", "1","-ca", "giraph.SplitMasterWorker=false"]}]'

## Upload data step

aws emr add-steps --cluster-id j-2K0AQZMQXYTIP --steps '[{"Name":"S3DistCp step","Args":["s3-dist-cp","--src=s3://lsc-bucket-v0/myfolder/facebook_graph_json_1.txt","--dest=hdfs:///user/hadoop/input"],"ActionOnFailure":"CONTINUE","Type":"CUSTOM_JAR","Jar":"command-runner.jar"},{"Type":"CUSTOM_JAR","Name":"GiraphApp2","MainClass":"org.apache.giraph.GiraphRunner","ActionOnFailure":"CONTINUE","Jar":"file:///home/hadoop/book-examples-1.0.0-jar-with-dependencies.jar","Args":["SimpleShortestPathsComputation","-vip", "input/tiny_graph.txt","-vif", "org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat","-vof", "org.apache.giraph.io.formats.IdWithValueTextOutputFormat","-op", "output1", "-w", "1","-ca", "giraph.SplitMasterWorker=false"]}]' 



{"Type": "CUSTOM_JAR","Name": "UploadInputData","ActionOnFailure" : "CANCEL_AND_WAIT","Jar": "/home/hadoop/lib/emr-s3distcp-1.0.jar","Args": [ "--src", "s3://unique.bucket/datasets/1/","--dest","hdfs:///user/hadoop/input"]},


s-SXH27B2FJ5CA





j-JO8EFSDPGFSQ





hadoop jar book-examples-1.0.0-jar-with-dependencies.jar org.apache.giraph.GiraphRunner SimpleShortestPathsComputation -vip input/facebook_graph_json_1.txt -vif org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat -vof org.apache.giraph.io.formats.IdWithValueTextOutputFormat -op output1 -w 2 -ca giraph.SplitMasterWorker=false
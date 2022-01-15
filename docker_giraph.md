# Instruction to install giraph using docker
## Starting container

1. `docker pull izone/hadoop`

2. `docker run -dit izone/hadoop` # returns IMAGE_ID

3. `docker exec -it {IMAGE_ID} bash `
## Setup in your image terminal
### Add for apt-get to work
`mkdir -p /usr/share/man/man1`
### Install git and maven
`sudo apt-get update`
`sudo apt-get install -y maven git`


## Build
### clone
`cd /usr/local/`
`sudo git clone https://github.com/apache/giraph.git`

`cd giraph`
### select version
`sudo git checkout release-1.3`
### build
Dhadoop.version=*your hadoop version* for this docker it's 2.8.5
`mvn package clean package -DskipTests -Phadoop_2 -Dhadoop.version=2.8.5`
## Configure

`export GIRAPH_HOME=/usr/local/giraph`

`PATH=$GIRAPH_HOME/bin:$PATH`

`cp $GIRAPH_HOME/conf/giraph-site.xml $HADOOP_HOME/etc/hadoop/`

`export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop/`

Copy hadoop classpath value from:

`hadoop classpath`

Add giraph core and previously copied classpath value to env variable.

`export HADOOP_CLASSPATH=/usr/local/giraph/giraph-core/target/*:<hadoop_classpath_value>`

Now you can run giraph apps with 

`giraph ...` in place `hadoop -jar ...`

## Test giraph build
### Prepare data
`nano /tmp/tiny_graph.txt`

Paste graph data in format [source_id,source_value,[[dest_id, edge_value],...]] into tiny_graph.txt:

```
[0,0,[[1,1],[3,3]]]
[1,0,[[0,1],[2,2],[3,1]]]
[2,0,[[1,2],[4,4]]]
[3,0,[[0,3],[1,1],[4,4]]]
[4,0,[[3,4],[2,4]]]
```
### Put to hdfs
`hdfs dfs -mkdir /input`

`hdfs dfs -copyFromLocal /tmp/tiny_graph.txt /input`

`hdfs dfs -ls /input`

### Test
`giraph $GIRAPH_HOME/giraph-examples/target/giraph-examples-1.3.0-SNAPSHOT-for-hadoopPATH=$GIRAPH_HOME/bin:$PATH2.8.5-jar-with-dependencies.jar org.apache.giraph.examples.SimpleShortestPathsComputation -vif org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat -vip /input/tiny_graph.txt -vof org.apache.giraph.io.formats.IdWithValueTextOutputFormat -op /output/shortestpaths -w 1 -ca giraph.SplitMasterWorker=false`

### See results
`hdfs dfs -cat /output/shortestpath/part-m-00000`
## Create project
It can be any empty directory. It will be addressed as *project directory*
### add pom.xml in your *project directory*
Note that version of hadoop-client should match your hadoop version.
```xml
<?xml version="1.0" encoding="UTF-8"?>


<project>
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <modelVersion>4.0.0</modelVersion>
    <groupId>giraph</groupId>
    <artifactId>book-examples</artifactId>
    <version>1.0.0</version>
    <dependencies>
        <dependency>
            <groupId>org.apache.giraph</groupId>
            <artifactId>giraph-core</artifactId>
            <version>1.3.0-hadoop2</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-client -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.8.5</version>
        </dependency>
    </dependencies>
    <build>
    </build>
</project>
```
### java file
Copy from giraph-examples *SimpleShortestPathsComputation* to your *project directory*/src/main/java
`cp $GIRAPH_HOME/giraph-examples/src/main/java/org/apache/giraph/examples/SimpleShortestPathsComputation.java <project directory>/src/main/java/`

Remove lines from SimpleShortestPathsComputation.java in your *project directory*/src/main/java
```java
`package org.apache.giraph.examples;`
//[...]
@Algorithm(
    name = "Shortest paths",
    description = "Finds all shortest paths from a selected vertex"
)
```
your *project directory* file structure should look like this:
* pom.xml
* src/main/java/SimpleShortestPathsComputation.java

### package project
run in your *project directory*
`mvn package`

## Run giraph
If you didn't Test earlier
Prepare data and put it to hdfs like is examplained in Test paragraph
### Run example code(one line)
Note that if shortestpath output file already exist you need to bump it up to shortestpath1 because every output file must be unique

`giraph target/book-examples-1.0.0.jar SimpleShortestPathsComputation -vif org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat -vip /input/tiny_graph.txt -vof org.apache.giraph.io.formats.IdWithValueTextOutputFormat -op /output/shortestpath -w 1 -ca giraph.SplitMasterWorker=false`
### See results
`hdfs dfs -cat /output/shortestpath/part-m-00000`
# Instruction to install giraph using docker
## Starting container

1. `docker pull izone/hadoop`

2. `docker run -d --name giraph -v {PATH_TO_PROJECT_IN_YOUR_DISK}:/root/myProject izone/hadoop`
## Setup in your image terminal
### Add for apt-get to work
`mkdir -p /usr/share/man/man1`
### Install git and maven
`sudo apt-get update`
`sudo apt-get install -y maven git`

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
        <sourceDirectory>src</sourceDirectory>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```
### java file
Copy from giraph-examples *SimpleShortestPathsComputation* to your *project directory*/src/main/java

`pwd` should return your *project directory*

`mkdir src src/main src/main/java`

`curl https://raw.githubusercontent.com/apache/giraph/release-1.3/giraph-examples/src/main/java/org/apache/giraph/examples/SimpleShortestPathsComputation.java -o src/main/java/SimpleShortestPathsComputation.java`

Remove these lines from SimpleShortestPathsComputation.java in your *project directory*/src/main/java
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

### Build your project
`mvn clean compile assembly:assembly -DdescriptorId=jar-with-dependencies`

## Run project
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
`hadoop jar target/book-examples-1.0.0-jar-with-dependencies.jar org.apache.giraph.GiraphRunner SimpleShortestPathsComputation -vif org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat -vip /input/tiny_graph.txt -vof org.apache.giraph.io.formats.IdWithValueTextOutputFormat -op /output/shortestpaths -w 1 -ca giraph.SplitMasterWorker=false`

### See results
`hdfs dfs -cat /output/shortestpath/part-m-00000`

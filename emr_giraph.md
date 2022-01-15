# Instruction to install giraph on EMR
## Cluster config
Nodes:
* 1 master

EMR version: emr-5.34.0

hadoop 2.10.1
Currently giraph use only one worker :(
## Setup
### Install git
`sudo yum install git-core`

### Install maven
1. `sudo wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo`
2. `sudo sed -i s/\$releasever/6/g /etc/yum.repos.d/epel-apache-maven.repo`
3. `sudo yum install -y apache-maven`

### Add env variables
`nano ~/.bashrc`

override
`export JAVA_HOME=/usr/lib/jvm/java-1.8.0-amazon-corretto.x86_64`
add
`export GIRAPH_HOME=/usr/local/giraph`
save ctrl+o and exit with ctrl+x
`source $HOME/.bashrc`
## Build
### clone
`cd /usr/local/`

`sudo git clone https://github.com/apache/giraph.git`

`cd $GIRAPH_HOME`
### select version
`sudo git checkout release-1.3`
### build
Dhadoop.version=*your hadoop version* for this emr it's 2.10.1

`sudo mvn package clean package -DskipTests -Phadoop_2 -Dhadoop.version=2.10.1`
### Notes
* sudo for creating directories
* -Dhadoop.version=2.7.2 worked for 2.10.1

## Run giraph
### Prepare data
`nano /tmp/tiny_graph.txt`

Paste this into:

    [0,0,[[1,1],[3,3]]]
    [1,0,[[0,1],[2,2],[3,1]]]
    [2,0,[[1,2],[4,4]]]
    [3,0,[[0,3],[1,1],[4,4]]]
    [4,0,[[3,4],[2,4]]]
### Put to hdfs
`hdfs dfs -mkdir /user/hadoop/input`

`hdfs dfs -copyFromLocal /tmp/tiny_graph.txt /user/hadoop/input/tiny_graph.txt`

`hdfs dfs -ls /user/hadoop/input`
### Run example code(one line)
`hadoop jar $GIRAPH_HOME/giraph-examples/target/giraph-examples-1.3.0-SNAPSHOT-for-hadoop-2.10.1-jar-with-dependencies.jar org.apache.giraph.GiraphRunner org.apache.giraph.examples.SimpleShortestPathsComputation -vif org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat -vip /user/hadoop/input/tiny_graph.txt -vof org.apache.giraph.io.formats.IdWithValueTextOutputFormat -op /user/hadoop/output/shortestpaths -w 1 -ca giraph.SplitMasterWorker=false`
### See results
`hdfs dfs -text /user/hadoop/output/shortestpaths/p* | less`
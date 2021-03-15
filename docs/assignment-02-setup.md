# Hadoop Setup

[part 2](https://rubigdata.github.io/bigdata-blog-2021-joshdev-de/assignment-02-own-jobs) of the assignment

## Creating Docker container
In this assignment I use the following docker image:  
`rubigdata/course:a2`  

In order to start the container with name "hello-hadoop" and ports 8088 and 9870 forwarded to the host the following command is given:  
`docker create --name hello-hadoop -it -p 8088:8088 -p 9870:9870 rubigdata/course:a2`

For my convenience I add a volume mount. The volume mount allows me to edit the code files on Windows using my favorite text editor, while staying in the docker CLI of the docker container and not needing to copy files from the host to the container, but only inside the container.  
`docker create --name hello-hadoop -it -p 8088:8088 -p 9870:9870 -v /home/josh/uni/big-data/hello-hadoop-2021-joshdev-de/:/mnt/shared:ro rubigdata/course:a2`

To start and attach to the container's CLI I use.  
`docker start hello-hadoop`  
`docker attach hello-hadoop`

## Initializing hdfs and insert input data
Next I initialize the master node of the hdfs (hadoop filesystem) wich is called namenode using:  
`hdfs namenode -format`  
and start the dfs deamons of the datanodes and secondary namenodes using:  
`start-dfs.sh`

The next step is to create home a home directory for the current user which is hadoop:  
`hdfs dfs -mkdir -p /user/hadoop`  
This home directoy is the working directory of all commands of the style `hdfs dfs -<some fs command>` as well as all hadoop programs executed on the cluster.  
Many command line tools such as `ls`, `rm`, `cat` and `mkdir` are available here. A full list can be found at [here](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html).  

As hadoop programms should be run on some input which is just given by specifying the directory it is stored in, creating a directory for that is needed:  
`hdfs dfs -mkdir input`  
Downloading the example input for this assignment as well as copying it into the hdfs is done using these commands:  
`wget https://raw.githubusercontent.com/rubigdata-dockerhub/hadoop-dockerfile/master/100.txt`  
`hdfs dfs -put 100.txt input`      (`hdfs dfs -put <source in local fs> <target in hdfs>`)  

## Start job scheduler
In order to allow scheduling jobs on the hadoop cluster I need yarn (yet another resource negotiator) to do that for me:  
`start-yarn.sh`  

## Compile and run a program
Now that I prepared everything I can just copy the .java file I want to use from `/mnt/shared` to `/opt/hadoop` using `cp`. In this case I chose the simple   `WordCount.java example`.  

After that I can continue and compile my first program inside the `/opt/hadoop` directory:  
`hadoop com.sun.tools.javac.Main WordCount.java` (compiles the .java files to .class files)  
`jar cf wc.jar WordCount*.class` (puts all the compiled files into a jar archive)  

Finally the compiled hadoop program can be run:  
`hadoop jar wc.jar WordCount input output` (`hadoop jar <jarFileName> <mainClassname> <input dir> <output dir>`  
The output directory must not to exist already, in case it does I simply delete it using:  
`hdfs dfs -rm -r output`

## Inspect output
In order to see the generated out put I have two options:
### 1. Direct inspection
`hdfs dfs -cat output/*`
### 2. First copy to local fs, then inspect
`hdfs dfs -get output output`      (`hdfs dfs -get <source in hdfs> <target in local fs>`)  
`cat output/*`

## Inspect the clusters Web UIs
### namenode
I just enter [http://localhost:9870](http://localhost:9870) into my browser to see the namenodes web interface.
There I can see a lot of statistics about the hadoop cluster.

### running jobs
In order to see the running jobs I enter the hadoop Web UI under [http://localhost:8088/cluster](http://localhost:8088/cluster)

### examples

![namenode-ui]
![hadoop-ui]

[namenode-ui]: https://raw.githubusercontent.com/rubigdata/bigdata-blog-2021-joshdev-de/master/docs/namenode_ui.png?token=AMB4WGQGB7AA3M52GKY2IBTALCU6G "Namenode UI"
[hadoop-ui]: https://raw.githubusercontent.com/rubigdata/bigdata-blog-2021-joshdev-de/master/docs/hadoop-ui.png?token=AMB4WGW3XZUQAE7IIEBYDGTALCU6G "Hadoop UI"


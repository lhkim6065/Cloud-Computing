# Running Hadoop

1. Making the working directories
```
mkdir running-hadoop2
cd running-hadoop2
mkdir eu2
mkdir wxw2
```

2. Clone the big-data-europe fork for commodity hardware in my eu2 and wxw2 directories.
```
cd wxw2
git clone https://github.com/wxw-matt/docker-hadoop
cd eu2
git clone https://github.com/big-data-europe/docker-hadoop
cd docker-hadoop
```

3. Test Docker 

I didn't have my Docker running and had to start it.
```
docker run hello-world
```

4. Bring the Hadoop containers up.

```
docker-compose up -d
```

5. In a Hadoop cluster, we mostly work within the namenode. 

I used "docker ps" to list the nodes, then picked the one with name in it. For me:
```
docker ps
CONTAINER ID   IMAGE                                                    COMMAND                  CREATED          STATUS                   PORTS                                            NAMES
eb613475efdb   bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh /run…"   17 minutes ago   Up 3 minutes (healthy)   9864/tcp                                         datanode
954804d2a729   bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8       "/entrypoint.sh /run…"   17 minutes ago   Up 3 minutes (healthy)   8042/tcp                                         nodemanager
4255310c346f   bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8   "/entrypoint.sh /run…"   17 minutes ago   Up 2 minutes (healthy)   8088/tcp                                         resourcemanager
0b7756f349cd   bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh /run…"   17 minutes ago   Up 3 minutes (healthy)   0.0.0.0:9000->9000/tcp, 0.0.0.0:9870->9870/tcp   namenode
58c5d43263f2   bde2020/hadoop-historyserver:2.0.0-hadoop3.2.1-java8     "/entrypoint.sh /run…"   17 minutes ago   Up 3 minutes (healthy)   8188/tcp                                         historyserver
```

So I used:
```
docker exec -it namenode /bin/bash 
```

I tested "hello world" here:
```
echo hell world
```

6. Set up the node

While within the node, we will need directories for data, for compute, and for results. I created those under a new 'app' directory.

```
mkdir app
mkdir app/data
mkdir app/res
mkdir app/jars
```

7. Fetch data to app/data

'hello world' for 'map reduce' is 'word count' so we get some words to count. I got two of my favorite books and also Wuthering Heights from Project Gutenberg in plaintext format like so:

```
cd /app/data
curl https://www.gutenberg.org/cache/epub/1342/pg1342.txt -o austen.txt
curl https://www.gutenberg.org/cache/epub/84/pg84.txt -o shelley.txt
curl https://www.gutenberg.org/cache/epub/768/pg768.txt -o bronte.txt
```

It should be easy enough to find other text files on the Internet, but I used these three. Before going further, I verified that I had files of some size:

```
ls -al
```

I got: 

```
total 1888
drwxr-xr-x 2 root root   4096 May 31 03:29 .
drwxr-xr-x 5 root root   4096 May 31 03:26 ..
-rw-r--r-- 1 root root 772420 May 31 03:44 austen.txt
-rw-r--r-- 1 root root 693877 May 31 03:44 bronte.txt
-rw-r--r-- 1 root root      3 May 31 03:44 hi.txt
-rw-r--r-- 1 root root 448937 May 31 03:44 shelley.txt
```

8. In a separate terminal tab, I worked outside the container. Within my wxw docker-hadoop directory, I navigated to the jars folder.
```
cd jobs/jars
ls
```

I got:
```
WordCount.jar
```

We are using docker cp to copy over the WordCount.jar file into our contaoner.
```
docker cp WordCount.jar namenode:/app/jars/WordCount.jar
```

9. Load data in HDFS

We need to move data from the Linux file system into the Hadoop file system. We use the "hdfs" commands.

```
cd /
hdfs dfs -mkdir /test-1-input
hdfs dfs -copyFromLocal -f /app/data/*.txt /test-1-input/
```

10. Run Hadoop/MapReduce

```
hadoop jar jars/WordCount.jar WordCount /test-1-input /test-1-output
```

11. Copy results out of hdfs

```
hdfs dfs -copyToLocal /test-1-output /app/res/
```

12. See the results!

```
head /app/res/test-1-output/part-r-00000
```

I get this:
```
#1342]	1
#768]	1
#84]	1
$5,000)	3
&	1
($1	3
(801)	3
(By	1
(Godwin)	1
(He	1
```



















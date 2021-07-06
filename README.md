# apache-hadoop
This repository uses a Big Data framework Apache Hadoop using;
- HDFS, 
- Pig, 
- Hive, 
- MapReduce.

# User&#39;s Manual

## Part 1.

- To display the users and corresponding directories open the terminal and type:

  - hdfs dfs -ls PATH //this will show the folders and files in the specified path
  - hdfs dfs -ls /user // thus, this will display users and corresponding directories

## Part 2.

- Initially create a folder in Cloudera with the following command:

  - hdfs dfs -mkdir /user/cloudera/folder_name

- Then create 2 folders as input1 and input2 in the folder you created above (folder_name):

  - hdfs dfs -mkdir /user/cloudera/input1 /user/cloudera/input2

- To load our csv file to input1 with indicated attributes (blocksize = 32 MB, replication factor = 3), use following commands in order:

  - hdfs dfs -D dfs.blocksize = 33554432 -D dfs.replication=3 -put 2459501.csv /user/cloudera/2459501/input1

- To load our csv file to input2 with indicated attributes (blocksize = 64 MB, replication factor = 2), use following commands in order:

  - hdfs dfs -D dfs.blocksize = 67108864 -D dfs.replication=3 -put 2459501.csv /user/cloudera/2459501/input2

## Part 3.

Create Project:

- Create a java project (File-\&gt;New-\&gt;Project) using Eclipse.
- Select Java Project and click &quot;Next&quot;.

- Enter your project name (Sensors) and click &quot;Finish&quot;.

- To create a class in the project you created, right click &quot;src&quot; select New-\&gt;Class.
- Enter the class name (e.g. ProcSensors) and click Finish.

- Add dependencies (right click the project Select &quot;Build Path-\&gt;Configure Build Path&quot;)
- Click on &quot;Add External JARS…&quot; and select all necessary jars (e.g. hadoop-common-\*.\*.\*.jar, hadoop-mapreduce-client-core-\*.\*.\*.jar, etc.).

- You need to set run configurations for the project. To do this, click &quot;Run-\&gt; Run Configurations…&quot; and browse your current project (Sensors) as project and select the main class of the project (i.e. ProcSensors). To select input and output locations, click &quot;Arguments&quot; tab in the open window. Write your input path put a space and write your output path for the project (folder_name/input1/2459501.csv output). Click apply to apply these changes.

Create Classes:

- To create a MapReduce Program, we need to have a Mapper Class (SensorMapper) to run on every single block (machine), Reducer Class (SensorReducer) to collect and process Mapper Class&#39;s outputs and main class (ProcSensors) to run the program. Optionally we may have a Combiner (SensorCombiner) Class in between Mapper and Reducer classes and MapResult (MapResult) Classes to keep multiple results in a MapResult object.

- SensorMapper class extends Mapper Class. In SensorMapper class there is a map function which gets the input and transform it to &quot;key, value&quot; pairs.

- SensorCombiner class extends Reducer Class. In SensorCombiner class there is a reduce function which gets input (key: place, value: MapResult object containing temperature value) from the output of map function. SensorCombiner outputs the place as key and MapResult object as value containing occurrences, sum of temperature, minimum and maximum temperatures of places.

- SensorReducer class extends Reducer Class. In SensorReducer class there is a reduce function which gets input (key: place, value: MapResult containing occurrences, sum of temperature, minimum and maximum temperatures of places.) from reduce function of SensorCombiner class. SensorReducer outputs the place as Text containing place, occurrences of this place, minimum temperature of this place, maximum temperature of this place and average temperature of this place.

- ProcSensors class is the main class of the program. It handles the configurations of the project. Input output file configurations are handled in this class and also it sets the MapReduce structure for the project by creating a job instance and setting Mapper, Reducer, Combiner classes.

- MapResult is a class containing count, total, minimum and maximum variables

## Part 4.

- Define a table in Hive using the following query. You can use Hue for convenience.
- The query below will create a table in which the values will be read by comma for each row and the header row at top will be neglected.


Create Table
    
    CREATE EXTERNAL TABLE sensortable(place STRING, temperature FLOAT)
    ROW FORMAT DELIMITED 
    FIELDS TERMINATED BY ','
    TBLPROPERTIES("skip.header.line.count" = "1");
    

- Then load the data by indicating its path as follows;

```
    load data nipath 'user/cloudera/folder_name/input1/2459501.csv' into table sensortable;
```

- Finally, write a query as follows to select occurrences, minimum temperature, maximum temperature, average temperature for every place. (MapReduce for the hdfs file will be handled by Hive itself);

```
SELECT place,
count(temperature) as occurences,
min(temperature) as mintemp,
max(temperature) as miaxtemp,
avg(temperature) as avgtemp
FROM sensortable
GROUP BY place;
```

- The results are as follows;

![image](https://user-images.githubusercontent.com/29654044/124651162-44308c80-dea3-11eb-8900-43b93ab7f63e.png)

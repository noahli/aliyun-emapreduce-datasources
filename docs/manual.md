# Spark on Aliyun

## Requirements

- Spark1.3+

## Introduction

- This project supports interaction with Aliyun's base service, e.g. OSS, ODPS, LogService and ONS, in Spark runtime environment.

## Build and Install

```

		git clone https://github.com/aliyun/aliyun-emapreduce-datasources.git
	    cd  aliyun-emapreduce-datasources
	    mvn clean package -DskipTests

```

#### Use SDK in Eclipse project directly

- copy sdk jar to your project
- right click Eclipse project -> Properties -> Java Build Path -> Add JARs
- choose and import the sdk
- you can use the sdk in your Eclipse project

#### Maven 

```
        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-maxcompute_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>

        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-logservice_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>

        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-tablestore</artifactId>
            <version>2.2.0</version>
        </dependency>

        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-ons_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>

        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-mns_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>
        
        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-redis_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>
        
        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-hbase_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>
        
        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-jdbc_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>
        
        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-dts_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>
        
        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-kudu_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>
        
        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-datahub_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>
        
        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-druid_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>
        
        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-sql_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>

        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-common_2.11</artifactId>
            <version>2.2.0</version>
        </dependency>
        
        <dependency>
            <groupId>com.aliyun.emr</groupId>
            <artifactId>emr-kafka-client-metrics</artifactId>
            <version>2.2.0</version>
        </dependency>

```

## OSS Support

Older OSS sdk was obsoleted by Jindofs sdk. Please refer to:
[Jindofs sdk howto](./jindofs_sdk_how_to_en.md)

## ODPS Support

In this section, we will demonstrate how to manipulate the Aliyun ODPS data in Spark.

### Step-1. Initialize and OdpsOps
Before read/write ODPS data, we need to initialize an OdpsOps, like:


```

	import com.aliyun.odps.TableSchema
	import com.aliyun.odps.data.Record
	import org.apache.spark.aliyun.odps.OdpsOps
	import org.apache.spark.{SparkContext, SparkConf}
	
	object Sample {
	  def main(args: Array[String]): Unit = {	
	    // == Step-1 ==
	    val accessKeyId = "<accessKeyId>"
	    val accessKeySecret = "<accessKeySecret>"
		// intranet endpoints for example
	    val urls = Seq("http://odps-ext.aliyun-inc.com/api", "http://dt-ext.odps.aliyun-inc.com") 
	
	    val conf = new SparkConf().setAppName("Spark Odps Sample")
	    val sc = new SparkContext(conf)
	    val odpsOps = OdpsOps(sc, accessKeyId, accessKeySecret, urls(0), urls(1))

        // == Step-2 ==
		...
		// == Step-3 ==
		...
	  }

	  // == Step-2 ==
      // function definition
	  // == Step-3 ==
      // function definition
	｝

```

In above codes, the variables accessKeyId and accessKeySecret are assigned to users by system; they are named as ID pair, and used for user identification and signature authentication for OSS access. See [Aliyun AccessKeys](https://ak-console.aliyun.com/#/accesskey) for more information.

### Step-2. Load ODPS Data into Spark

```

		// == Step-2 ==
        val project = <odps-project>
	    val table = <odps-table>
	    val numPartitions = 2
		val inputData = odpsOps.readTable(project, table, read, numPartitions)
		inputData.top(10).foreach(println)

		// == Step-3 ==
        ...

```

In above codes, we need to define a `read` function to preprocess ODPS data：

```

		def read(record: Record, schema: TableSchema): String = {
	      record.getString(0)
	    }

```

It means to load ODPS table's first column into Spark.

### Step-3. Save results into Aliyun ODPS.

```

		val resultData = inputData.map(e => s"$e has been processed.")
		odpsOps.saveToTable(project, table, resultData, write)

```

In above codes, we need to define a `write` function to preprocess reslult data before write odps table：

```

		def write(s: String, emptyReord: Record, schema: TableSchema): Unit = {
	      val r = emptyReord
	      r.set(0, s)
	    }

```

It means to write each line of result RDD into the first column of ODPS table.

## ONS Support

In this section, we will demonstrate how to comsume ONS message in Spark.

```
    // cId: Aliyun ONS ConsumerID
    // topic: Message Topic
    // subExpression: Message Tag
    val Array(cId, topic, subExpression, parallelism, interval) = args

    val accessKeyId = "accessKeyId"
    val accessKeySecret = "accessKeySecret"

    val numStreams = parallelism.toInt
    val batchInterval = Milliseconds(interval.toInt)

    val conf = new SparkConf().setAppName("Spark ONS Sample")
    val ssc = new StreamingContext(conf, batchInterval)

	// define `func` to preprocess each message 
    def func: Message => Array[Byte] = msg => msg.getBody
    val onsStreams = (0 until numStreams).map { i =>
      println(s"starting stream $i")
      OnsUtils.createStream(
        ssc,
        cId,
        topic,
        subExpression,
        accessKeyId,
        accessKeySecret,
        StorageLevel.MEMORY_AND_DISK_2,
        func)
    }

    val unionStreams = ssc.union(onsStreams)
    unionStreams.foreachRDD(rdd => {
      rdd.map(bytes => new String(bytes)).flatMap(line => line.split(" "))
        .map(word => (word, 1))
        .reduceByKey(_ + _).collect().foreach(e => println(s"word: ${e._1}, cnt: ${e._2}"))
    })

    ssc.start()
    ssc.awaitTermination()
```

## LogService Support

In this section, we will demonstrate how to comsume Loghub data in Spark Streaming.

```
	if (args.length < 8) {
      System.err.println(
        """Usage: TestLoghub <sls project> <sls logstore> <loghub group name> <sls endpoint> <access key id>
          |         <access key secret> <receiver number> <batch interval seconds>
        """.stripMargin)
      System.exit(1)
    }

    val logserviceProject = args(0)    // The project name in your LogService.
    val logStoreName = args(1)         // The name of of logstream.
    val loghubGroupName = args(2)      // Processes with the same loghubGroupName will consume data of logstream together.
    val loghubEndpoint = args(3)       // API endpoint of LogService 
    val accessKeyId = args(4)          // AccessKeyId
    val accessKeySecret = args(5)      // AccessKeySecret
    val numReceivers = args(6).toInt   
    val batchInterval = Milliseconds(args(7).toInt * 1000) 

    val conf = new SparkConf().setAppName("Test Loghub")
    val ssc = new StreamingContext(conf, batchInterval)
    val loghubStream = LoghubUtils.createStream(
      ssc,
      loghubProject,
      logStream,
      loghubGroupName,
      endpoint,
      numReceivers,
      accessKeyId,
      accessKeySecret,
      StorageLevel.MEMORY_AND_DISK)

    loghubStream.foreachRDD(rdd => println(rdd.count()))

    ssc.start()
    ssc.awaitTermination()
```

## Future Work

- Support more Aliyun base service, like OTS and so on.
- Support more friendly code migration.

## License

Licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0.html)

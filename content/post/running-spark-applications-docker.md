+++

author = "Jacob Hell"
title = "Running Spark Applications on Dockerized Spark"
date = "2021-01-18"
description = "Running Spark Applications on Dockerized Spark"
tags = [ "spark", "docker", "scala" ]

+++

Sparking joy in Docker.

<!--more-->

I'm getting into data engineering stuff. The biggest thing in data engineering right now is Spark. Spark lets you perform distributed processes. I consider it to be the Hadoop successor, since it's so much faster.

Requirements:

1. Docker
2. Scala version 2.12.12 (Spark doesn't work with 2.13.* at the time of this writing)



## Getting Dockerized Spark Running

First, pull the docker image `bitnami/spark`, using this command:

```
docker pull bitnami/spark
```

It's going to take awhile to download, so I suggest pulling up some Rick and Morty. Two episodes should do the trick.

Then run it, using this command:

```
docker run -d bitnami/spark
```

`-d` runs in detached mode, so you retain access to your terminal emulator. Docker prints out the hash, keep this handy.

## Packaging a jar using sbt

Go to a Scala program that you want to run in Spark. If you are in need of one, use this:

```scala
/* SimpleApp.scala */
import org.apache.spark.sql.SparkSession

object SimpleApp {
  def main(args: Array[String]) {
    val logFile = "YOUR_SPARK_HOME/README.md" // Should be some file on your system
    val spark = SparkSession.builder.appName("Simple Application").getOrCreate()
    val logData = spark.read.textFile(logFile).cache()
    val numAs = logData.filter(line => line.contains("a")).count()
    val numBs = logData.filter(line => line.contains("b")).count()
    println(s"Lines with a: $numAs, Lines with b: $numBs")
    spark.stop()
  }
}
```

I took this snippet from [here](https://spark.apache.org/docs/latest/quick-start.html).

Package the program using `sbt`:

```
sbt package
```

## Uploading the jar and Running the Spark Application

To upload the jar to the docker container, we are going to use the `docker cp` command. This is where you need the hash.

Run this command:

```
docker cp <jar_file_on_your_machine>.jar <hash>:/opt/bitnami/spark/app.jar
```

Then, shell into your docker container using the command:

```
docker exec -it <hash> bash
```

Lastly, in your spark docker container, run this command:

```
bin/spark-submit --class "SimpleApp" --master local[4] app.jar
```

If you see something similar to **Lines with a: 46, Lines with b: 23**, then good job, it works! You are ready for more Spark adventures.
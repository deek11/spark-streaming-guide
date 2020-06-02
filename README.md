This reposistory is an organized collection of resources to help understand the basics of spark streaming. 
It also covers the common issues associated with running a spark job in production.

Basics 
=======

  

*   [How Applications are Executed on a Spark Cluster](https://www.informit.com/articles/article.aspx?p=2928186)
*   [Deep-dive into Spark internals and architecture](https://www.freecodecamp.org/news/deep-dive-into-spark-internals-and-architecture-f6e32045393b/)
*   [Spark Streaming](https://jaceklaskowski.gitbooks.io/spark-streaming/)
*   Spark Programming
    *   [Understanding Closures](https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#understanding-closures-a-nameclosureslinka)
    *   [Broadcast Variables](https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#broadcast-variables)
*   [Spark Tuning](https://spark.apache.org/docs/latest/tuning.html)
*   [Spark Memory Management](https://spark.apache.org/docs/latest/tuning.html#memory-management-overview)
*   [Spark Monitoring](https://spark.apache.org/docs/latest/monitoring.html)
*   [Running Spark on YARN](http://spark.apache.org/docs/latest/running-on-yarn.html)
*   [Spark Streaming Configuration](https://spark.apache.org/docs/latest/configuration.html#spark-streaming)
*   [Executor Memory and Parallelism](https://blog.cloudera.com/how-to-tune-your-apache-spark-jobs-part-2/#:~:text=The%20cores%20property%20controls%20the,tasks%20at%20the%20same%20time.)  
      
      
    

Common Issues 
==============

  

  

Issue

Details

Fix

Reference

1

Spark Streaming failed executor tasks

Spark DAGScheduler builds stages of task and submit task to TaskScheduler to run the task. TaskScheduler launch the task on executors and re-run the failed task into different executors and reports to DAGScheduler. You can control the number of times, task to be retried before it gives up using spark.task.maxFailures.

  

For long-running jobs you could consider to boost maximum number of task failures before giving up the job. By default tasks will be retried 4 times and then job fails.

*   spark.task.maxFailures=8

[https://stackoverflow.com/questions/40195309/spark-streaming-failed-executor-tasks](https://stackoverflow.com/questions/40195309/spark-streaming-failed-executor-tasks)

[http://mkuthan.github.io/blog/2016/09/30/spark-streaming-on-yarn/](http://mkuthan.github.io/blog/2016/09/30/spark-streaming-on-yarn/)

2

OutOfMemory Error

*   Driver Level
*   Executor Level
*   Node Manager Level

  

  

[Spark Tuning - Memory Management 1](https://spark.apache.org/docs/latest/tuning.html#memory-management-overview)

[Spark Tuning - Memory Management 2](https://unraveldata.com/common-reasons-spark-applications-slow-fail-part-1/)

[Memory Configuration 1](https://spark.apache.org/docs/latest/configuration.html#application-properties)

[Memory Configuration 2](https://spark.apache.org/docs/latest/configuration.html#memory-management)

  

3

Excessive Garbage Collection

  

  

[Spark Tuning - Garbage Collection](https://unraveldata.com/common-failures-slowdowns-part-ii/)

4

WebUI consuming Driver Heap space

Apart from creating the RDDs and co-ordinating the execution of tasks within the different stages of a Job, the driver is also responsible for running the web-ui on 4040 port. The Web ui is used for monitoring the Spark job. For long running jobs, we should limit the data being shown on web-ui to avoid memory consumption.

The default SparkUI values are much larger than needed. After setting them down to 1/20th of the default values, the system runs stably for 24 hours with no increase in heap usage over that time.For clarity, the values that were edited were:

*   spark.ui.retainedJobs=50
*   spark.ui.retainedStages=50
*   spark.ui.retainedTasks=500
*   spark.worker.ui.retainedExecutors=50
*   spark.worker.ui.retainedDrivers=50
*   spark.sql.ui.retainedExecutions=50
*   spark.streaming.ui.retainedBatches=50

  

5

Jobs stuck in the Spark UI without any progress. This complicates identifying which are the active jobs/stages versus the dead jobs/stages.

Whenever there are too many concurrent jobs running on a cluster, there is a chance that the Spark internal `eventListenerBus` drops events. These events are used to track job progress in the Spark UI. Whenever the event listener drops events you start seeing dead jobs/stages in Spark UI, which never finish. The jobs are actually finished but not shown as completed in the Spark UI.

You observe the following traces in driver logs:

There is no way to remove dead jobs from the Spark UI without restarting the cluster. 

[Active v/s Dead Jobs](https://kb.databricks.com/jobs/active-vs-dead-jobs.html)

6

Spark Streaming - Input Rate is too high to process within the batch interval

  

  

Spark Steaming ingests data through receivers at the rate of producer (or a user configured rate limit) when the processing time for a batch is longer than the batch interval the system is un-stable, data queues up, exhaust resources and fails with out of memory(aka OOM) error.

Use Backpressure - For each batch completed, estimate a new rate based on previous batch processing and scheduling delay.

This enables the Spark Streaming to control the receiving rate based on the current batch scheduling delays and processing times so that the system receives only as fast as the system can process. Internally, this dynamically sets the maximum receiving rate of receivers. Backpressure rate is upper bounded by the maxRatePerPartition.

*   spark.streaming.backpressure.initialRate=1
*   spark.streaming.backpressure.enabled=true  
    
*   spark.streaming.kafka.maxRatePerPartition=3  
    

  

[Spark Streaming - Backpressure](https://www.linkedin.com/pulse/short-note-spark-streaming-backpressure-ram-ghadiyaram/)

7

Every SparkContext launches a web UI, by default on port 4040, that displays useful information about the application.

Note that this information is only available for the duration of the application by default. To view the web UI after the fact, set  `spark.eventLog.enabled`  to true before starting the application. This configures Spark to log Spark events that encode the information displayed in the UI to persisted storage.

Set spark.eventLog.enabled=true

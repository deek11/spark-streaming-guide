This reposistory is an **organized collection of resources** to help understand the basics of spark streaming. 
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

|S.No.|Issue|Details|Fix|Reference|
|--- |--- |--- |--- |--- |
|1|Spark Streaming failed executor tasks|Spark DAGScheduler builds stages of task and submit task to TaskScheduler to run the task. TaskScheduler launch the task on executors and re-run the failed task into different executors and reports to DAGScheduler. You can control the number of times, task to be retried before it gives up using spark.task.maxFailures.|For long-running jobs you could consider to boost maximum number of task failures before giving up the job. By default tasks will be retried 4 times and then job fails.spark.task.maxFailures=8|https://stackoverflow.com/questions/40195309/spark-streaming-failed-executor-tasks http://mkuthan.github.io/blog/2016/09/30/spark-streaming-on-yarn/|

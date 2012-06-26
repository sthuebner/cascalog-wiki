# Cascalog and Hadoop Security

If your cluster features *Hadoop Security* your queries may run into
exceptions like this one:

    org.apache.hadoop.ipc.RemoteException: token (...) can't be found in cache

That exception fails the second step in any multi-step Cascalog (or
Cascading for that regard) query. Reason is, the Kerberos token gets
cancelled after the first step succeeded.

A solution to this is to configure JobConf with
`mapreduce.job.complete.cancel.delegation.tokens` set to `false`, like
so:

```clojure
    (with-job-conf {"mapreduce.job.complete.cancel.delegation.tokens" false}
      ...)
```

Or add it to your `job-conf.clj`.

Also, if you happen to schedule your Cascalog jobs via Oozie, you may
want to
[google for HADOOP_TOKEN_FILE_LOCATION and mapreduce.job.credentials.binary](https://www.google.com/search?q=%22HADOOP_TOKEN_FILE_LOCATION%22%20%22mapreduce.job.credentials.binary%22)
and set your jobconf accordingly.


## Resources

- [Owen O'Malley: Motivations for Apache Hadoop Security](http://hortonworks.c
om/blog/motivations-for-apache-hadoop-security/)
- [Owen O'Malley: Hadoop Security in Detail](http://developer.yahoo.com/blogs/ydn/posts/2010/07/hadoop_security_in_detail/) (video)
- [Cloudera CDH3 Documentation: Introduction to Hadoop Security](https://ccp.cloudera.com/display/CDHDOC/Introduction+to+Hadoop+Security)
- [MAPREDUCE-1430](https://issues.apache.org/jira/browse/MAPREDUCE-1430)
- [MAPREDUCE-4324](https://issues.apache.org/jira/browse/MAPREDUCE-4324)

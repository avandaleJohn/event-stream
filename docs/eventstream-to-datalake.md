

## EventStream data volumes

Within the various POC’s performed against the EventStream, questions often centre on how the DMS/Kinesis combination would handle the EventStream.

From examining the data over the past 18 months, we can see that the EventStream generates approx. 10 million events per month.

Over the past 90 days, the events have averaged per day:

On a typical Tuesday (in this example March 24th), the events peak at approx. 3PM GMT (9AM CST) with roughly 40,000 events over a 60 min period.

And in that peak hour, the events are distributed like so:

Apart from a couple of spikes, we don’t really go above 200 per minute or approx. 3 per second at peak running hour.

The numbers for the event stream patterns are consistent on an hour per day, day per week basis, week to week basis.  The only outlier in the data so far seem to be a roughly 10% drop off in March 2020 activity, presumably due to covid-19 fallout.

Getting the EventStream into the Data Lake

To use DMS, we enable CDC on the Quest database and create a replication task that replicates the rows from the EventStream onto Kinesis, using a Replication Instance.

Database Migration Service uses an EC2 service with an AWS agent dependent upon the type of task that is being invoked. It offers the T2 ($0.036 -$0.292 per hour multi-AZ), C4 ($0.308-$2.47) and R4 ($0.41-$6.60) classes for the Replication Instance.  DMS documentation is unclear on the size of instance that should be used in production.  However, it recommends T2 for dev and QA environments.

The storage of the replication instance is used for caching of transaction log data for long running transactions. The rate at which the DMS can process the events is tied heavily to the size of instance chosen.  However, if the DMS is overwhelmed, it simply pauses reading from the transaction log of the source database until it has caught up.

We may be ok with T2 for production grade replication of the event stream as we would be reading from a single table which is resource constrained to a single thread; additionally, there is no complicated mapping of the records.  We are simply packaging each row from the database table into a Kinesis record, which would be less memory intensive that if we were manipulating the structure of the payloads.

Storage is charged at $0.23 per gb per month in a multi AZ instance.  For a T2 instance, the storage size is 50Gb (so, $11.50 per month)

Due to the relatively low data volumes coming from quest, we can extrapolate that at 5 events per second, we would reach 13 million in a month.  This would be 25% higher than any month so far.

However, using 5 events per second, at roughly 3kb per event, we’ve used the Kinesis calculator to calculate the following:

Firehosing is priced per GB of ingested data $0.029 per GB.


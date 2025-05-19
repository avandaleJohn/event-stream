

## Working with the EventStream in the Data Lake

If we attempt to iterate through the quest data, table by table, we will need to have a process per table (presumably via a python script) that can clean the table and move to a raw bucket and apply to the existing data in the data lake.

Because each table has different structures, each one will require a glue job to execute the python script. The job will require a cloudformation stack, a deployment pipeline and associated tests for all 4. We will also need to have a process for when the quest schema changes, as it does, on a quarterly basis.  There are 620 tables in Quest, with more to be added via the OCE functionality coming in from Quest Finance.  This mode changes with each release, so we will also need to be able to successfully handle multiple versions of the quest data model.

The major advantage of streaming the data into the lake from the EventStream is that it allows us to have a single process that can interpret the incoming events against a single known pattern (the event source structure) effectively as they are produced.

This would greatly reduce the effort required to ingest the Quest database; and process the deltas. We should also be conscious that the tables do not move in concert with one another.  A person entity is made up of 59 different tables that update at different paces.

By using the events from the event stream, we can use a single master process that can interpret the files that have been firehose into S3:

Kinesis Firehoses in the data in 1mb files

A glue crawler can identify a single schema in these files.

We can load the S3 object (1mb) into memory and group by each affected table in the file (e.g. 5 person rows, 15 person jobs, 3, person restriction).

For each affected table, we can locate the object that holds the “latest” version of that row in the lake (e.g. access the folders for person, personjob, person restriction)

We can then create the next version of the record by applying the changes from the eventstream record with an in memory version of the previous row. (e.g. load the object holding personrestriction 1234 into a dataframe and apply the change)

We can then persist the latest version of the object with the change.

We can also mark the previous version as “not current”

By having a single worker process that can apply these changes we would have something that is more testable and more manageable in production. Changes and additions to the Quest data model would have not a material effect on this process, as the eventstream data structure does not change from release to release.

We also significantly reduce the cost of storage and compute needed to maintain the Quest data within the data lake. By only ingesting the new data into the lake, we avoid extraneous, redundant data (the person root table alone has 124 columns)



## Flattening the data within the data lake

Alongside syncing the data in the lake from the event stream, we could also flatten out the events themselves for fast rollup style queries.

Rather than interrogating the nested json within each row, we can flatten out in order to make the rows indexable.  This would mean that the changeset from the earlier examples would go from this JSON blob:

[{"ColName":"TransactionId","New":"54BAB960-C3B2-4CB6-A08F-AAA11FB0F16E","Old":"A0018218-B959-4F79-9D01-CBF25A65F73D"},{"ColName":"PersonRestrictionCessationReasonCategoryId","New":"6","Old":null},{"ColName":"EndDate","New":"2020-04-05 16:47:34.4164179","Old":"2020-05-03 15:25:41.4689487"},{"ColName":"UpdEmpEPID","New":"9626","Old":null},{"ColName":"PersonRestrictionUpdDT","New":"2020-04-05 16:47:34.4634179","Old":"2020-04-03 15:25:41.4909508"},{"ColName":"PersonRestrictionVersionNumber","New":"2","Old":"1"}]

To This table structure:

In a similar vein, we could break out each affected entity so it would have its own copy of the row.

Once we had the data flattened out like this, the sample queries from above become more generic:

--how many jobs had their primary indicator switched from 1 to 0 in the past 7 days?

select count(*) from eventstream.eventsource

where TriggerInvokeTimestamp > '20200318'

and TransactionTable = 'PersonJob'

and ColumnName = 'PrimaryIndicator'

and NewValue = '1'

and OldValue = '0'

select count(*) from eventstream.eventsource

where TriggerInvokeTimestamp > '20200318'

and TransactionTable = 'Person'

and ColumnName = 'ExecutiveEntityRestrictionSnapshotCategoryId'

and NewValue = '3'

and OldValue = null

--Day by Day breakdown of the above query

select Day(TriggerInvokeTimestamp) as DateOfChange,count(1) as ChangeCount from eventstream.eventsource

where TriggerInvokeTimestamp > '20200318'

and TransactionTable = 'Person'

and ColumnName = 'ExecutiveEntityRestrictionSnapshotCategoryId'

and NewValue = '3'

group by Day(TriggerInvokeTimestamp)

Once we’ve done this, we could then perform aggregations on the data and persist the outputs into the data warehouse.

By persisting the flattened data in S3, Data Scientists could also use Athena to execute the above queries.


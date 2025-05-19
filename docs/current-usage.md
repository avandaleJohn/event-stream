

## How the EventStream is used today in the Quest application

Once a row has been written to the EventSource table, there is a further trigger which cross references the table to a whitelist of downstream products that need to be notified when certain changes occur.  For PersonRestriction, there are 2 downstream products:

Based upon the example changeset in the example above, restrictions will re-evaluate the Person for any outstanding restrictions now that this one has been ended.  It will use the AffectedEntity column to re-evaluate person 8584947.

AssignmentAudit will log that a person who had been reserved was subsequently eliminated. It will convert the changeset into an audit record along with the EmployeeId and TriggerInvokeTimestamp. It will then mark them against the AffectedEntity of type 3 (Assignment) Assignment Id 220014024.

In this way, the EventStream proactively pushes out messages to products as pertinent events arise.  This has eliminated the thundering herd effect that has occurred in the past when multiple resource-intensive batch processes have attempted to interrogate the Quest database and Quest History databases at the same time.

Today the event stream has replaced the quest history database which previously captured a copy of every version of every roll in every table in the quest database.  When a developer is attempting to obtain the history of a record, they no longer need to view different versions of the row and figure out a process for understanding the deltas.

We also can replay these events in sequence into an older copy of Quest. If nightly backups are taken of the production database, we have the ability to sync a copy of Quest to a declared point in time.   Each event contains enough information to be converted back to an Insert, Update or Delete statements.  We have a process that can iterate through them from a given start point up to an end point. This process was used to sync data from Production back to Production Support as part of the Workday HR effort in 2019.

However, all the products that currently use the event stream must do so through SQL Server message broker each product gets its own broker service.

This is not sustainable or scalable for future use. We intend for some of these events to be represented as nodes and relationships in graph databases, but we do not expect those graph databases to query a relational database.

As part of the initial design for the event stream we also intended to allow the database here to publish these events outside of SQL Server, so that they can be consumed by products that will not have any interaction whatsoever with our database layer.

Our identified mechanism for doing this is a combination of the AWS Database Migration Service and Kinesis Firehose.

The above diagram is from a POC that we built in the summer of 2019.

It illustrates how we could use the event stream to proactively “sync” events to a graph database, to be used for products such as restrictions or relationships.  The events coming from the event stream are read from the Quest database by the Database Migration Service, with each row subsequently converted into a Kinesis message then pushed to a Kinesis DataStream.  The messages are then simultaneously pushed to Lambda while also using Kinesis Firehose to push them to an S3 bucket for long term storage.

Putting aside the Neo4J use case for the moment; this approach demonstrates how we can leverage the Database Migration Service and Kinesis to push the Event Stream events to the Data Lake in near real time by following steps 1 and 4 in the diagram.


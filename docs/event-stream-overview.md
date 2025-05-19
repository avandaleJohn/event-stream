

## Purpose

The purpose of this document is to centralise the knowledge of the EventStream and its capabilities so that team members can gain a holistic understanding of its capabilities and possible future use.



## What is the Event Stream?

The event stream is an append only, immutable log produced in SQL Server based upon events created in the quest database. It was deployed to production in December 2018 as a replacement for the quest history database.

It was constructed to overcome the following problems:

The quest database was pushing every version of every row on every table to a QuestHistory database that mirrored the structure of the Quest database 590 tables.  This was costly to maintain and unwieldy to develop against.

Significant resources were being expended on attempting to reconstruct state of entities after the fact for historical purposes

Multiple processes were attempting to answer similar “change” questions using contradictory logic.

The intention of the event stream was to build a single source of truth that could publish what happened, as it happened; and then notify products of the exact changes that had occurred.

There are currently 6 products within the Quest eco-system that use the EventStream today in production to respond to changes as they occur:

AssignmentAudit

ConfidentialAccess

PersonReadModel

QuestReports

Restrictions

SOLR



## How the EventStream is produced



## How its built

The event stream uses a process built in PowerShell to analyze all tables in our quest database, including their foreign key relationships to one another. This process then generates triggers for each table that can compare a newly modified row with its predecessor and derive the changes that occurred, column by column.

All the code used in the EventStream code is automatically generated through Powershell scripts that can detect change to DDL.

As the schema evolves, the event stream triggers are regenerated automatically to reflect the altered DDL.   This is not a Quest specific solution, an event stream can be produced for any of our relational databases that use integers as their primary keys.



## How it runs

At runtime, the jumping off point is a single table called EventSource.  All 591 tables of Quest are funnelled into this single table structure in the order in which the events are produced. Each table pushes a payload of the changed columns to a processor queue as part of the transaction commit on our SR replication of the Quest database.

It is then picked up by a processor which can trace the root entities of each record.

The final output is then persisted to a table called the EventSource table which is the raw stream of all events.

When this happens, it triggers a process to cross reference each event with a service list that determines which products need to be notified of changes.  E.g. a change to CompanyLocationPostalAddress is sent to PersonReadModel, AssignmentAudit and QuestReports.

This entire process gives us a single log table that contains every change to the Quest data since December 2018.


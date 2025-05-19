---
title: Event Stream Architecture
---

# Event Stream Architecture

![Architecture Diagram](./assets/event-stream-architecture.png)

This documentation provides a detailed overview of an append-only Event Stream architecture implemented in SQL Server for the Quest application suite.

## Overview

The Event Stream provides a central, immutable log of data changes, enabling real-time change data capture (CDC) and downstream data integration with AWS services like Kinesis, S3, and Redshift.

## Features

- Fully append-only, immutable log
- Real-time processing via SQL triggers and PowerShell automation
- Integration with AWS DMS and Kinesis for scalable event delivery
- Downstream consumption by Quest microservices
- Simplified historical state reconstruction
- Support for data lake and federated Redshift querying

## Architecture

See below for a high-level view of the architecture:

![Architecture Flow](./assets/event-stream-architecture.png)

## Components

- **EventSource Table**: Core table capturing all data changes
- **PowerShell Trigger Generator**: Auto-generates SQL triggers from schema
- **Processor**: Derives affected entities and notifies consumers
- **AWS DMS**: Captures changes and streams them to AWS
- **Kinesis Firehose**: Streams event payloads to S3
- **Glue & Athena**: Enables analytics in the Data Lake
- **Redshift Federated Query**: Optional architecture for real-time analytics

## Use Cases

- Real-time audit logging
- Change notification to downstream systems
- Time-travel debugging and data reconstruction
- Stream processing into graph databases

## Learn More

- [Event Stream Data Model](./data-model.md)
- [Sample Queries](../queries/sample-queries.sql)
- [Working with the Data Lake](./data-lake-usage.md)
- [Redshift Integration Idea](./redshift-idea.md)

# event-stream

## Event Stream Architecture

This repository outlines the architecture, design decisions, and operational use of an append-only Event Stream built on SQL Server for the Quest application suite. Developed to replace a costly historical logging model, this solution provides real-time data change capture, downstream processing, and integration with AWS services including DMS, Kinesis, and S3.

The Event Stream has scaled to support 10M+ events monthly across 6+ consumer applications, offering a maintainable, high-fidelity log of change events with potential extensions to data lakes, graph databases, and Redshift.

Explore the documentation in `/docs/` to understand the data structure, implementation details, and use cases.


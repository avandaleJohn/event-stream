

## Out of the box idea for Redshift

Another option could be to not push this data to the data lake, and instead leverage the federated querying capabilities of Redshift.

Federated Query allows you to incorporate live data as part of your business intelligence (BI) and reporting applications. The intelligent optimizer in Redshift pushes down and distributes a portion of the computation directly into the remote operational databases to speed up performance by reducing data moved over the network. Redshift complements query execution, as needed, with its own massively parallel processing capabilities. Federated Query also makes it easy to ingest data into Redshift by letting you query operational databases directly, applying transformations on the fly, and loading data into the target tables without requiring complex ETL pipelines.

Using Redshift Federated Query ()
You can now also access data in RDS and Aurora PostgreSQL stores directly from your Redshift data warehouse. In this way, you can access data as soon as it is available. Straight from Redshift, you can now perform queries processing data in your data warehouse, transactional databases, and data lake, without requiring ETL jobs to transfer data to the data warehouse.

Redshift leverages its advanced optimization capabilities to push down and distribute a significant portion of the computation directly into the transactional databases, minimizing the amount of data moving over the network.

The idea here would be to use the Database migration service to sync the events from Quest in SQL Server to a copy of Quest in Aurora (Redshift federated querying is not compatible with SQL Server). The data in Aurora would then retain its structure and referential integrity. This data could then be ingested directly into Redshift and shaped according to the redshift schema, then used for the BI queries.   This would avoid the need for costly glue jobs to join and shape the quest data.

As part of this process, we could also use the S3 Export functionality to push this data from Redshift into the data lake for further use by other consumers.


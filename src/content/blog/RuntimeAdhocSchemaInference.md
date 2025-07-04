How to deal with runtime adhoc schema inference in Starburst/Trino

True runtime adhoc schema inference is not inherently available for formats like parquet,ORC, JSON , or CSV in trino. The reason for this is that when creating tables from these file formats, it's necessary to explicitly declare the schema in the CREATE TABLE statement. THis means that the user must specify column names and types. 

Starbursts Schema Discovery Solution

The discover_schema procedure in Starburst is designed to address the challenges of schema inference when querying data stored in specific formats within object storage. The discover_schema procedure automatically scans files located in objet storage to infer the schema.

Workflow

Procedure Call: To use the discover_schema, you initiate it with a SQL command like:

CALL hive.system.discover_schema(
    catalog => 'hive',               -- Target catalog
    schema => 'my_schema',           -- Target schema/database
    table => 'my_table',             -- Desired name for the new table
    location => 's3a://my-bucket/path/',  -- S3 location to analyze
    format => 'PARQUET'              -- Data file format (supports PARQUET, CSV, JSON, etc.)
);

Return Value: The discover_schema procedure returns a CREATE TABLE Data Definition Language (DDL) statement, which contains inferred columns and types based on the scanned data files.
Manual Execution Required: After receiving the DDL statement, users must manually execute it to register the table in the catalog. This step is critical for making the inferred schema available for querying.

Key Points to consider

The Discover_schema procedure supports formats such as Parquet, ORC, JSON and CSV
It’s important to note that schema inference is not dynamic. If the structure of the underlying files changes, the schema discovery process will then have to be re-run and the DDL executed manually to reflect the updates
Conclusion

The discover schema procedure streamlines the process of schema management within Starburst by  automatically inferring file schemas.

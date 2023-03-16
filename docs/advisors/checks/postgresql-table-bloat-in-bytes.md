# Check table bloat in bytes

## Description

This check verifies the table bloat from the currently connected database.

It returns a warning if any table in the current database has a bloat over 1GB size and the bloat represents at least 20% of the total table size. This is to avoid messages for 1GB bloat on huge tables.


!!! warning alert alert-success "Warning"
    
    At the moment of creation of this advisor, the PMM advisor engine connects only to the *pmm* database, so this advisor only checks that database. We would need the engine to be able to establish connections to each database in the PostgreSQL instance to get information.

Bloat is a natural side effect of how the Multiversion Concurrency Control (MVCC) works in a PostgreSQL instance. 

Every time a tuple gets updated or deleted, a "dead" version of it is created within the data pages. These dead tuples are tracked in the visibility map, and the regular VACUUM operations should be enough to release them and mark them as free space so new inserts can reuse the space. 

There are some situations where the VACUUM cannot catch up with the volume of changes, and the number of dead tuples starts to rise. In such conditions, after some time, the table gets bloated because of all this unused and wasted space. 

When a table gets bloated, this affects the general performance of the queries over the table. This is because accessing the data needs to go through all the existing dead tuples, adding time to the read operations. 


## Resolution
When a table accumulates significant bloat, running VACCUM will be insufficient. The solution is to rebuild the table so all the unused space from the dead tuples is eradicated. 

There are two main ways to do this:

- Execute [pg_repack](https://reorg.github.io/pg_repack/) in the bloated table.
- Run VACUUM FULL on the bloated table.

In both cases, the table gets physically rebuilt. But you need to consider that the VACUUM FULL operation holds an exclusive lock on the table, preventing any other operation until it finishes. This could impact a running production service and generally is undesired.

The **pg_repack** approach is less intrusive than the VACUUM FULL, it only acquires brief locks on the table, and all the rebuild operations happen online on a "shadow" table. Once it is complete, the process just flips the tables, and the new one with no bloat becomes the active one. In many production environments, the **pg_repack** option is the chosen one. 

Remember that using **pg_repack** requires the table to have a primary key, or at least a UNIQUE total index on a NOT NULL column. 

Also, suppose the table is the source of a logical replication (publication). 
In that case, it is recommended to stop the replication during the repack and resume it once the operation is finished to avoid unexpected effects. 

## Need more support from Percona?
Subscribe to Percona Platform to get database support with guaranteed SLAs or proactive database management services from the Percona team.

[Learn more :fontawesome-solid-paper-plane:](https://per.co.na/subscribe){ .md-button }
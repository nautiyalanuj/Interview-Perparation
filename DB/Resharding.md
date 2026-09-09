# How to reshard your database? So, suppose you have 4 db and you want to move to 8 db, how will you do resharding without stopping traffic?
Sharing some of the ways, detail points need to be factored out, but sharing some of the possible option

## Dual Write
- Before moving old data, configure your database proxy, router, or application layer to write all new incoming mutations (inserts, updates, deletes) to both locations simultaneously.
  - Writes: Sent to both the old 4 shards and the new 4 shards (totaling 8 shards).
  - Reads: Still served exclusively from the old 4 shards to ensure performance and accuracy.
  - Error Handling: If a write to a new shard fails, log it to an asynchronous retry queue so no updates are dropped.
-  Backfill Historical Data
  - With new data safely streaming to both setups, copy the existing data from the old 4 shards to the new 4 shards.
  - Chunking: Extract data in small, sequential batches using primary keys to avoid overloading production CPU.
  - Deduplication: If a historical record has already been updated by the live dual-writes, skip it or overwrite it using a timestamp check (last-write-wins).
- Run Checksum Verification
  - Do not trust the migration blindly. Before changing read traffic, run a verification loop.
 - Streaming Checksums: Use tools like Vitess VReplication or custom scripts to compute hashes of data blocks on both the old and new shards.
 - Fix Discrepancies: Automatically catch up on any missed rows before proceeding.
- Cutover (Switch Reads)
  - Once the new 8 shards are perfectly synchronized and verified, change the application routing rules.
  - Switch Traffic: Flip the read traffic from the old 4 shards to the new 8 shards.
  - Turn Off Dual-Writes: Stop writing to the old 4 shards.
  - Decommission: Keep the old 4 shards running in read-only mode for a few days as a backup, then safely wipe them.

## CDC + Delta Backfilling
- Take a Point-in-Time Snapshot: Extract a backup copy of the data from your 4 shards. Record the exact log position (the transaction ID or timestamp) when the snapshot was taken.
- Restore and Re-hash to 8 Shards: Load that snapshot into your new 8 shards. Your sharding proxy (like Vitess or Citus) will process the backup and distribute the rows to the correct new nodes based on the new 8-shard map.
- Start the "Delta" Stream (CDC): Tell your migration tool to read the old database's logs starting exactly from the log position recorded in Step 1.
- Continuous Catch-Up: The tool catches up on the "lost data"—any inserts, updates, or deletes that happened while your snapshot was transferring. It reads them sequentially from the log, hashes them, and routes them to the new 8 shards
- Once the replication lag drops to zero, temporarily block writes for a few seconds, let the final few transactions drain into the new shards, and flip your application traffic to the 8 shards.
- Why this is better than Dual-Writing
  - Zero Application Code Changes: Your application does not need to know about the 8 shards until the exact second you cut over.
  - No Lost Data: The database transaction logs are persistent. If the migration tool crashes, it resumes exactly where it left off without missing a single write.
  - Lower Performance Overhead: Reading transaction logs places far less stress on production CPUs than application-level dual-writing. [1] (https://medium.com/google-cloud/online-database-migration-by-dual-write-this-is-not-for-everyone-cb4307118f4b)

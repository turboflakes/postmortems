# Postmortems

## 22/Mar/2022
### 1. Incident on turboflakes.io/COCO and turboflakes.io/MOMO nodes resulting on two offline events in each node.

The incident happened after a node migration from v0.9.17 to v0.9.18. 

_History_

This Kusama validator node was running with the following flags at the time the incident happened: 

``` 
--chain=kusama --validator --unsafe-pruning --pruning=256 --database=paritydb-experimental
```

**Note**: While attempting this migration I become aware of the issue described here - [paritydb breaks on upgrade to v0.9.18](https://github.com/paritytech/polkadot/issues/5168) also related to this migration on a ParityDB database.

_Tasks performed_

1. As performed in previously migrations, I started to execute on node (turboflakes.io/COCO) the script to perform the binary update to v0.9.18.
2. After the node restarted the following error could be seen in logs:
```
Mar 21 21:26:21 localhost polkadot[475304]: Error:
Mar 21 21:26:21 localhost polkadot[475304]:    0: Error at calling runtime api: Runtime code error: `:code` hash not found
Mar 21 21:26:21 localhost polkadot[475304]:    1: Runtime code error: `:code` hash not found
```
3. The node was not starting so I have updated the binary to the previous version v0.9.17 and the following logs were seen:

```
Mar 21 21:29:39 localhost polkadot[475567]: Error:
Mar 21 21:29:39 localhost polkadot[475567]:    0: Backend error: Corruption: Bad column metadata
Mar 21 21:29:39 localhost polkadot[475567]: Backtrace omitted.
Mar 21 21:29:39 localhost polkadot[475567]: Run with RUST_BACKTRACE=1 environment variable to display it.
Mar 21 21:29:39 localhost polkadot[475567]: Run with RUST_BACKTRACE=full to include source snippets
```
4. Reverting again the binary to v0.9.18 we got the same error as before, by searching around we found that the error is the same as mentioned in the issue [#5168](https://github.com/paritytech/polkadot/issues/5168) described earlier, so as suggested, the value `version=6` was changed to `version=5` in the metadata filename here -> `/polkadot/chains/ksmcc3/paritydb/full/metadata`.

4. After the restart, the node started to sync and all seemed to be OK.

5. The same process was executed on the second node (turboflakes.io/MOMO). 
6. After the binary update and changing the `version` in the metadata filename the node started to sync and like in the first node, all seemed to be OK. A few errors in the logs, but these have already been seen by others and they seemed not to be raising to much alarm.
```
Mar 21 21:48:12 localhost polkadot[476261]: 2022-03-21 21:48:12 error=Runtime(RuntimeRequest(Execution { runtime_api_name: "SessionInfo", source: Application(UnknownBlock("State already discarded for BlockId::Hash(0x4bf39044ad1bccd388691a2189c7a321230c606d8b2688d5e9a7d8939691700b)")) }))
Mar 21 21:48:12 localhost polkadot[476261]: 2022-03-21 21:48:12 Dispute on candidate concluded with 'valid' result candidate_hash=0x03f8337cf9831ea4ca1fc3942acd31eb53c827f3235e49c881573f339b0614c3 session=20260
 
 ```
7. Unfortunately was already late and I couldn't stay for much longer so by the time I realised that one the nodes was _Chilled_ by the network because of an _Offline Event_ was when I realised that both of the nodes crashed - one of them was automatically _Chilled_ by the network showing the following error:
```
Mar 21 23:14:42 localhost polkadot[476261]: 2022-03-21 23:14:42 ✨ Imported #11911614 (0xfed5…2d54)
Mar 21 23:48:58 localhost polkadot[476261]: 2022-03-21 23:48:58 Locating closest peers for replication failed: Err(Timeout { key: Key(b"\xb55\xe4\xea\xfe\x9cN\x16N\x9c\xba\x05\xddk\x7f\xb7\xe4\x1e\xc73\xccd\x06$\x82\xc9\xb3\x8a\x02\xe1\x06["), success: [], quorum: 20 })
Mar 21 23:48:58 localhost polkadot[476261]: 2022-03-21 23:48:58 Locating closest peers for replication failed: Err(Timeout { key: Key(b"\xaa\xb4\xdc\x1a>\xedb\xec\x0e|\x0f\x8a\x82\xcbTm]\xf7\x95\x08\xe1\xf9V$\xe3X\x86\xaf\x8fw\xe56"), success: [], quorum: 20 })
Mar 21 23:48:58 localhost polkadot[476261]: 2022-03-21 23:48:58 Locating closest peers for replication failed: Err(Timeout { key: Key(b"\xbf\xe7\x02o\x03b\x13\xd45zh\x92\xbd\xaf/\xc5YX\x86\xfc\x1d\xa8W\x1aZ\xaa\xc7\xc0\x89\x99\xff\x86"), success: [], quorum: 20 })
Mar 21 23:48:58 localhost polkadot[476261]: 2022-03-21 23:48:58 Locating closest peers for replication failed: Err(Timeout { key: Key(b"\x8c\xc94\x81\xd8K\x0b\xc5\xa7^\x92\x04\x91\xca\x8f\n\xc8\x93u\x1f\xa2\x95\xe7\xd3\xc4J\xacpK\x94\xb2~"), success: [], quorum: 20 })
```
8. After a few restarts both nodes started to sync again. But since was late (fatigue was hitting), I than decided to revert back to previous v0.9.17 using one of out internal snapshots.
10. Our _ParityDB_ snapshot available was older than expected (more than 1 month older), but I tried to give it a go. After extracting the snapshot, syncing was taking longer than expected and another _Offline event_ was just hitting one of the nodes again.
11. Since to get a full sync from our internal snapshot would take at least a few more hours, I than decided for the first time to give it a try to one of the community snapshots available https://validators.polkachu.com/resources-and-links#node-snapshot. And also revert to the _RocksDB_ database.
12. I end up deciding to give it a try https://parasnaps.io/kusama. The reason of this choice, _parasnaps_ seems to be the one wher I could get my nodes live again within the shortest time.
13. After a few minutes by following the _interactive_ process as decribed [here](https://parasnaps.io/kusama#interactive). Kudos to _parasnaps.io_ by the really nice CLI by the way:
```Download s3get
Verify checksum

Select the Download source for the snapshot (closest to the host):
1) (Europe) Ireland
2) (Europe) Frankfurt
3) (US East) N. Virginia
4) (US West) Oregon
5) (Asia Pacific) Singapore
6) Quit
Please enter your choice: 3
you chose (US East) N. Virginia

Select the chain:
1) polkadot
2) kusama
3) Quit
Please enter your choice: 2
you chose kusama

Select the chain:
1) rocksdb
2) paritydb-experimental
3) Quit
Please enter your choice: 1
you chose rocksdb

Enter download directory [/root/.local/share/polkadot/chains/ksmcc3/]:/polkadot/chains/ksmcc3/
you chose /polkadot/chains/ksmcc3/

Downloading:
```
14. After downloading and updating the service `/etc/systemd/system/kusama-node.service` with the right flags. The node (turboflakes.io/MOMO) was syncing and all seemed to be fine. So I changed the validate intention and MOMO was already back on track.
15. I followed the same process on my other node (turboflakes.io/COCO). But on this one the download failed with the following error:
```
Downloading:
Error: Error during dispatch: connection closed before message completed
tar: Unexpected EOF in archive
tar: Unexpected EOF in archive
tar: Error is not recoverable: exiting now
```
16. After a few attempts of trying to download the snapshot - not sure yet why it failed - changing the drive to where the download was being done and worked. 
17. I than copied the DB folder to right path `/polkadot/chains/ksmcc3/db/`, updated the service `/etc/systemd/system/kusama-node.service` with the right flags and started the node.
18. Cool. The full sync was just minutes away to be completed and I was happy to get another node back on track. Validate intention performed and good to go.
19. After a few more minutes both nodes were fully operational with visible _ImOnline_ events and authored blocks performed.
20. By the end of these 2-3 hours journey both of my nodes already had 2 Offline events each.
21. In the end I was just happy how things turned out. The migration could have been handled much better now by looking to all of the decisions and actions performed described in these steps. But it could also end up really bad.
22. Special Kudos to @tuggy and @polkachu to make community snapshots a reliable source of information and tools. By easing the process of a node update when things get hot and not happen as initially expected.

Overall it was an enrichment experience :)

_Lessons learned_

- Before migration, plan ahead and review your migration process.
- Internal snapshots must not be older than 7 days. At least a weekly snapshot must be available.
- Try to book a 3 hours dedicated slot time for the migration process. One hour to review, prepare and perform migration and the next two hours for monitoring node and logs.
- Do not perform a migration on both nodes in the same time slot.
- Probably do not run `paritydb-experimental` in production nodes :)

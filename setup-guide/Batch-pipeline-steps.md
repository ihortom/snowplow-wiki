[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » Batch Pipeline Steps

*This page refers to Snowplow R102+*

*Click [here](Batch-pipeline-steps-r91) for the corresponding documentation for R91-R101*

*Click [here](Batch-pipeline-steps-r90) for the corresponding documentation for R90*

*Click [here](Batch-pipeline-steps-r89) for the corresponding documentation for R89*

*Click [here](Batch-pipeline-steps-r87) for the corresponding documentation for R87-R88*

*Click [here](Batch-pipeline-steps-r86) for the corresponding documentation for R86 and earlier*

## Dataflow diagram

[[/images/batch_pipeline_steps.png]]

## Recovery steps

The below table summarizes the actions to be taken at each particular step failure from the dataflow diagram above.

Failed step | Recovery actions
:---:|---
 1 | If no files have been moved yet (`raw:processing` [A] is empty), rerun the *EmrEtlRunner* as usual. If (on the other hand) some files have already been moved, rerun the *EmrEtlRunner* with `--skip staging` option to proceed with processing of those log files.
 2 | Rerun the *EmrEtlRunner* with `--skip staging` option.|
 3 | Rerun the *EmrEtlRunner* with `--skip staging` option.<br><br>**Note**: The `enriched:bad` [D] and `enriched:error` [E] could contain the files produced as a result of the step 3. Therefore rerunning the *EmrEtlRunner* could result in duplicated `bad`/`error` files. This could be significant if `elasticsearch` step [8-9] is engaged for examining `bad` data [D]. The outcome would be the same data timestamped with different time values by different EMR runs.
 4 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with either `--skip staging` option or with `--resume-from enrich`.
 5 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with either `--skip staging` option or with `--resume-from enrich`.
 6 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with either `--skip staging` option or with `--resume-from enrich`.
 7 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with either `--skip staging` option or with `--resume-from enrich`.<br><br>**Note**: The `enriched:bad` [D] and `shredded:bad` [H] could contain the files produced as a result of the step 3 and 6 respectively. Therefore rerunning the *EmrEtlRunner* could result in duplicated `bad` files. This could be significant if `elasticsearch` step (8-9) is engaged for examining `bad` data ([D],[H]). The outcome would be the same data timestamped with different time values by different EMR runs.
 8 | Delete `enriched:good` [F] and `shredded:good` [K]. Rerun the *EmrEtlRunner* with either `--skip staging` option or with `--resume-from enrich`.
 9 | Rerun the *EmrEtlRunner* with either `--skip staging,enrich,shred` option or with `--resume-from elasticsearch` (Elasticsearch is used) or `--resume-from archive_raw`.
 10 | If duplicated `bad` data is not critical rerun the *EmrEtlRunner* with `--skip staging,enrich,shred` option. If duplicated bad data **is** critical, *instructions to come ([#2593](https://github.com/snowplow/snowplow/issues/2593))*.<br><br>**WARNING**: In R90/R91, if you pass `--skip shred` to EmrEtlRunner then RDB Loader does not load unstructured events and contexts. This issue is resolved in R92.
 11 | If duplicated `bad` data is not critical rerun the *EmrEtlRunner* with `--skip staging,enrich,shred` option. If duplicated bad data **is** critical, *instructions to come ([#2593](https://github.com/snowplow/snowplow/issues/2593))*.<br><br>**WARNING**: In R90/R91, if you pass `--skip shred` to EmrEtlRunner then RDB Loader does not load unstructured events and contexts. This issue is resolved in R92.
 12 | Rerun the *EmrEtlRunner* with `--skip staging,enrich,shred,elasticsearch` option or `--resume-from archive_raw`.<br><br>**WARNING**: In R90/R91, if you pass `--skip shred` to EmrEtlRunner then RDB Loader does not load unstructured events and contexts. This issue is resolved in R92.
 13 | The data loads are wrapped in a single transaction, so an *RDB Loader* failure will not result in a partial load. However, if multiple data targets are used and some targets already been loaded, you may need to temporarily remove those from `config.yml` during your recovery process.<br><br>There are 3 stages in `rdb_load` step, namely "discover", "load", and "analize" (in that order). At the "discover" stage the availability of JSONPaths files are checked. After the data is loaded at "load" stage, the tables are analized to update table statistics for use by the query planner. To start *RDB Loader* from the beginning, use the `--resume-from rdb_load` option.<br><br>If the failure occurred at the analyze stage (i.e. after the data was successfully loaded), you can skip the analyze with the `--resume-from archive_enriched` option. To analyze, resume with `--resume-from analyze`.
 14 | Rerun the *EmrEtlRunner* with `--resume-from archive_enriched` option.
 15 | Rerun the *EmrEtlRunner* with `--resume-from archive_shredded` option.

## Dataflow diagram for Stream Enrich mode

[[/images/batch_pipeline_steps_r102_stream_mode.png]]

## Recovery steps for Stream Enrich mode

**IMPORTANT**. Due [a bug in R102][r102-shred-bug] Stream Enrich mode, `--resume-from shred` and `--skip staging_stream_enrich` do not work as expected and subsequent S3DiscCp step fails. This was addressed in R104.

Failed step | Recovery actions
:---:|---
 1 | If no files have been moved yet (`enriched:good` [A] is empty), rerun the *EmrEtlRunner* as usual. If, on the other hand, some files have already been moved, rerun the *EmrEtlRunner* with `--skip staging_stream_enrich` option to proceed with processing of those enriched files files.
 2 | Rerun the *EmrEtlRunner* with `--skip staging_stream_enrich` option.
 3 | Rerun the *EmrEtlRunner* with `--skip staging_stream_enrich` option.
 4 | Delete `shredded:good` [D]. Rerun the *EmrEtlRunner* with `--skip staging_stream_enrich` option.
 5 | You can ignore moving `_SUCCESS` file. Resume from step 6.
 6 | The data load cannot result in partial load due to the use of `COMMIT`. However, if more than one data target is used you would need to rerun the *EmrEtlRunner* with the successfully loaded target removed from the `config.yml` configuration file to retry loading the "failed" target.<br><br>**Note**: If the failure occurred at `analyze` stage, you can skip it with `--skip staging_stream_enrich,shred,rdb_load` option.
 7 | Rerun the *EmrEtlRunner* with `--skip staging_stream_enrich,shred,rdb_load` option.
 8 | Rerun the *EmrEtlRunner* with `--skip staging_stream_enrich,shred,rdb_load,` option.

 [r102-shred-bug]: https://github.com/snowplow/snowplow/issues/3722

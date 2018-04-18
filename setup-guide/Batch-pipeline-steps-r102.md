[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » Batch Pipeline Steps

*This page refers to Snowplow R102*

*Click [here](Batch-pipeline-steps) for the corresponding documentation for other releases*

## Dataflow diagram for Batch Enrich mode

[[/images/batch_pipeline_steps_r90.png]]

## Recovery steps for batch enrich mode

The below table summarizes the actions to be taken at each particular step failure in "batch enrich mode" from the dataflow diagram above.

Failed step | Recovery actions
:---:|---
 1 | If no files have been moved yet (`raw:processing` [A] is empty), rerun the *EmrEtlRunner* as usual. If (on the other hand) some files have already been moved, rerun the *EmrEtlRunner* with `--skip staging` option to proceed with processing of those log files.
 2 | Rerun the *EmrEtlRunner* with `--skip staging` option.|
 3 | Rerun the *EmrEtlRunner* with `--skip staging` option.<br><br>**Note**: The `enriched:bad` [D] and `enriched:error` [E] could contain the files produced as a result of the step 3. Therefore rerunning the *EmrEtlRunner* could result in duplicated `bad`/`error` files. This could be significant if `elasticsearch` step [8-9] is engaged for examining `bad` data [D]. The outcome would be the same data timestamped with different time values by different EMR runs.
 4 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with `--skip staging` option.
 5 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with `--skip staging` option.
 6 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with `--skip staging` option.<br><br>**Note**: The `enriched:bad` [D] and `shredded:bad` [H] could contain the files produced as a result of the step 3 and 6 respectively. Therefore rerunning the *EmrEtlRunner* could result in duplicated `bad` files. This could be significant if `elasticsearch` step (8-9) is engaged for examining `bad` data ([D],[H]). The outcome would be the same data timestamped with different time values by different EMR runs.
 7 | Delete `enriched:good` [F] and `shredded:good` [K]. Rerun the *EmrEtlRunner* with `--skip staging` option.
 8 | If duplicated `bad` data is not critical rerun the *EmrEtlRunner* with `--skip staging,enrich,shred` option. If duplicated bad data **is** critical, *instructions to come ([#2593](https://github.com/snowplow/snowplow/issues/2593))*.
 9 | If duplicated `bad` data is not critical rerun the *EmrEtlRunner* with `--skip staging,enrich,shred` option. If duplicated bad data **is** critical, *instructions to come ([#2593](https://github.com/snowplow/snowplow/issues/2593))*.
 10 | Rerun the *EmrEtlRunner* with `--skip staging,enrich,shred,elasticsearch` option.
 11 | The data load cannot result in partial load due to the use of `COMMIT`. However, if more than one data target is used you would need to rerun the *EmrEtlRunner* with the successfully loaded target removed from the `config.yml` configuration file to retry loading the "failed" target.<br><br>**Note**: If the failure occurred at `analyze` stage, you can skip it with `--skip staging,enrich,shred,archive_raw,rdb_load` option.
 12 | Rerun the *EmrEtlRunner* with `--skip staging,enrich,shred,archive_raw,rdb_load` option.

## Dataflow diagram for Stream Enrich mode

**TODO**

## Recovery steps for Stream Enrich mode

**IMPORTANT**. Due [a bug in R102][r102-shred-bug] Stream Enrich mode, `--resume-from shred` and `--skip staging_stream_enrich` do not work as expected and subsequent S3DiscCp step fails. This was addressed in R104.

Failed step | Recovery actions
:---:|---
 1 | If no files have been moved yet (`enriched:good` [A] is empty), rerun the *EmrEtlRunner* as usual. If (on the other hand) some files have already been moved, rerun the *EmrEtlRunner* with `--skip staging_stream_enrich` option to proceed with processing of those enriched files files.
 2 | Rerun the *EmrEtlRunner* with `--skip staging_stream_enrich` option.|
 7 | Delete `shredded:good` [K]. Rerun the *EmrEtlRunner* with `--skip staging_stream_enrich` option.|
 10 | Rerun the *EmrEtlRunner* with `--skip staging_stream_enrich,shred,elasticsearch` option|
 11 | The data load cannot result in partial load due to the use of `COMMIT`. However, if more than one data target is used you would need to rerun the *EmrEtlRunner* with the successfully loaded target removed from the `config.yml` configuration file to retry loading the "failed" target.<br><br>**Note**: If the failure occurred at `analyze` stage, you can skip it with `--skip staging_stream_enrich,shred,rdb_load` option.
 12 | Rerun the *EmrEtlRunner* with `--skip staging_stream_enrich,shred,rdb_load` option.

 [r102-shred-bug]: https://github.com/snowplow/snowplow/issues/3722

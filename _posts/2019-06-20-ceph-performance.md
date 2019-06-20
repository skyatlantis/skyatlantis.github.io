---
layout: post
title:  ceph-performance
date: 2019-06-20
categories: 分布式文件系统 性能
tags: 文件系统
excerpt: ceph-performance
---


ceph-performance
---

[global]
fsid = xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
mon_host = xxx.xxx.xxx.xxx
cluster_network = xxx.xxx.xxx.xxx/xx
public_network = xxx.xxx.xxx.xxx/xx
mon_max_pg_per_osd = 400
objecter_inflight_ops = 10240
objecter_inflight_op_bytes = 2147483648
osd_op_num_shards = 6
osd_op_num_threads_per_shard = 4
ms_async_op_threads = 6
ms_async_max_op_threads = 6
ms_crc_data = false
throttler_perf_counter = false
debug_none = 0/0
debug_lockdep = 0/0
debug_context = 0/0
debug_crush = 0/0
debug_mds = 0/0
debug_mds_balancer = 0/0
debug_mds_locker = 0/0
debug_mds_log = 0/0
debug_mds_log_expire = 0/0
debug_mds_migrator = 0/0
debug_buffer = 0/0
debug_timer = 0/0
debug_filer = 0/0
debug_striper = 0/0
debug_objecter = 0/0
debug_rados = 0/0
debug_rbd = 0/0
debug_rbd_mirror = 0/0
debug_rbd_replay = 0/0
debug_journaler = 0/0
debug_objectcacher = 0/0
debug_client = 0/0
debug_osd = 0/0
debug_optracker = 0/0
debug_objclass = 0/0
debug_filestore = 0/0
debug_journal = 0/0
debug_ms = 0/0
debug_mon = 0/0
debug_monc = 0/0
debug_paxos = 0/0
debug_tp = 0/0
debug_auth = 0/0
debug_crypto = 0/0
debug_finisher = 0/0
debug_reserver = 0/0
debug_heartbeatmap = 0/0
debug_perfcounter = 0/0
debug_rgw = 0/0
debug_civetweb = 0/0
debug_javaclient = 0/0
debug_asok = 0/0
debug_throttle = 0/0
debug_refs = 0/0
debug_xio = 0/0
debug_compressor = 0/0
debug_bluestore = 0/0
debug_bluefs = 0/0
debug_bdev = 0/0
debug_kstore = 0/0
debug_rocksdb = 0/0
debug_leveldb = 0/0
debug_memdb = 0/0
debug_kinetic = 0/0
debug_fuse = 0/0
debug_mgr = 0/0
debug_mgrc = 0/0
debug_dpdk = 0/0
debug_eventtrace = 0/0

[mon]
mon_allow_pool_delete = true
mgr_initial_modules = restful status dashboard balancer
mon_health_preluminous_compat_warning = false

[mgr]
mgr_op_latency_sample_interval = 300
mon_pg_warn_min_per_osd = 0
mon_warn_on_pool_no_app = false
rbd_perf_report_enabled = false

[osd]
osd_journal_size = 0
bluestore_block_db_size = 0
bluestore_block_wal_size = 0
osd_objectstore = bluestore
bluestore_throttle_bytes = 134217728
bluestore_csum_type = none
bluestore_cache_size = 1073741824
osd_client_message_cap = 1024
bluestore_rocksdb_options = compression=kNoCompression,max_write_buffer_number=8,min_write_buffer_number_to_merge=2,recycle_log_file_num=4,compaction_style=kCompactionStyleLevel,write_buffer_size=536870912,target_file_size_base=67108864,max_background_compactions=28,max_background_flushes=4,level0_file_num_compaction_trigger=8,level0_slowdown_writes_trigger=32,level0_stop_writes_trigger=64,num_levels=7,max_bytes_for_level_base=536870912,max_bytes_for_level_multiplier=4,compaction_readahead_size=2097152,compaction_threads=32,flusher_threads=8,rate_limiter_bytes_per_sec=10485760
osd_class_update_on_start = false
osd_scrub_max_interval = 0
osd_scrub_begin_hour = 50
osd_scrub_end_hour = 100
osd_max_backfills = 100
osd_recovery_sleep_hdd = 0
osd_recovery_sleep_hybrid = 0
bluestore_max_blob_size = 4194304
osd_client_message_size_cap = 2147483648
bluestore_prefetch_blob = true
bluestore_meta_fixed_length = 65536

# async recovery
osd_async_recovery = true
osd_async_recovery_min_cost = 0
osd_force_auth_primary_missing_objects = 0

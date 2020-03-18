# WAN acceleration

By combining multiple technologies such as network compression, multi-threading, dynamic TCP window size, variable block size deduplication and global caching, WAN acceleration provides sufficient capability when the network bandwidth is low or limited when performing Backup Copy and Replication jobs. This technology is specifically designed to accelerate Veeam job. Any other WAN acceleration technology should be disabled for Veeam traffic.
Starting with v10 Veeam Backup and Replication supports two operational modes for WAN accelerators: Low bandwidth mode for links with less than 100 Mbps throughput and High bandwidth mode for links up to 500-700 Mbps throughput. The operational mode can be controlled in WAN accelerator properties.
WAN accelerators work in pairs: one WAN accelerator should be deployed at the source side and another at the target site. Many to one scenario is also possible but needs accurate sizing and has higher system requirements.
*High bandwidth mode doesn`t utilize “Global cache” If you are not sure about the mode you are going to use, choose to work in Low bandwidth to ensure global cache folder creation on the target Wan accelerator, otherwise when switching from High to Low mode, the job may fail due to the absence of target cache.
Configuration and sizing
When configuring the WAN accelerator, not all configuration parameters affect both source and target WAN accelerators. In this section we will highlight what settings should be considered on each side.

---
title: Analyzing WAN acceleration workload
parent: Analyzing WAN acceleration workload
grand_parent: 4-Operate
nav_order: 10
---


Operate 
Primary backup job Mode 
When used with Backup Copy job it is important to remember that Backup copy job works in forever incremental mode, transferring to the target repository only the differences between previous and current job cycles no matter what type of the backup file is in the primary repository: Full or Incremental. 
When planning for WAN acceleration with Backup Copy job, review the backup mode used on the primary backup job. Some backup methods result in a random I/O workload on the source repository (as opposed to sequential I/O patterns in other backup modes). Thus, the retrieval of needed blocks takes longer. The methods of reading from source is illustrated by the figure below:






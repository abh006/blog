---
title: Redis Keeps Going Down
description: One replica or another was always down
date: 18 Nov 2022
draft: false
---

After some log checking i figured out that the reason was that Redis was unable to read the complete AOF file.

### No way to know the size of the AOF file

The pod is down. Tried scaling up the pod memory to 24GB. But the pod won’t scale. The autoscaling group in AWS had 3 instance types one of which had 33GB memory capacity. But the cluster-autoscaler was not using it.

### Seeking for the why

After a long session of google search and reading (or surfing through) whatever I found, Sid asked me to stick to the documentation. There it was. Autoscaling group must consist of similar sized nodes. Mine included an m5.xlarge and m5.2xlarge. Removed the m5.2xlarge and created a new node-group for that. Cluster-autoscaler was happy. It started scaling to my requirement.

### Redis master was up.

Still had issues with the replicas. The persistence mode was AOF, which basically logs all the commands that are run in the DB, to re-run again to create the latest version of DB. In the /data folder of the redis pod, there was the AOF file with 240GB and the db snapshot RDB file with just 5.4GB.

Deleted the release using helm. Updated the redis helm chart with a `commonConfiguration` field disabling AOF and enabling RDB mode of persistence (which loads from the snapshot each time the pod starts up). All the pods were up. The startup time for them were around 90-95s instead of the “forever” seconds.

Replicas are now being synchronised from the master. I hope everything is alright.

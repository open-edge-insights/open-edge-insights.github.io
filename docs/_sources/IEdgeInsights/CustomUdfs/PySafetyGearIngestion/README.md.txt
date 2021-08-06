## Native Safety Gear Ingestion

This container is based out of VideoIngestion container. It doesn't need any pre-processing udf hence we have used dummy udf which is part of VideoIngestion container.
In case user wants to add any pre-processing udf, then the dummy udf can be replaced with the correspomding pre-processsing udf.

Refer [udfs-README](../../common/video/udfs/README.md) for more information on configs of other in-built udfs like dummy, fps and resize udfs.

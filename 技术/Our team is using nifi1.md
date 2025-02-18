Our team is using nifi1.26.0. When using UpdateRecord processor or other processor sets that inherit org.apache.nifi.processors.standard.AbstractRecordProcessor, we found that its processing performance dropped by more than 50%. After our analysis, we found that in the Parser object created by org.apache.nifi.json.JsonParserFactory, an ObjectMapper object is created when processing each data, which consumes a lot of performance.

![img](blob:https://issues.apache.org/3657b4e8-1d9f-48fa-9d3f-762c6d067311)

 

ObjectMapper is a heavyweight object and is thread-safe. It seems that a static global ObjectMapper should be created.

We have run the performance tests and found that it was useful，here is the compare results：

BEFORE：

![img](blob:https://issues.apache.org/25cc6456-b051-4b56-9fa2-e128b1720dec)

AFTER： 

![img](blob:https://issues.apache.org/6b39e9b5-a28d-42ca-b36b-11d3b1861989)

When each JSON data is large, the gap will be more obvious！！！！
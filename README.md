# Kafka
This repo is for solving kafka related problem
## FETCH_SESSION_ID_NOT_FOUND
Error:  org.apache.kafka.clients.FetchSessionHandler : [Consumer clientId=xxx, groupId=xxx] Node x was unable to process the fetch request with (sessionId=224593981, epoch=969): FETCH_SESSION_ID_NOT_FOUND.
### Questions
1. What is a fetch request?
1. What is a fetch session ?
2. The number of session depend on what?
3. What is a fetch session id ?
4. What is fetch epoch ?
5. Who decide how long session is alive ? What conf ?
6. Best pratice to configure MAX_FETCH_SESSION

### Answers
1. Fetch requests:

    Sent by consumers and follower replicas when they read messages from Kafka
brokers. 
* Consumers send fetch requests to fetch data from topics
* Follower replicas send fetch requests to keep in-sync with master node
  
1. Fetch session: 
    A fetch session encapsulates the state of individual fetcher. This allows kafka to avoid resending this state as part of each fetch request.
    The fetch session includes: 
    1. A randomly generated 32bit session id which is unique on the leader
    2. The 32bit fetch epoch
    3. Cached data about each partition which the fetchers is interested in
    4. The privileged bit: is set if the fetch session was created by a follower. It is cleared if the fetch session as created by a reguler consumer.
    5. The time when the fetch session was last used: This is the time in wall-clock milliseconds when the fetch session was last used. This is used to expire fetch sessions after they have been inactive.
    Cache session data about each partition:
    1. Topic name
    2. The partition index
    3. The maximum number of bytes to fetch from this partition
    4. The fetch offset
    5. The high water mark
    6. The fetcher log start offset
    7. The leader log start offset
    **The leader uses this cached information to decide which partitions to include in FetchReponse. Whenever any of these element change, or if there is new data available for a partition, the partition will be include**
1. The number of fetch session:
        As you see, the number of fetch sessions are depend on the number of regular consumers and the number of follower. nConsumers + nReplicas
        For example: With 3 brokers, 2k topics, 4k partitions(2 for each), 8k replicas(2 for each). The number of consumer should be less then the number of partitions. nSession <= 4k + 9k = 12k
        
3. Fetch session id:
    A randomly generated 32bit session id which is unique on the leader. It is unique, immutable identifier for each session.
    Since IDs is randomly generated, it cannot leak information to unpriveleged clients.
1. Fetch epoch: 
    The fetch epoch is monotonically incrementing 32bit counter. After processing request N, the brokers expects to receive request N + 1.
    The sequence number is always greater then 0. After reaching MAX_INT, it wraps around to 1.

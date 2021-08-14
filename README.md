# Kafka
This repo is for solving kafka related problem
## FETCH_SESSION_ID_NOT_FOUND
Error:  org.apache.kafka.clients.FetchSessionHandler : [Consumer clientId=xxx, groupId=xxx] Node x was unable to process the fetch request with (sessionId=224593981, epoch=969): FETCH_SESSION_ID_NOT_FOUND.
### Questions
1. What is a fetch request?
1. What is a fetch session ?
1. What is a fetch session id ?
1. What is fetch epoch ?
3. Who decide how long session is alive ? What conf ?
4. Best pratice to configure MAX_FETCH_SESSION

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
    4. The privileged bit
    5. The time when the fetch session was last used

1. Fetch session id:
    A randomly generated 32bit session id which is unique on the leader. It is unique, immutable identifier for each session.
    Since IDs is randomly generated, it cannot leak information to unpriveleged clients.
1. Fetch epoch: 
    The fetch epoch is monotonically incrementing 32bit counter. After processing request N, the brokers expects to receive request N + 1.
    The sequence number is always greater then 0. After reaching MAX_INT, it wraps around to 1.

https://access.redhat.com/articles/3009361


# Relevant aspects to check for a migration from JBoss Fuse 6 to JBoss Fuse 7
- ports
- cluster usage
- threads
- jetty
- ConfigMap (can't use all chars)
- Logging (has to go to stdout, no multiple ones)
- password --> secrets
- servicewrapper -> k8s service ready/heath check
- zookeeper
- activemq
- standalone master/slave
- gateway
- jolokia --> expose service

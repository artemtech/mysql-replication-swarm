# MySQL Docker Swarm Master Slave Replication

## Requirements
- docker
- minimum 2 node server with port 25060 open


## Usage
**Assigning servers to swarm nodes**
```bash
(node 1) `docker swarm init --advertise-addr 192.168.1.100`
# sample output
Swarm initialized: current node (bvz81updecsj6wjz393c09vti) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-3pu6hszjas19xyp7ghgosyx9k8atbfcr8p2is99znpy26u2lkl-1awxwuwd3z9j1z3puu7rcgdbx \
    192.168.1.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

(node 2) docker swarm join \
    --token SWMTKN-1-3pu6hszjas19xyp7ghgosyx9k8atbfcr8p2is99znpy26u2lkl-1awxwuwd3z9j1z3puu7rcgdbx \
    192.168.1.100:2377

adjust 192.168.1.100 with your NODE1 ip address
```
**Assigning 2 nodes to act as manager for HA**
```bash
(node 1) docker node ls
# output
ID                            HOSTNAME    STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
nerx2ow5tmwrea3lhjb9ecscg *   node1       Ready     Active         Reachable        20.10.1
m85uv0k6d55c8dpuapiv1n6lx     node2       Ready     Active                          20.10.0

(node 1) docker node promote node2
(node 1) docker node ls
# output
ID                            HOSTNAME    STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
nerx2ow5tmwrea3lhjb9ecscg *   node1       Ready     Active         Reachable        20.10.1
m85uv0k6d55c8dpuapiv1n6lx     node2       Ready     Active         Reachable        20.10.0
```
**Assigning node labels for master and slave**
```bash
(node 1) docker node update --label-add database=master node1
(node 1) docker node update --label-add database=slave node2
```
**Time for cluster up!**
```bash
# preparing directories for volume binding
(node 1) sudo mkdir -pv /data/mysql/master && sudo chown 1001:1001 /data/mysql/master
(node 2) sudo mkdir -pv /data/mysql/slave && sudo chown 1001:1001 /data/mysql/slave

# lets rock it!
(node 1) docker stack deploy -c mysql mysql
# wait for 10-30s until master database ready to use, you can check it by running this command:
# docker service logs mysql_master

# time to fire up slave database
(node 1) docker service scale mysql_slave=1
```

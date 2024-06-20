# Docker Swarm tips
```
// Restart all tasks with force
docker service update --force <service_name>

// Explore a service configuration
docker service inspect <service_name>
..
"Mode": {
     "Replicated": {
         "Replicas": 15
          }
     },
So this is global mode presented, if there are 5 nodes, 3 tasks will run on each node
 
// Show serivce tasks, on which nodes they are running, etc..
docker service  ps <service_name>

// Redeploy
docker service rm <service_name> && docker stack deploy --prune --with-registry-auth --compose-file /opt/docker/<service_name>.yml <STACK_name>
```

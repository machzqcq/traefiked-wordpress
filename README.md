# traefiked-wordpress

This *example project* will create four services in a Docker swarm mode cluster:
* Two Wordpress services for two different wordpress sites (beluga and humpback), with labels for Traefik to load-balance
* One MariaDB database configured for Galera-based clustering using swarm mode DNS for discovery
* One Traefik proxying/load balancing container 

Note this is for demonstration purposes, only. To use this in
production, data for Traefik, WordPress, and MariaDB needs to be
stored in appropriate durable Docker Volumes

# Usage

These services can be started using the following command:
    
```
docker stack deploy --compose-file docker-compose.yml traefiked-wordpress
```

This will bring up 2 wordpress containers, a single dbcluster
container, and a traefik container.

## Testing
In this example, Traefik is configured to provide hostname-based
HTTP proxying for 2 WordPress sites: beluga and humpback. You'll
probably need to edit your hosts file to define these 2 hosts,
probably to 127.0.0.1 if you are running the composition locally
through Docker for (Mac,Windows,Linux). If you're running the stack
somewhere other than on your local machine, hopefully you'll know
the IP address that you're connecting to, and can use that in your
hosts file. Rackspace has a nice set of instructions
[here](https://support.rackspace.com/how-to/modify-your-hosts-file/) to
edit your hosts file for various OSes.

When finished, the following command shuts everything down:

```
docker service rm traefiked-wordpress_dbcluster traefiked-wordpress_beluga-wordpress traefiked-wordpress_traefik traefiked-wordpress_humpback-wordpress
```

# Productionizing

If you would like to do the same on aws and get closer to production, then do the following

- Install docker swarm on aws - see the [article](https://www.linkedin.com/pulse/docker-swarm-aws-pradeep-macharla)
- You can scale the nodes in the Swarm by updating the CF stack and increasing managers/workers - If you want to do this programmatically, then call into AWS API's for CF and have the triggers defined in your own program
- Any ports published on the Swarm are published on the ELB (which is a resource created as part of CF stack). Random port publishing is not discovered by ELB and hence it leaves those
- In our case we want the traffic to be routed based on rule set defined in traefik - hence any request that comes from DNS -> Route 53 Hosted Zone -> ELB -> Traefik (port 80) 
- From here on its Traefik that manages the routing
- So if we hit beluga.seleniumframework.com, then Traefik will route it to "beluga" service deployed on the swarm, by looking at the host header (The host header is set as a docker label for the beluga service in the compose file)
- Similarly if we hit humpback.seleniumframework.com, then Traefik will route it to "humpback" service deployed on the swarm, by looking at the host header (The host header is set as a docker label for the humpback service in the compose file)
- You can directly publish a SERVICE PORT to the Swarm and then ELB discovers that and listens - however what you will not get is the granular routing based on many matching conditions - that traefik can do. (Based on host headers and how it automatically adds the scaled service container IPs to the BACKEND it defines)



# References
This project uses the following Docker images:
* PHP 7.1/Apache version of the [Official Docker Wordpress image](https://hub.docker.com/_/wordpress/)
* ToughIQ's [MariaDB Galera Cluster image](https://hub.docker.com/r/toughiq/mariadb-cluster/)
* The [Traefik](https://hub.docker.com/_/traefik/) load balancer

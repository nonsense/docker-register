### Changes to https://github.com/jwilder/docker-register

`docker-register` registers etcd configuration in the following format:

      etcdctl set /backends/service_name/1/container_a "127.0.0.1:8000" --ttl 60
      etcdctl set /backends/service_name/1/container_b "127.0.0.2:8000" --ttl 60
      etcdctl set /backends/service_name/1/port "8005" --ttl 60
      etcdctl set /backends/service_name/2/container_a "127.0.0.1:8001" --ttl 60
      etcdctl set /backends/service_name/2/container_b "127.0.0.2:8001" --ttl 60
      etcdctl set /backends/service_name/2/port "8006" --ttl 60

rather than the original:

      etcdctl set /backends/service_name/container_a "127.0.0.1:8000" --ttl 60
      etcdctl set /backends/service_name/container_b "127.0.0.2:8000" --ttl 60
      etcdctl set /backends/service_name/port "8005" --ttl 60

which basically provides multi-port support / registration.

------

docker-register sets up a container running [docker-gen][1].  docker-gen dynamically generate a
python script when containers are started and stopped.  This generated script registers the running
containers host IP and port in etcd with a TTL.  It works in tandem with docker-discover which
generates haproxy routes on the host to forward requests to registered containers.

Together, they implement [service discovery][2] for docker containers with a similar architecture
to [SmartStack][3]. docker-register is analagous to [nerve][4] in the SmartStack system.

See also [Docker Service Discovery Using Etcd and Haproxy][5]

### Usage

To run it:

    $ docker run -d -e HOST_IP=1.2.3.4 -e ETCD_HOST=1.2.3.4:4001 -v /var/run/docker.sock:/var/run/docker.sock -t jwilder/docker-register

Then start any containers you want to be discoverable and publish their exposed port to the host.

    $ docker run -d -P -t ...

If you run the container on multiple hosts, they will be grouped together automatically.

### Limitations

There are a few simplications that were made:

* *Containers can only expose one port* - This is a simplification but if the container `EXPOSE`s
multiple ports, it won't be registered in etcd.
* *Exposed ports must be unique to the service* - Each container must expose it's service on a unique
port.  For example, if you have two different backend web services and they both expose their service
over port 80, then one will need to use a port 80 and the other a different port.


[1]: https://github.com/jwilder/docker-gen
[2]: http://jasonwilder.com/blog/2014/02/04/service-discovery-in-the-cloud/
[3]: http://nerds.airbnb.com/smartstack-service-discovery-cloud/
[4]: https://github.com/airbnb/nerve
[5]: http://jasonwilder.com/blog/2014/07/15/docker-service-discovery/

### TODO

* Support http, udp proxying
* Support multiple ports
* Make ETCD prefix configurable
* Support other backends (consul, zookeeper, redis, etc.)

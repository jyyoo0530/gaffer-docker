What you have here, is a Docker container for Gaffer.  This is a small
instance and is really only useful only development / trial
purposes.  But containerising allows a quick way to find out what Gaffer is
like and develop against the interfaces.  To run Gaffer, you need:
- Wildfly, hosting the Gaffer REST API and UI.
- Accumulo, hosting the Gaffer extensions.
- Zookeeper, which is used by Accumulo.
- Hadoop, which is used by Accumulo.

The code here:
- Compiles Gaffer from source in a build container, and extracts a set of
  things to be loaded into the deployable containers.
- Downloads Wildfly, and creates a container integrating Wildfly and Gaffer.
- Creates a container integrating Accumulo (cybermaggedon/accumulo) and
  Gaffer.

The memory settings of Accumulo are low to ensure Accumulo runs in a
small footprint.

To run:

```
  # Run Hadoop
  docker run --rm --name hadoop cybermaggedon/hadoop:2.7.3

  # Run Zookeeper
  docker run --rm --name zookeeper cybermaggedon/zookeeper:3.4.9

  # Run Accumulo
  docker run --rm --name accumulo --link zookeeper:zookeeper \
        --link hadoop:hadoop cybermaggedon/accumulo-gaffer:0.4.4

  # Run Wildfly, exposing port 8080.
  docker run --rm --name wildfly --link zookeeper:zookeeper \
    --link hadoop:hadoop --link accumulo:accumulo \
    -p 8080:8080 cybermaggedon/wildfly-gaffer:0.4.4

You can then access the Gaffer API at port 8080, e.g. try accessing URL
http://HOSTNAME:8080/rest.  The UI is available at http://HOSTNAME:8080/ui.

When the container dies, the data is lost.  If you want data to persist,
put volumes on /data for Hadoop and Zookeeper.

```
  # Run Hadoop
  docker run --rm --name hadoop -v /data/hadoop:/data cybermaggedon/hadoop:2.7.3

  # Run Zookeeper
  docker run --rm --name zookeeper /data/zk:/data cybermaggedon/zookeeper:3.4.9

  # Run Accumulo
  docker run --rm --name accumulo --link zookeeper:zookeeper \
        --link hadoop:hadoop cybermaggedon/accumulo-gaffer:0.4.4

  # Run Wildfly, exposing port 8080.
  docker run --rm --name wildfly --link zookeeper:zookeeper \
    --link hadoop:hadoop --link accumulo:accumulo \
    -p 8080:8080 cybermaggedon/wildfly-gaffer:0.4.4
```

The default schema deployed is usable.  If you want to set your own schema
then you need to change /usr/local/wildfly/schema/* by e.g. mounting
replacement volumes.

Accumulo makes considerable use of Zookeeper, to the point that, at high
input load, Zookeeper seems to continually grow until it exhausts the
container memory footprint.  Workaround: run containers in a container engine
like Kubernetes, so that everything restarts.




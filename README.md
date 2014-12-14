# TeamCity Continuous Integration server and agent in Docker

This project contains two Docker containers for deployment [TeamCity](https://www.jetbrains.com/teamcity/) server and agents.
It also could be running  on [CoreOS](https://coreos.com) cluster.

## How to use

To start, we need:

- TeamCity server container
- TeamCity build agents containers
- [PostgreSQL](https://hub.docker.com/u/clayman/postgresql/) to store TeamCity data (optional)

### Start TeamCity on Ubuntu

#### Start TeamCity server

If you running Docker container on Ubuntu or another Linux, just run:

    docker run -d --name teamcity_server -p 8111:8111 clayman/teamcity_server

#### Start TeamCity build agent

To start TeamCity build agent on same machine as server, just run:

    docker run -d --name teamcity_agent --link teamcity_server:teamcity_server -e TEAMCITY_SERVER=http://teamcity_server:8111 clayman/teamcity_agent

### Start TeamCity on CoreOS cluster

If you want to run TeamCity server and build agents on CoreOS cluster, you should create service configuration files

#### Start TeamCity server

Create `teamcity-server.service` service configuration for TeamCity server

    [Unit]
    Description=TeamCity Continuous Integration server

    # Requirements
    Requires=etcd.service
    Requires=docker.service

    # Dependency ordering
    After=etcd.service
    After=docker.service

    [Service]
    # Let processes take awhile to start up (for first run Docker containers)
    TimeoutStartSec=0

    # Change killmode from "control-group" to "none" to let Docker remove
    # work correctly.
    KillMode=none

    # Get CoreOS environmental variables
    EnvironmentFile=/etc/environment

    # Pre-start and Start
    ExecStartPre=-/usr/bin/docker pull clayman/teamcity_server:9.0
    ExecStartPre=-/usr/bin/docker create --name teamcity-server -p ${COREOS_PUBLIC_IPV4}:8111:8111 clayman/teamcity_server
    ExecStart=/usr/bin/docker start -a teamcity-server

    # Stop
    ExecStop=/usr/bin/docker stop teamcity-server

    [X-Fleet]
    Conflicts=teamcity-server.service

After this, let's load and start service

    $ fleetctl submit teamcity-server.service
    $ fleetctl load teamcity-server.service
    $ fleetctl start teamcity-server.service

Now TeamCity server will be available at `http://{COREOS_PUBLIC_IPV4}:8111`

#### Start TeamCity build agent

Create `teamcity-agent.service` service configuration for TeamCity build agent

    [Unit]
    Description=TeamCity Continuous Integration build agent

    # Requirements
    Requires=etcd.service
    Requires=docker.service
    Requires=teamcity-server.service

    # Dependency ordering
    After=etcd.service
    After=docker.service
    After=teamcity-server.service

    [Service]
    # Let processes take awhile to start up (for first run Docker containers)
    TimeoutStartSec=0

    # Change killmode from "control-group" to "none" to let Docker remove
    # work correctly.
    KillMode=none

    # Get CoreOS environmental variables
    EnvironmentFile=/etc/environment

    # Pre-start and Start
    ExecStartPre=-/usr/bin/docker pull clayman/teamcity_agent
    ExecStartPre=-/usr/bin/docker create --name teamcity-agent -e TEAMCITY_SERVER=http://teamcity_server:8111 clayman/teamcity_agent
    ExecStart=/usr/bin/docker start -a teamcity-agent

    # Stop
    ExecStop=/usr/bin/docker stop teamcity-agent

    [X-Fleet]
    MachineOf=teamcity-server.service

After this, let's load and start service

    $ fleetctl submit teamcity-server.service
    $ fleetctl load teamcity-server.service
    $ fleetctl start teamcity-server.service

This will start one build agent for your TeamCity server

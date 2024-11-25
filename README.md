- [Introduction](#introduction)
- [Installing WSL](#installing-wsl)
- [Sample ROS Autonomous Driving Project in Docker](#sample-ros-autonomous-driving-project-in-docker)
- [Kuksa](#kuksa)
  - [Sample Kuksa Project:](#sample-kuksa-project)
  - [A Sample Python Project Creating and Subscribing to a Kafka Topic](#a-sample-python-project-creating-and-subscribing-to-a-kafka-topic)
  - [Vehicle Signal Specification (VSS) Datatypes:](#vehicle-signal-specification-vss-datatypes)
- [Autoware](#autoware)
- [Ankaios:](#ankaios)
  - [Install Podman:](#install-podman)
  - [Create OCI Image from Docker Instance (via Docker2OCI tool)](#create-oci-image-from-docker-instance-via-docker2oci-tool)
  - [A sample Ankaios Project](#a-sample-ankaios-project)
- [Autowrx Gitlab Repository](#autowrx-gitlab-repository)
- [ThreadX](#threadx)
- [Uprotocol](#uprotocol)
- [Influxdb Connection](#influxdb-connection)
- [TODOS](#todos)


## Introduction

This document aims to provide an insight into the installation and usage of various tools and technologies for creating a simulation environment for autonomous systems and IoT projects. It covers the setup and integration of tools like Windows Subsystem for Linux (WSL), Nvidia Docker, ROS, Kuksa, Autoware, and Ankaios, allowing users to:

- Simulate and control autonomous vehicles using containerized environments.
- Process and communicate vehicle signal data through standards like VSS with Kuksa.
- Manage fleets using Ankaios and OCI-compatible tools.

## Installing WSL
In order to run a container based simulation environment there is a need to access to Nvidia GPU and setup Nvidia docker run time.
It is easier to setup a WSL for Nvidia GPU configuration as it by default has direct access to sources of the host (compared to VM setup).

To install WSL the following tutorial can be followed:
[Installing WSL on Windows 10](https://www.youtube.com/watch?v=X-DHaQLrBi8)

## Sample ROS Autonomous Driving Project in Docker

After setting up Nvidia Docker run-time,

A sample autonomous driving project can be initialized with:

```bash
docker pull noshluk2/ros2-self-driving-car-ai-using-opencv:latest

docker run -it \
    --name=ros2_sdc_container \
    --env="DISPLAY=$DISPLAY" \
    --env="QT_X11_NO_MITSHM=1" \
    --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
    --env="XAUTHORITY=$XAUTH" \
    --volume="$XAUTH:$XAUTH" \
    --net=host \
    --privileged \
    noshluk2/ros2-self-driving-car-ai-using-opencv \
    bash

# Bridging the environment with Prius Car
ros2 launch self_driving_car_pkg world_gazebo.launch.py

# In another terminal run:
 docker exec -it ros2_sdc_container bash

cd ~/ROS2-Self-Driving-Car-AI-using-OpenCV/
ros2 run self_driving_car_pkg computer_vision_node # Self driving mode
```

[![Watch the video](./AutonomousDriving.png)](./AutonomousDriving.mp4)
## Kuksa

[Eclipse Kuksa Quick Start](https://eclipse-kuksa.github.io/kuksa-website/quickstart/)

> **Important Note:** Based on the documentation the original port to publish Kuksa.Val serveris 55555, however it doesn't work, using port 55556 is an alternative.

### Sample Kuksa Project:
```bash
 docker run -it --rm  --publish 55556:55556 ghcr.io/eclipse/kuksa.val/databroker:master  --port 55556 --insecure
```

```bash
# To install Kuksa Python library:
pip install kuksa_client
```

### A Sample Python Project Creating and Subscribing to a Kafka Topic

```python
#!/usr/bin/env python3

# sample_kuksa.py
# The following project is executed using version 0.4.3

from kuksa_client.grpc import VSSClient
from kuksa_client.grpc import Datapoint

import time

with VSSClient('127.0.0.1', 55556) as client: # Attach to local host port 55556 to listen kuksa.val server
    for speed in range(0, 100):
        client.set_current_values({
            'Vehicle.Speed': Datapoint(speed),
        })
        print(f"Feeding Vehicle.Speed to {speed}")

        # Read back the value to confirm it was set correctly
        response = client.get_current_values(['Vehicle.Speed'])
        if 'Vehicle.Speed' in response:
            read_back_speed = response['Vehicle.Speed'].value
            print(f"Read back Vehicle.Speed: {read_back_speed}")
        else:
            print("Failed to read back Vehicle.Speed")

        time.sleep(1)
print("Finished.")

```

```bash
usr@usr:~$ chmod +x ./sample_kuksa.py
usr@usr:~$ python3 ./sample_kuksa.py

# Expected output:
Feeding Vehicle.Speed to 0
Read back Vehicle.Speed: 0.0
Feeding Vehicle.Speed to 1
Read back Vehicle.Speed: 1.0
Feeding Vehicle.Speed to 2
Read back Vehicle.Speed: 2.0
Feeding Vehicle.Speed to 3
Read back Vehicle.Speed: 3.0
...
```

###  Vehicle Signal Specification (VSS) Datatypes:
Kuksa supports VSS data type specification. For detailed documentation on VSS data types and definitions please refer to:

- [Vehicle Signal Specification Data Types](https://covesa.github.io/vehicle_signal_specification/rule_set/data_entry/data_types/)

## Autoware

[Autoware Documentation](https://autowarefoundation.github.io/autoware-documentation/main/installation/autoware/docker-installation/):

- Installation Process
```bash
# Clone autowarefoundation/autoware and move to the directory.

git clone https://github.com/autowarefoundation/autoware.git
cd autoware
# The setup script will install all required dependencies with Ansible:

./setup-dev-env.sh -y docker
# To install without NVIDIA GPU support:

./setup-dev-env.sh -y --no-nvidia docker
```

In case you get such error "NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver." Please follow the video:

- [NVIDIA-SMI Installation](https://www.youtube.com/watch?v=bhPCjPdqDEM)

## Ankaios:
- [Ankaios Quick Start](https://eclipse-ankaios.github.io/ankaios/0.1/usage/quickstart/)
- [Sample Fleet Management Project via MQTT Broker](https://eclipse-ankaios.github.io/ankaios/0.5/usage/tutorial-fleet-management/)

> **Important Note:** Ankaios uses OCI image format. Hence it leverages Podman as container manager.

### Install Podman:

To install Podman on Ubunutu the following tutorial can be followed:
- [Podman Installation](https://docs.vultr.com/how-to-install-podman-on-ubuntu-24-04)

```bash
sudo sh -c "echo 'deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_${VERSION_ID}/ /' > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list"
wget -nv https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable/xUbuntu_${VERSION_ID}/Release.key -O- | sudo apt-key add -
sudo apt-get update -qq
sudo apt-get -qq --yes install podman
```
### Create OCI Image from Docker Instance (via Docker2OCI tool)

It is possible to convert an existing Docker format image to OCI image format.
Following tutorial shows the steps on how to convert a Docker image format into OCI image format which Ankaios supports.

- [Docker2OCI](https://github.com/coolljt0725/docker2oci)
  -  Supports Docker2OCI conversion
  -  Supports validation of an Image based on OCI formatting

### A sample Ankaios Project

```yaml
# state.yaml
apiVersion: v0.1
kind: Config
workloads:
  nginx:
    runtime: podman
    agent: agent_A
    restart: true
    updateStrategy: AT_MOST_ONCE
    accessRights: #
      allow: []
      deny: []
    tags:
      - key: owner
        value: Ankaios team
    runtimeConfig: |
      image: docker.io/nginx:latest
      ports:
      - containerPort: 80
        hostPort: 8081
```

```bash
ank-server -k  --startup-config state.yaml # Start ankaios server with  provided configuration
ank -k apply state.yaml # Apply the workload

ank -k  get workload # Get all workload states
ank -k get state # Get detailed state about all workload states
```

## Autowrx Gitlab Repository
- [Autowrx Repository](https://gitlab.eclipse.org/eclipse/autowrx)
- [Sample Autoware Project](https://www.youtube.com/watch?v=iW-a7cKUxuY)
## ThreadX
- [ThreadX - Getting Started](https://github.com/eclipse-threadx/getting-started)
## Uprotocol
- [Eclipse Uprotocol](https://github.com/eclipse-uprotocol)
- [upython](https://github.com/eclipse-uprotocol/up-python)
- [up-transport-zenoh-python](https://github.com/eclipse-uprotocol/up-transport-zenoh-python)
## Influxdb Connection

[Kubernetes InfluxDB Setup](https://medium.com/starschema-blog/monitor-your-infrastructure-with-influxdb-and-grafana-on-kubernetes-a299a0afe3d2)

## TODOS
- [ ] For IOT tracability demonstrate ROS/Autoware topic subscription and data visualization through Influxdb / Grafana

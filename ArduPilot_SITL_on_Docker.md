# Deploying ArduPilot in a Docker Environment

This guide explains the process of creating a Docker container that runs ArduPilot SITL and connecting to the simulator from Mission Planner and Dronekit running outside the container.

---

## Intuition

Running ArduPilot SITL inside Docker provides a clean, reproducible simulation environment that behaves the same across different machines. Instead of installing ArduPilot’s build tools and dependencies directly on your system, Docker encapsulates everything into a single container. This makes setups easy to share and allows you to reset or rebuild the environment at any time without affecting your host system.

This guide describes a workflow that starts from a minimal Linux container and uses Docker to execute the ArduPilot installation and build process a single time, resulting in an image with the SITL fully installed and configured. This image then becomes a reusable artifact to the team: rather than reinstalling dependencies and rebuilding ArduPilot on every machine, it can be distributed and deployed directly, allowing a ready-to-run SITL instance to be launched instantly.

---

## 1. Install Docker Desktop

Head over to the [Docker Desktop website](https://www.docker.com/products/docker-desktop/) and follow the instructions to install for either Windows, MacOS or Linux.

**NOTE: After installation, briefly open Docker Desktop to ensure Docker is running on your computer!**

---

## 2. Create a Dockerfile from Template

On your host machine, create a new file called **Dockerfile** (no extention) and paste in the template provided below. As you'll notice, this dockerfile closely follows the instructions for installing ArduPilot SITL on Linux available **HERE**. Besides installing ArduPilot SITL inside the container, it handles the creation of a new user _ardupilot_ and ensures the SITL gets activated on launch of the container through a wrapper script. 

**NOTE: This template is configured to install the ArduPlane SITL, for other vehicle types (e.g. ArduCopter) the installation procedure is slightly different!**

```dockerfile

# Define build arguments for Ubuntu version and ArduPilot tag
ARG UBUNTU_VERSION=22.04
ARG ARDUPILOT_TAG=Plane-4.6.1 

# Set default coordinates for simulated vehicle
ENV LAT=38.9607456899675
ENV LON=-77.3121938109398
ENV ALT=94
ENV DIR=45

# Use the official Ubuntu image as the base image
FROM ubuntu:${UBUNTU_VERSION}

# Set noninteractive mode for APT to avoid prompts during install
ARG DEBIAN_FRONTEND=noninteractive

# Update system and install necessary packages
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y sudo git python3 python3-dev python3-pip python-is-python3 tzdata lsb-release cmake

# Install python packages
RUN pip3 install pexpect pymavlink

# Create a user for running ArduPilot
ARG USER_NAME=ardupilot
ARG USER_UID=1000
ARG USER_GID=1000
RUN groupadd ${USER_NAME} --gid ${USER_GID} && \
    useradd -l -m ${USER_NAME} -u ${USER_UID} -g ${USER_GID} -s /bin/bash

# Allow new user to execute sudo commands without a password
ENV USER=${USER_NAME}
RUN echo "${USER_NAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${USER_NAME}
RUN chmod 0440 /etc/sudoers.d/${USER_NAME}
RUN chown -R ${USER_NAME}:${USER_NAME} /home/${USER_NAME}

# Switch to ardupilot user
USER ${USER_NAME}
WORKDIR /home/${USER_NAME}

# Clone the ArduPilot repo and checkout the specified tag
ARG ARDUPILOT_TAG
RUN git clone https://github.com/ArduPilot/ardupilot.git
WORKDIR /home/${USER_NAME}/ardupilot
RUN git checkout ${ARDUPILOT_TAG}
RUN git submodule update --init --recursive

# Install additional dependencies using ArduPilot's setup script
RUN /home/${USER_NAME}/ardupilot/Tools/environment_install/install-prereqs-ubuntu.sh -y 

# Build ArduPilot SITL
RUN ./waf configure --board sitl
RUN ./waf plane

# Copy the wrapper script (launches SITL on boot)
COPY wrapper.sh /wrapper.sh

# Expose the TCP port 5760 (used for MAVLink and SITL communication)
EXPOSE 5760/tcp

# Run wrapper script when the container starts
CMD sh /wrapper.sh $LAT $LON $ALT $DIR

```

---

## 3. Create a Wrapper Script from Template

In the same folder as the Dockerfile, create a new file called **wrapper.sh** and paste in the template provided below. The last line of the Dockerfile instructs the container to execute this script at startup, so we use it to launch SITL. This is just one of several ways to achieve “launch SITL on boot” behavior; using a wrapper script also makes it easy to pass and manipulate vehicle location coordinates, as shown in the template.

**NOTE: This template is configured to launch the ArduPlane SITL, for other vehicle types (e.g. ArduCopter) the command is slightly different!**

```bash
#!/bin/bash

# Start ArduPilot SITL
HOME=${1},${2},${3},${4}
python3 /home/ardupilot/ardupilot/Tools/autotest/sim_vehicle.py -v ArduPlane --no-rebuild --no-mavproxy -l $HOME
```

---

## 4. Build the ArduPilot SITL Image

---

## 5. Deploy the ArduPilot SITL Image

---



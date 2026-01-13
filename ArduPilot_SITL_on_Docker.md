# Deploying ArduPilot in a Docker Environment

This guide explains the process of creating a Docker container that runs ArduPilot SITL and connecting to the simulator from Mission Planner and Dronekit running outside the container.

---

## Intuition

Running ArduPilot SITL inside Docker provides a clean, reproducible simulation environment that behaves the same across different machines. Instead of installing ArduPilot’s build tools and dependencies directly on your system, Docker encapsulates everything into a single container. This makes setups easy to share and allows you to reset or rebuild the environment at any time without affecting your host system.

This guide describes a workflow that starts from a minimal Linux container and uses Docker to execute the ArduPilot installation and build process a single time, resulting in an image with the SITL fully installed and configured. This image then becomes a reusable artifact to the team: rather than reinstalling dependencies and rebuilding ArduPilot on every machine, it can be distributed and deployed directly, allowing a ready-to-run SITL instance to be launched instantly.

---

## 1. Install Docker Desktop

Head over to the [Docker Desktop website](https://www.docker.com/products/docker-desktop/) and follow the instructions to install for either Windows, MacOS or Linux.

**After installation, briefly open Docker Desktop to ensure Docker is running on your computer!**

---

## 2. Create a Dockerfile from Template

--

## 3. Build the ArduPilot SITL Image

--

## 4. Deploy the ArduPilot SITL Image

--


# Wazuh SIEM Home Lab

A personal home lab running a Wazuh single-node deployment for practicing threat detection and endpoint monitoring. The Wazuh manager and dashboard run in Docker on a local machine, with an Ubuntu 22.04 Docker container enrolled as an agent.

## Table of Contents

- [Setup Guide](#setup-guide)
  - [1. Deploy a Wazuh Single-Node Docker Stack](#1-deploy-a-wazuh-single-node-docker-stack)
  - [2. Deploy the Ubuntu Agent (Docker)](#2-deploy-the-ubuntu-agent-docker)
- [Known Issues](#known-issues)
- [Links](#links)

## Setup Guide

### 1. Deploy Wazuh central components in a single-node stack

Follow the official guide:

[Wazuh Single-Node Docker Deployment](https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html#single-node-stack)

### 2. Deploy the Ubuntu Agent (via Docker)

#### a) Find the Wazuh Manager IP address

On the host machine, run:

```bash
docker inspect single-node-wazuh.manager-1 | grep IPAddress
```

_Note the returned IP address — it will be needed later._

#### b) Verify the Docker network name

On the host machine, run:

```bash
docker network ls
```

Look for the network with `single-node` in the name and note the full name.

#### c) Start an Ubuntu container on the Wazuh Docker network

On the host machine, run:

```bash
docker run -it --network single-node_default --name ubuntu-agent ubuntu:22.04 bash
```

> Replace `single-node_default` with the network name confirmed in step b, if different.

#### d) Install the Wazuh agent

Follow the "APT" tab from the official Wazuh Linux agent documentation:

[Wazuh Agent — Linux](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html)

> You should already be root inside the container — omit `sudo` from all commands.

#### e) Verify and fix the Manager IP in the agent config

The generated install command does not always write the Manager IP correctly. Inside the container, verify:

```bash
grep address /var/ossec/etc/ossec.conf
```

If the output shows `MANAGER_IP` or an empty `<address></address>`, replace it with the correct IP:

```bash
sed -i 's/MANAGER_IP/YOUR-MANAGER-IP/' /var/ossec/etc/ossec.conf
```

or if the field is empty:

```bash
sed -i 's/<address><\/address>/<address>YOUR-MANAGER-IP<\/address>/' /var/ossec/etc/ossec.conf
```

Verify the change:

```bash
grep address /var/ossec/etc/ossec.conf
```

#### f) Deploy the agent via the dashboard

- Run `uname -m` inside the container to confirm architecture — `aarch64` maps to **DEB aarch64**
- In the dashboard go to **Agents → Deploy new agent**
- Select the correct OS and architecture
- Enter the Manager IP from step a as the server address
- Assign an agent name
- Copy the generated install command and run it inside the container

Confirm the agent appears as **Active** in the Wazuh dashboard under the Agents tab.

If needed, start the agent from inside the container:

```bash
/var/ossec/bin/wazuh-control start
```

## Known Issues

### Agent disconnects after stack restart

When the Wazuh Docker stack is stopped and restarted, the Manager container may be assigned a new IP address. The agent config will still reference the old IP and the agent will show as disconnected in the dashboard.

To fix, verify the current Manager IP on the host machine:

```bash
docker inspect single-node-wazuh.manager-1 | grep IPAddress
```

Then update the agent config inside the container, replacing the old IP with the new one:

```bash
sed -i 's/OLD-IP/NEW-IP/' /var/ossec/etc/ossec.conf
```

Restart the agent inside the container:

```bash
/var/ossec/bin/wazuh-control restart
```

## Links

- [Wazuh Single-Node Docker Deployment](https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html#single-node-stack)
- [Wazuh Agent — Linux](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html)
- [Active Response](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html)
- [MITRE ATT&CK](https://documentation.wazuh.com/current/user-manual/ruleset/mitre.html)
  - [attack.mitre.org](https://attack.mitre.org/)

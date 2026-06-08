---

# Playit Setup in a TrueNAS Docker Container

This README explains how to install and run Playit inside a minimal Debian 13 (Trixie) container on TrueNAS, avoiding systemd errors and making the agent connect successfully.

---

## 1. Prerequisites

* TrueNAS with Docker support
* Debian 13 container (minimal)
* Root access inside the container
* Internet access for downloading Playit packages

---

## 2️. Install Dependencies

Inside your container, run:

```sh
apt update
apt install -y gnupg curl ca-certificates
```

> **Note:** `sudo` is not required inside the container because you are already root.

---

## 3️. Install Playit

Download and install the official Playit package:

```sh
# Download GPG key
curl -SsL https://packages.playit.gg/keys/playit.gpg -o /usr/share/keyrings/playit.gpg

# Download repository list
curl -fsSL -o /etc/apt/sources.list.d/playit.list https://packages.playit.gg/repo-files/playit-debian.list

# Update package index
apt update

# Install Playit
apt install -y playit
```

**Ignore systemd errors** during installation—they are normal in Docker.

---

## 4️. Run the Playit Daemon Manually

Do **not** use `playit start` or `systemctl`. Instead, run the daemon directly:

```sh
/usr/local/bin/playitd
```

* The daemon will run in the foreground
* It will initially say:

```
Waiting for frontend secret provisioning over IPC
```

---

## 5️. Claim and Provision the Agent

1. Generate a claim code:

```sh
playit claim generate
```

This will output a URL like:

```
https://playit.gg/claim/XXXXXXXX
```

2. Exchange the code to provision the agent:

```sh
playit claim exchange XXXXXXXX
```

3. Verify that the daemon now has a `.toml` configuration:

```sh
ls -l /root/.config/playit_gg/playit.toml
```

* If this file exists, your agent is correctly provisioned.

---

## 6️. Check Connection

To verify the agent is connected:

```sh
playit status
```

* The agent should now appear as **online** on the Playit web dashboard.

---

## 7️. Optional: Keep Daemon Running in Docker

Create a simple entrypoint script `/start-playit.sh`:

```sh
#!/bin/sh
/usr/local/bin/playitd
```

Then run your container:

```sh
docker run -d --name playit \
  -v /your/config:/root/.config/playit_gg \
  your-playit-image \
  /start-playit.sh
```

* This keeps the agent running continuously
* Logs can be viewed via `docker logs playit`

---

## Notes / Tips

* Never rely on `playit start` or `playit attach` in Docker—they assume systemd.
* Always run `/usr/local/bin/playitd` directly for containers.
* Claiming and exchanging the code is mandatory for the agent to appear online.
* Persist `/root/.config/playit_gg/` as a Docker volume to retain the agent across restarts.

---

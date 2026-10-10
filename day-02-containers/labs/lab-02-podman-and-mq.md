# Lab 02: Podman Basics and an IBM MQ Container

**Time:** 45 to 60 minutes | **Cost:** Free

> **Safety:** Passwords below are lab-only placeholders. Never commit real passwords, keys, or tokens to this repo. By setting `LICENSE=accept` you accept the IBM MQ developer license terms; read them first.

## Exercise 1: Install Podman (10 min)

**macOS (Homebrew):**
```bash
brew install podman
podman machine init
podman machine start
podman --version
```
**Windows:** install Podman Desktop and enable WSL2. **Linux:** `sudo dnf install podman` or `sudo apt install podman`.

(If you prefer Docker, replace `podman` with `docker` in all commands.)

Verify:
```bash
podman run --rm hello-world
```

## Exercise 2: Your first container (10 min)

```bash
podman pull docker.io/library/nginx:latest
podman images
podman run -d --name web -p 8080:80 nginx
podman ps
curl http://localhost:8080
podman logs web
```

Questions:
1. What does `-p 8080:80` mean?
2. What happens to the page if you run `podman stop web`?

## Exercise 3: Look inside (10 min)

```bash
podman exec -it web sh
ps aux          # how many processes can you see?
hostname
exit
podman inspect web | head -40
podman stats --no-stream web
```

Observe: inside the container you see only a few processes (PID namespace). On the host, the same process is just a normal process.

## Exercise 4: Data is lost without a volume (10 min)

```bash
podman run -d --name tmp1 nginx
podman exec tmp1 sh -c 'echo hello > /tmp/data.txt'
podman rm -f tmp1
podman run -d --name tmp2 nginx
podman exec tmp2 cat /tmp/data.txt     # file is gone
```

Now with a volume:
```bash
podman volume create keepme
podman run -d --name vol1 -v keepme:/data nginx
podman exec vol1 sh -c 'echo hello > /data/data.txt'
podman rm -f vol1
podman run -d --name vol2 -v keepme:/data nginx
podman exec vol2 cat /data/data.txt    # file survives
```

Clean up:
```bash
podman rm -f web tmp2 vol2
```

## Exercise 5: Run IBM MQ in a container (20 min)

```bash
podman volume create qm1data

podman run -d --name qm1 \
  -e LICENSE=accept \
  -e MQ_QMGR_NAME=QM1 \
  -e MQ_ADMIN_PASSWORD=ChangeMe-Lab-Only1 \
  -e MQ_APP_PASSWORD=ChangeMe-Lab-Only2 \
  -p 1414:1414 -p 9443:9443 \
  -v qm1data:/mnt/mqm \
  icr.io/ibm-messaging/mq:latest
```

Check it started (may take 30 to 60 seconds):
```bash
podman ps
podman logs qm1 | tail -20
```

Explore:
```bash
podman exec -it qm1 bash
dspmq                         # list queue managers and status
runmqsc QM1                   # MQSC prompt
  DISPLAY QLOCAL(DEV.*)
  DISPLAY CHANNEL(DEV.*)
  END
exit
```

Web console: open `https://localhost:9443/ibmmq/console` (accept the self-signed certificate warning) and log in as `admin` with the password you set.

> **Apple Silicon note:** depending on the image version, MQ may run under emulation or you may need an ARM-compatible tag. If the container exits immediately, check `podman logs qm1` and see the troubleshooting file.

Questions:
1. Which ports did we publish and why?
2. What is stored in the `qm1data` volume, and what happens if you delete the container but keep the volume?
3. How is this similar to what the MQ Operator will create on OpenShift?

Test persistence: `podman rm -f qm1`, rerun the same `podman run` command, and confirm the queue manager keeps its data.

## Wrap-up and clean up

```bash
podman rm -f qm1
podman volume rm qm1data keepme
podman system prune -f      # careful: removes unused data
```

Write three sentences: what is a container, why do we need volumes, what image are we using for MQ and where does it come from?



Introduction:
=========

This is a minimal, self-hosted bundle that brings an Overleaf-style LaTeX editing experience to your own machine. It’s designed to run on localhost via Docker Compose and use a TeX Live ISO for an offline installation, giving you a reproducible LaTeX toolchain without relying on external mirrors during setup.

With a single docker compose up -d, OpenLeaf starts a local instance and exposes a launchpad and project workspace on port 8080. The ISO-based install keeps your LaTeX environment consistent across machines and rebuilds, which is especially useful for air-gapped networks, classrooms, and teams that need predictable, repeatable builds.

Key goals
=========

Local & simple: One Compose file; no cloud account required.

Offline-friendly: Installs TeX Live from a mounted ISO, avoiding network variability and mirror drift.

Reproducible builds: Pin your LaTeX toolchain to the ISO you choose, so documents compile the same today and months from now.

Familiar workflow: A browser UI similar to Overleaf, accessible at http://localhost:8080.


What this repository provides
-----------------------------

Dockerfiles and a Compose setup to run the editor locally.

A helper script to fetch/mount the TeX Live ISO used during the container build.

Quick-start instructions to launch the service, create the first user, and open your project space.

If you’re looking for a lightweight way to draft, compile, and share LaTeX documents locally—while keeping your toolchain locked to a specific TeX Live release—OpenLeaf gives you a fast, reproducible starting point.



First Run:
=========

1. Clone this repository
```bash
git clone git@github.com:lucribas/openleaf.git
cd openleaf
```

2. Download ISO (6GB)

Please, change `docker/get_iso.bash` to use the mirror closest to you.

```bash
( cd docker && ./get_iso.bash )
```

3. Build containers

```bash
docker compose -f docker-compose.yaml build
```

4. Run open leaf

```bash
docker compose -f docker-compose.yaml  up -d
```


5. Create user (first time)

Go to: http://localhost:8080/launchpad


6. Open main page

Go to: http://localhost:8080/project


Next Runs:
=========

1. Run open leaf

```bash
docker compose -f docker-compose.yaml  up -d
```


2. Open main page

Go to: http://localhost:8080/project


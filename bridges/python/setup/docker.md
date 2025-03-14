# Bridge setup with Docker

{{ #include ../selector.html }}

<p class="bridge-filter" bridges="signal" bridge-no-generic>
  <strong>The Signal bridge requires a second docker container for signald.
  Instructions for setting up everything can be found on the
  <a href="../signal/setup-docker.md">Signal-specific Bridge setup with Docker page</a></strong>
</p>

## Requirements
* Docker
* A Matrix homeserver that supports application services (e.g. [Synapse](https://github.com/matrix-org/synapse))

## Setup
Docker images are hosted on dock.mau.dev. Available docker tags are generally
`:latest`, `:git tag`, `:git commit-amd64` and `:git commit-arm64`. The latest
and git tag specific docker tags are manifests that contain both amd64 and
arm64 images.

0. Create a directory for the bridge and cd into it: `mkdir mautrix-$bridge && cd mautrix-$bridge`.  
   **N.B.** The docker image will `chown` its `/data` directory to UID 1337.
   The commands below mount the working directory as `/data`, so make sure you
   always run them in the correct directory.
1. Pull the docker image with `docker pull dock.mau.dev/mautrix/$bridge:<version>`.
   Replace `<version>` with the version you want to run (e.g. `latest` or `v0.6.0`).
2. Run the container for the first time, so it can create a config file for you:
   ```
   docker run --rm -v `pwd`:/data:z dock.mau.dev/mautrix/$bridge:<version>
   ```
3. Update the config to your liking. You'll at least need to change the
   homeserver settings, appservice address and permissions.
4. Generate the appservice registration by running the container again, same
   command as above.
5. Add the path to the registration file to your Synapse's `homeserver.yaml`
   under `app_service_config_files`. Restart Synapse to apply changes.
6. Run the bridge:
   ```
   docker run --restart unless-stopped -v `pwd`:/data:z dock.mau.dev/mautrix/$bridge:<version>
   ```
   Additionally, you should either add the bridge to the same Docker network as
   Synapse with `--network=synapsenet`, or expose the correct port with
   `-p <port>:<port>`.

## Upgrading
1. Pull the new version (setup step 1)
2. Start the new version (setup step 7)

## Docker compose
0. Create a directory for the bridge like step #0 in the Docker CLI
   instructions above.
1. Create `docker-compose.yml` that contains something like this:
   ```yaml
   version: "3.7"

   services:
     mautrix-$bridge:
       container_name: mautrix-$bridge
       image: dock.mau.dev/mautrix/$bridge:<version>
       restart: unless-stopped
       volumes:
       - .:/data
   ```
2. Follow the rest of the Docker setup, but use compose commands instead of the
   raw `docker` commands: `docker-compose up -d` to start, `docker-compose stop`
   to stop and `docker-compose pull` to update.

If you want to set it up in an existing docker-compose file instead of a new
dedicated one, simply adjust the `volumes` section to mount a subdirectory
instead of the current directory as the data directory:

```yaml
volumes:
- ./mautrix-$bridge:/data
```

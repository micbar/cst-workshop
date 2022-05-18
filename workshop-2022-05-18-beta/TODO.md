# Discover oCIS 2.0.0-beta1 with docker

Hint: depending on your docker installation you may have to use `docker compose` instead of `docker-compose`.

## start the docker-compose stack

- `docker-compose up -d`
- look a the logs `docker-compose logs -f`
- change the log level and apply using `docker-compose up -d`
- look again at the logs

## ocis doesn't start

- ocis has been secured for the public beta phase
- ocis refuses to start without changing all secrets
- the ocis init command has been created [ocis init](https://owncloud.dev/ocis/getting-started/)
- change the docker-compose.yml that ocis starts and use your own admin password

## ports

- expose the port 9200 from the oCIS container to your local machine port 9200
- open <https://localhost:9200> in you browser
- play around with the oCIS, you can create more users, work with spaces and share resources

## volumes / persistence

- look at the running containers `docker-compose ps`
- take the name of the container and pass it to `docker inspect <name>`, look what information you can get from there, ask if you got questions

- filter the output to only see volumes the container uses: `docker inspect -f '{{ json .Mounts }}' <name>` (you also can pipe the output through jq to pretty print it)
  - have a look where the volume is mounted in the container
  - have a look where the volume is mounted on the docker host
  - you may also take the path of the volume and see whats in there (e.g. with `tree`)

- stop the container by running `docker-compose down`

- start the container by running `docker-compose up -d`

- look again at the volume mounts of the oCIS container
  - did it change?
  - is your stuff which you previously uploaded to oCIS still there?

- add persistence by adding a named volume to the service (<https://docs.docker.com/storage/volumes>)
  - therefore look where the anonymous volume was mounted previously

- check, that your data now survives a `docker-compose down` and `docker-compose up -d`

## resilience

- the docker-compose stack currently would not survive a reboot of the docker host or even a restart of the docker daemon. How can you ensure this? (you can test without rebooting by just restarting the docker daemon eg `systemctl restart docker`)

- read about logging <https://docs.docker.com/config/containers/logging/configure> / <https://docs.docker.com/compose/compose-file/compose-file-v3/#logging> and ensure log rotation for the oCIS container (in the docker-compose file, no docker daemon configuration)

## update images

- check for the availability of an updated docker image by running `docker-compose pull` and `docker-compose up -d`

- Questions:
  - Why should one not use the `latest` image tag for production deployments?
  - How can I ensure that I always get the exact same image?

## oCIS demo users

Demo users are not created anymore by default. They can only be created once during the first startup when running ocis with `IDM_CREATE_DEMO_USERS=true`.

## Secrets

Examine the content of the ocis config file in `/etc/ocis/ocis.yaml` via `docker-compose exec ocis cat /etc/ocis/ocis.yaml`.

## Switch on the new audit service in a separate container

- add the ocis audit service and run `docker-compose up -d`
- check that the audit service runs
- add a bind mount from a local folder on the host into the container with the audit service to store the audit.log
- configure the audit service to log to a file
- configure the audit service to start after the ocis nats service is healthy (tip: use the docker-compose healthcheck feature together with ocis subcommands)
- watch the audit.log file on the host with `tail -f ...` while doing some actions in ocis

## change domain name / ip address where you can reach oCIS

- try to access oCIS from a different interface / ip in the browser
  - therefore ensure that the docker container port is exposed on all your local interfaces (eg. `ss -lntp`)
  - pick one ip of your interfaces from `ip a`, eg `eth0` oder `wlan0`
  - you'll get a white page (next oC Web version will show an error) or will be redirected to your configured domain because you accessed the web UI from a different url than provided in `OCIS_URL`

- change the `OCIS_URL` to the ip / domain name you tried to access oCIS from
- Currently you will lose your data because if you change the IdP it is a different user for ocis. Needs to be fixed.
- Workaround `docker-compose down -v` (-v flag deletes all persistent volumes) and `docker-compose up -d`
- try to access oCIS from the different interface / ip in the browser, which should work now.
- If you like, Join the iOS Testflight and test the new spaces feature on iOS.

## oCIS version / services

- shell into the container
- run `ocis version` to see all running services / versions

## If you still got time

- Start this docker-compose <https://owncloud.dev/ocis/deployment/ocis_traefik/> / <https://github.com/owncloud/ocis/tree/master/deployments/examples/ocis_traefik>. If you do it on a Hetzner VM, get valid SSL certificates.

# Discover oCIS with docker

## start the docker-compose stack
- `docker-compose up -d`
- look a the logs `docker-compose logs -f`
- change the log level and apply using `docker-compose up -d`
- look again at the logs

## ports
- expose the port 9200 from the oCIS container to your local machine port 9200
- open https://localhost:9200 in you browser
- play around with the oCIS, you can use these demo users https://owncloud.dev/ocis/getting-started/demo-users/


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

- add persistence by adding a named volume to the service (https://docs.docker.com/storage/volumes)
  - therefore look where the anonymous volume was mounted previously

- check, that your data now survives a `docker-compose down` and `docker-compose up -d`

## resilience

- the docker-compose stack currently would not survive a reboot of the docker host or even a restart of the docker daemon. How can you ensure this? (you can test without rebooting by just restarting the docker daemon eg `systemctl restart docker`)

- read about logging https://docs.docker.com/config/containers/logging/configure / https://docs.docker.com/compose/compose-file/compose-file-v3/#logging and ensure log rotation for the oCIS container (in the docker-compose file, no docker daemon configuration)

## update images

- check for the availability of an updated docker image by running `docker-compose pull` and `docker-compose up -d`

- Questions:
  - Why should one not use the `latest` image tag for production deployments?
  - How can I ensure that I always get the exact same image?

## disable oCIS demo users / change default secrets

- disable the demo user generation https://owncloud.dev/ocis/deployment/#delete-demo-users
  - start from scratch by running `docker-compose down -v` (first read what it does)
  - ensure that oCIS doesn't generate the demo users anymore

- create your own user with admin privileges

- use the accounts cli to change the default passwords of the "IDP" and "Reva" system accounts
  - run `docker-compose exec ocis sh` and you'll have an interactive shell in the container
  - run `ocis accounts help` to see how you can manipulate accounts)
  - you also need to set the updated passwords in certain environment variables, for more details see https://owncloud.dev/ocis/deployment/#change-default-secrets

- when you read https://owncloud.dev/ocis/deployment/#change-default-secrets, you noticed that there are more default secrets, change them, too!

## change domain name / ip address where you can reach oCIS

- try to access oCIS from a different interface / ip in the browser
  - therefore ensure that the docker container port is exposed on all your local interfaces (eg. `ss -lntp`)
  - pick one ip of your interfaces from `ip a`, eg `eth0` oder `wlan0`
  - you'll get a white page (next oC Web version will show an error) or will be redirected to your configured domain because you accessed the web UI from a different url than provided in `OCIS_URL`

- change the `OCIS_URL` to the ip / domain name you tried to access oCIS from
  - after doing so shell into the container and look at the file `/var/lib/ocis/idp/identifier-registration.yaml`, what does it do?
  - Delete the file, because it be recreated if it doesn't exist and therefore will pick up your domain / ip change from `OCIS_URL` automatically. Otherwise you could manually change your domain /ip in that file
  - restart oCIS by running `docker-compose restart`
  - If you didn't change / delete the file you will see an error `Unexpected HTTP response: 400. Please check your connection and try again.` on the login screen.

- try to access oCIS from the different interface / ip in the browser, which should work now.

## oCIS version / services

- shell into the container
- run `ocis version` to see all running services / versions

## If you still got time

- Start this docker-compose https://owncloud.dev/ocis/deployment/ocis_traefik/ / https://github.com/owncloud/ocis/tree/master/deployments/examples/ocis_traefik. If you do it on a Hetzner VM, get valid SSL certificates.

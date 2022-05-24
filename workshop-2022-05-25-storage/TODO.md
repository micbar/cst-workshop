Work on these topics as a group of ~3 people:

- ssh onto your group's server
  - ssh commands:
    - group 1: `ssh admin@ocis.group1.storage-workshop.owncloud.works`
    - group x: #TODO

- run oCIS 2.0.0-beta2 with oCIS file system on local disk, see https://owncloud.dev/ocis/getting-started/#binaries
- set OCIS_URL=`https://ocis.group<<X>>.storage-workshop.owncloud.works`
- `export IDM_ADMIN_PASSWORD=admin` if you want to set the initial admin password to `admin`
- `export PROXY_HTTP_ADDR=0.0.0.0:443` to make oCIS listen on the default https port
- `sudo setcap CAP_NET_BIND_SERVICE=+eip ./ocis` to allow oCIS to bind to ports below 1024 without sudo
- `sudo ufw allow https` open the firewall for https
- `./ocis init` bootstrap an oCIS configuration
- create custom users, upload data, create spaces via the Web UI
- install `tree` with `sudo apt install tree`
- and have a look at the oCIS data directory `tree ~/.ocis`

- install nfs dependencies `sudo apt install nfs-common`
- create nfs mount directory `mkdir ~/nfs-mount`
- mount nfs `sudo mount -v -t nfs 127.0.0.1:/ ~/nfs-mount`
- `sudo chown -R admin:admin ~/nfs-mount`

- stop oCIS
- move oCIS' data + metadata to NFS share
- `rsync -avzX ~/.ocis ~/nfs-mount` https://access.redhat.com/solutions/3628891
- `rm -rf ~/.ocis/`
- configure oCIS to use NFS location `export OCIS_BASE_DATA_PATH=/home/admin/nfs-mount/.ocis/` and `export OCIS_CONFIG_DIR=/home/admin/nfs-mount/.ocis/config/`
- start oCIS
- ensure no data is lost (don't trust cached previews)

- stop oCIS
- get minio client https://docs.min.io/docs/minio-client-quickstart-guide.html
- s3 info:
  - address: 127.0.0.1:9000
  - access key: ocis
  - secret key: ocis-secret-key
- configure an alias `./mc alias set ocis http://127.0.0.1:9000 ocis ocis-secret-key`
- list buckets `./mc ls ocis`
- move oCIS' data (blobs) into the bucket
  - NOTE: when the spaceid has the format xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx,
    - oCIS FS has following blob layout: <root>/spaces/xx/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/blobs/...
    - S3NG has following blob layout <root>/xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/..
  - example copy command `./mc cp --recursive ~/nfs-mount/.ocis/storage/users/spaces/ad/85e54c-3038-47af-a3d0-81ac5c8c246e/blobs/ ocis/ocis-bucket/ad85e54c-3038-47af-a3d0-81ac5c8c246e/`
  - -> do this for every space
- configure oCIS to use s3ng driver
  - `export STORAGE_USERS_DRIVER=s3ng`
  - `export STORAGE_USERS_S3NG_ENDPOINT=http://127.0.0.1:9000`
  - `export STORAGE_USERS_S3NG_REGION=default`
  - `export STORAGE_USERS_S3NG_ACCESS_KEY=ocis`
  - `export STORAGE_USERS_S3NG_SECRET_KEY=ocis-secret-key`
  - `export STORAGE_USERS_S3NG_BUCKET=ocis-bucket`
- start oCIS
- ensure no data is lost (don't trust cached previews)

Work on these topics as a group of ~3 people:

- ssh onto your group's server
  - ssh commands:
    - group 1: `ssh admin@xxx`

- run oCIS 2.0.0-beta2 with oCIS file system on local disk, see https://owncloud.dev/ocis/getting-started/#binaries
- set OCIS_URL=`https://xxx`
- `export IDM_ADMIN_PASSWORD=admin` if you want to set the initial admin password to `admin`
- `export PROXY_HTTP_ADDR=0.0.0.0:443`
- `sudo setcap CAP_NET_BIND_SERVICE=+eip ./ocis`
- `sudo ufw allow https`
- `./ocis init`
- create custom users, upload data, create spaces

- start nfs server
- `sudo apt install nfs-common`
- mount nfs `sudo mount -v -t nfs 127.0.0.1:/ ./nfs-mount`
- `sudo chown -R admin:admin nfs-mount`

- stop oCIS
- move oCIS' data + metadata to NFS share
- `rsync -avzX /home/admin/.ocis /home/admin/nfs-mount` https://access.redhat.com/solutions/3628891
- `rm -rf ~/.ocis/`
- configure oCIS to use NFS location `export OCIS_BASE_DATA_PATH=/home/admin/nfs-mount/.ocis/` and `export OCIS_CONFIG_DIR=/home/admin/nfs-mount/.ocis/config/`
- start oCIS
- ensure no data is lost

- stop oCIS
- get minio client https://docs.min.io/docs/minio-client-quickstart-guide.html
- s3 info:
  - address: 127.0.0.1:9000
  - access key: ocis
  - secret key: ocis-secret-key

- move oCIS' data (blobs) to S3 bucket
  - `./mc alias set ocis http://127.0.0.1:9000 ocis ocis-secret-key`
  - `./mc ls ocis`
  - NOTE: when the spaceid has the format xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx,
    - oCIS FS has following blob layout: <root>/spaces/xx/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/blobs/...
    - S3NG has following blob layout <root>/xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/..
  - `./mc cp --recursive ~/nfs-mount/.ocis/storage/users/spaces/ad/85e54c-3038-47af-a3d0-81ac5c8c246e/blobs/ ocis/ocis-bucket/ad85e54c-3038-47af-a3d0-81ac5c8c246e/` -> do this for every space
- configure oCIS to use s3ng driver
  - `export STORAGE_USERS_DRIVER=s3ng`
  - `export STORAGE_USERS_S3NG_ENDPOINT=http://127.0.0.1:9000`
  - `export STORAGE_USERS_S3NG_REGION=default`
  - `export STORAGE_USERS_S3NG_ACCESS_KEY=ocis`
  - `export STORAGE_USERS_S3NG_SECRET_KEY=ocis-secret-key`
  - `export STORAGE_USERS_S3NG_BUCKET=ocis-bucket`
- start oCIS
- ensure no data is lost (don't trust cached previews)

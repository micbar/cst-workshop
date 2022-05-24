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
- mount nfs `sudo mount -v -t nfs 127.0.0.1:/ ./nfs-mount`
- `sudo chown -R 1000:1000 nfs-mount`

- stop oCIS
- move oCIS' data + metadata to NFS share
- configure oCIS to use NFS location
- start oCIS
- ensure no data is lost

- stop oCIS
- move oCIS' data (blobs) to S3 bucket
  - TODO: minio cli
  - NOTE: when the spaceid has the format xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx,
    - oCIS FS has following blob layout: <root>/spaces/xx/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/blobs/...
    - S3NG has following blob layout <root>/xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/blobs/..
- configure oCIS to use s3ng driver
- start oCIS
- ensure no data is lost

# Garage

[Garage](https://garagehq.deuxfleurs.fr/) is open source S3 API compatible object storage.

This is a Docker Compose project that enables a single node Garage server for development and testing, but **not for production**.

## Generate a node configuration file

Use the `generate_config.sh` script to generate a Garage configuration file.

```shell
bash ./scripts/generate_config.sh
```

## Deploy a single node

Use the Docker Compose file to deploy a single node Garage instance.

```shell
docker compose up -d
```

Check the container status to ensure it is up and running.

```shell
docker ps -f name=garage-node --format "table {{.Names}}\t{{.Status}}"
```

1. Check the Garage server stats to get the server's node ID.

    ```shell
    docker compose exec garage-node /garage status
    ```

    Example output:

    ```plaintext
    2026-03-16T15:06:52.396866Z  INFO garage_net::netapp: Connected to 127.0.0.1:3901, negotiating handshake...
    2026-03-16T15:06:52.441854Z  INFO garage_net::netapp: Connection established to 9d823fdb137717b5
    ==== HEALTHY NODES ====
    ID                Hostname      Address         Tags  Zone  Capacity          DataAvail  Version
    9d823fdb137717b5  ddf882ba1ea9  127.0.0.1:3901              NO ROLE ASSIGNED             v2.2.0
    ```

## Configure the node storage

1. Assign 1 GB of storage capacity (replace <NODE_ID> with your actual ID).

    ```shell
    docker compose exec garage-node \
        /garage layout assign -z default -c 1G <NODE_ID>
    ```

    Example output:

    ```plaintext
    2026-03-16T15:08:56.691354Z  INFO garage_net::netapp: Connected to 127.0.0.1:3901, negotiating handshake...
    2026-03-16T15:08:56.732928Z  INFO garage_net::netapp: Connection established to 9d823fdb137717b5
    Role changes are staged but not yet committed.
    Use `garage layout show` to view staged role changes,
    and `garage layout apply` to enact staged changes.
    ```

1. Apply the storage capacity changes.

    ```shell
    docker compose exec garage-node /garage layout apply --version 1
    ```

    Example output:

    ```plaintext
    2026-03-16T15:10:29.314612Z  INFO garage_net::netapp: Connected to 127.0.0.1:3901, negotiating handshake...
    2026-03-16T15:10:29.356619Z  INFO garage_net::netapp: Connection established to 9d823fdb137717b5
    ==== COMPUTATION OF A NEW PARTITION ASSIGNATION ====
    
    Partitions are replicated 1 times on at least 1 distinct zones.
    
    Optimal partition size:                     3.9 MB
    Usable capacity / total cluster capacity:   1000.0 MB / 1000.0 MB (100.0 %)
    Effective capacity (replication factor 1):  1000.0 MB
    
    default             Tags  Partitions        Capacity   Usable capacity
      9d823fdb137717b5  []    256 (256 new)     1000.0 MB  1000.0 MB (100.0%)
      TOTAL                   256 (256 unique)  1000.0 MB  1000.0 MB (100.0%)
    
    
    New cluster layout with updated role assignment has been applied in cluster.
    Data will now be moved around between nodes accordingly.
    ```
 
## Create a key and bucket

Now that you configured storage, you can create a key and bucket.

1. Create an example key, and record the **Key ID** and **Secret key** values for later use.

    ```shell
    docker compose exec garage-node /garage key create my-key
    ```

    Example output:

    ```plaintext
    2026-03-16T15:12:14.764689Z  INFO garage_net::netapp: Connected to 127.0.0.1:3901, negotiating handshake...
    2026-03-16T15:12:14.808644Z  INFO garage_net::netapp: Connection established to 9d823fdb137717b5
    ==== ACCESS KEY INFORMATION ====
    Key ID:              GKc6c10a028c8b11795be08b09
    Key name:            my-key
    Secret key:          1d3279ba5b32dc9180c272175a06a67ffafb9c8c082d1fbf82bd369de2672c13
    Created:             2026-03-16 15:12:14.808 +00:00
    Validity:            valid
    Expiration:          never
    
    Can create buckets:  false
    
    ==== BUCKETS FOR THIS KEY ====
    Permissions  ID  Global aliases  Local aliases
    ```

1. Create an example bucket.

    ```shell
    docker compose exec garage-node /garage bucket create my-bucket
    ```

    Example output:

    ```plaintext
    2026-03-16T15:13:24.819330Z  INFO garage_net::netapp: Connected to 127.0.0.1:3901, negotiating handshake...
    2026-03-16T15:13:24.861944Z  INFO garage_net::netapp: Connection established to 9d823fdb137717b5
    ==== BUCKET INFORMATION ====
    Bucket:          8f3a5bf748fccbfeee75af6ec5197dd918313ed197d90ea11dcdda275f4c8967
    Created:         2026-03-16 15:13:24.862 +00:00
    
    Size:            0 B (0 B)
    Objects:         0
    
    Website access:  false
    
    Global alias:    my-bucket
    
    ==== KEYS FOR THIS BUCKET ====
    Permissions  Access key    Local aliases
    ```

1. Allow read and write on the example bucket for the example key.

    ```shell
    docker compose exec garage-node \
        /garage bucket allow --read --write my-bucket --key my-key
    ```

    Example output:

    ```plaintext
    2026-03-16T15:14:42.087073Z  INFO garage_net::netapp: Connected to 127.0.0.1:3901, negotiating handshake...
    2026-03-16T15:14:42.129089Z  INFO garage_net::netapp: Connection established to 9d823fdb137717b5
    ==== BUCKET INFORMATION ====
    Bucket:          8f3a5bf748fccbfeee75af6ec5197dd918313ed197d90ea11dcdda275f4c8967
    Created:         2026-03-16 15:13:24.862 +00:00
    
    Size:            0 B (0 B)
    Objects:         0
    
    Website access:  false
    
    Global alias:    my-bucket
    
    ==== KEYS FOR THIS BUCKET ====
    Permissions  Access key                          Local aliases
    RW           GKc6c10a028c8b11795be08b09  my-key
    ```

## Use it

Now you have a key, `my-key`, and bucket, `my-bucket` that you can write a file to, so give that a try with the `awscli` CLI tool in a Python virtual environment.

I use `uv` for the environment, but you can use whatever you like to install awscli.

1. Create a virtual environment.

    ```shell
    uv venv
    ```

1. Activate the virtual environment.

    ```shell
    source .venv/bin/activate
    ```

1. Use uv pip to install awscli.

    ```shell
    uv pip install awscli
    ```

1. Export environment variables to communicate with Garage.

    ```shell
    export AWS_ACCESS_KEY_ID=xxxx      # put your Key ID here  \
        AWS_SECRET_ACCESS_KEY=xxxx  # put your Secret key here \
        AWS_DEFAULT_REGION='garage' \
        AWS_ENDPOINT_URL='http://localhost:3900'
    ```

1. List the `my-bucket` bucket.

    ```shell
    aws s3 ls
    2026-03-16 11:22:08 my-bucket
    ```

## Cleanup

1. Stop Docker infrastructure.

    ```shell
    docker compose down
    ```

1. Remove data, metadata, and configuration.

```shell
rm -rf ./data/* ./meta/* ./garage.toml
```

## References

- [Garage Docker image](https://hub.docker.com/r/dxflrs/garage)
- [Quick start](https://garagehq.deuxfleurs.fr/documentation/quick-start/)
- [Documentation](https://garagehq.deuxfleurs.fr/documentation/quick-start/)
- [Cookbook](https://garagehq.deuxfleurs.fr/documentation/cookbook/)

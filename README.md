# Mempool Guru (https://mempool.guru)


## The architecture

Mempool Guru is a system that collects mempool data and displays real-time analysis.

The data collection system consists of a centralized DB and a number of mempool monitor nodes (MMN).

![mempool-logger drawio](https://user-images.githubusercontent.com/2434648/186568431-34a0eae7-9253-4c75-bae4-1ecd237077b0.png)

### DB

The centralized DB runs on a dedicated server in Virginia, USA. The DB can be accessed by MMNs provisioned with proper credentials.

### MMNs (Mempool monitor nodes)

Currently we have 6 MMNs deployed cross the globe. Each MMN runs a geth client along side a python script that fetches the mempool content from geth and stores it in DB.

### Dashboard

We built a dashboard at http://mempool.guru.

## Deploy an MMN

### Set up a full node

First, you need a full node set up using [eth-docker](https://eth-docker.net/).

```
git clone https://github.com/eth-educators/eth-docker
cd eth-docker

# patch geth configuration to our needs
curl https://gist.githubusercontent.com/bl4ck5un/ce5205f50a93e122fe1fc6c4517064a4/raw/70cdb31b312c69ac480ab55e73cc32599755053d/geth.diff | git apply -v

# configure the full node: **please choose geth as the execution client**.
./ethd config

# start the node!
./ethd up
```

It will take some time to sync. You can check progress with `docker logs -f eth-docker-execution-1`.


### Start MMN services

> 🚨 Important: read this carefully.

Then, you just need to spin up the docker service defined in [docker-compose.yml](docker-compose.yml) following these steps:

1. Clone this repo & go to the root directory
2. Use `cp .env-template .env` to create an environment file named `.env`. This file will be used by docker to populate environment variables in `docker-compose.yml`.
3. Configure your node by editing `.env`:
    - 🚨 change `MONITOR_ID` to a **unique ID for your node**, preferably something readable.
    - 🚨 Replace `POSTGRES_DB_URL` with a DB credential provided by the admin. It should look like `POSTGRES_DB_URL=postgresql://user:passwd@IP:port/db_name`. **Please keep the credential secret.**
    - Other environment variables **must** remain as is.
6. Start the service by `docker-compose up -d`.

This will start three docker containers

- `tlistener`: record pending transaction arrival time
- `blistener`: record block arrival time
- `txdump`: record full transaction details

You can check logs of these containers using, e.g., `docker logs -f tlistener`. Similarly, you can check the logs of `txdump` and `blistener`.


### Some useful commands

- Attach to geth console: `docker exec -it eth-docker-execution-1 geth attach`


## Upgrade to a newer version

```
git pull
docker compose up -d
```

## TODO

Many.

- improving monitor
- add automated scripots
- add more stuff to the dashboard

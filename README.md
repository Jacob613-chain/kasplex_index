# Introduction to indexer components

## Description

1. Node-Syncer - pulls the original data (Block/Transaction/VSPC) of the node in real time and writes it to the archive database.
2. OP-Executor - parses and executes the OP data in the transaction and updates the state data.
3. OP-Stats (optional) - performs additional statistics and indexing on the state data.
4. API-Server - provides a data query interface.

## Steps

1. Deploy Node-Syncer

- Download the binary file and unzip it.

```bash
sudo mkdir /var/kasplex
```

```bash
sudo mkdir /var/kasplex/syncer
```

```bash
sudo wget https://download.kasplex.org/indexer/kpsyncer/kpsyncer-linux-gnu-amd64-latest.zip -O ./kpsyncer.zip
```

```bash
sudo unzip -o -d /var/kasplex/syncer ./kpsyncer.zip
```

```bash
sudo chmod 0775 /var/kasplex/syncer/kpsyncer
```

- Configuration, config.json file in the directory (rename after editing config.sample.json).

```bash
node: This part is the node parameters.
grpc: array, the addresses and ports of multiple node grpc, such as 127.0.0.1:16110.
start: the hash of the sync start block, only when testnet=true.
daaScoreStart: the DaaScore of the sync start block, only when testnet=true.
archive: the archive api address for the first initialization sync. The default is https://api.kasplex.org.
cassandra: This part is the cluster db connection parameters.
host: connection address.
port: connection port, the default is 9142.
user: database user name.
pass: database password.
crt: connection certificate.
space: database name.
testnet: please set it to true when connecting to the TN10 node.
debug: debugging info level, optional range 0-3.
```

```bash
sudo mv config.sample.json config.json
```

For example:
```bash
{
    "node": {
        "grpc": [
            "127.0.0.1:16110",
            "127.0.0.1:16110",
            "127.0.0.1:16110"
        ],
        "start": "",
        "daaScoreStart": 0,
        "archive": "https://api.kasplex.org"
    },
    "cassandra": {
        "host": "127.0.0.1",
        "port": 9042,
        "user": "cassandra",
        "pass": "cassandra",
        "crt": "",
        "space": "kpdb01mainnet"
    },
    "testnet": false,
    "debug": 2
}
```

- Run manually.

```bash
sudo /var/kasplex/syncer/kpsyncer
```

- Configure system services. Create /etc/systemd/system/kasplex-syncer.service file.

```bash
[Unit]
	Description=Kasplex Syncer Service
	After=network.target
[Service]
	WorkingDirectory=/var/kasplex/syncer
	ExecStart=/var/kasplex/syncer/kpsyncer
	Restart=always
	RestartSec=10s
	User=root
	Group=root
[Install]
	WantedBy=multi-user.target
```

```bash
sudo nano /etc/systemd/system/kasplex-syncer.service
```

- Update service configuration.

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl enable kasplex-syncer.service
```

- Start service.

```bash
sudo systemctl start kasplex-syncer.service
```

- The first run will enter the initial sync process to pull historical data from the archive db provided by the Kasplex public API service. This process will take more time (hours or days).


2. Deploy OP-Executor

- Download the binary file and unzip it.

```bash
sudo mkdir /var/kasplex
```

```bash
sudo mkdir /var/kasplex/executor
```

```bash
sudo wget https://download.kasplex.org/indexer/kpexecutor/kpexecutor-linux-gnu-amd64-latest.zip -O ./kpexecutor.zip
```

```bash
sudo unzip -o -d /var/kasplex/executor  ./kpexecutor.zip
```

```bash
sudo chmod 0775 /var/kasplex/executor/kpexecutor
```

- Install rocksdb-6.15 and related dependencies, or download the binary file directly.

```bash
sudo apt install -y libsnappy-dev libgflags-dev
```

```bash
sudo wget https://download.kasplex.org/indexer/kpexecutor/librocksdb.so.6.15.zip -O ./librocksdb.so.6.15.zip
```

```bash
sudo unzip -o -d /var/kasplex/executor ./librocksdb.so.6.15.zip
```

```bash
sudo cp /var/kasplex/executor/librocksdb.so.6.15 /usr/lib
```

- Configuration, config.json file in the directory (rename after editing config.sample.mainnet.json).

```bash
startup: This part is the startup parameters.
hysteresis: the DaaScore of execution hysteresis, it is recommended to set it to 3.
start: the hash of the sync start block, only when testnet=true.
daaScoreRange: array, the allowed DaaScore range, only when testnet=true.
tickReserved: array, the reserved tick list, only when testnet=true.
cassandra: This part is the cluster db connection parameters.
host: connection address.
port: connection port, the default is 9142.
user: database user name.
pass: database password.
crt: connection certificate.
space: database name.
rocksdb: This part is the local database parameters.
path: database path.
testnet: please set it to true when connecting to the TN10 node.
debug: debugging info level, optional range 0-3.
```

```bash
sudo mv config.sample.mainnet.json config.json
```

- Run manually.

```bash
sudo /var/kasplex/executor/kpexecutor
```

- Configure system services. Create /etc/systemd/system/kasplex-executor.service file.

```bash
[Unit]
	Description=Kasplex Executor Service
	After=network.target
[Service]
	WorkingDirectory=/var/kasplex/executor
	ExecStart=/var/kasplex/executor/kpexecutor
	Restart=always
	RestartSec=10s
	User=root
	Group=root
[Install]
	WantedBy=multi-user.target
```

```bash
sudo nano /etc/systemd/system/kasplex-executor.service
```

- Update service configuration.

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl enable kasplex-executor.service
```

- Start service.

```bash
sudo systemctl start kasplex-executor.service
```

- The first run will process from a preset starting block, sourced from the archived data generated by the Node-Syncer.

3. Deploy API-Server

- Download the binary file and unzip it.

```bash
sudo mkdir /var/kasplex
```

```bash
sudo mkdir /var/kasplex/api
```

```bash
sudo wget https://download.kasplex.org/indexer/kpapi/v1kpapi-linux-gnu-amd64-latest.zip -O ./v1kpapi.zip
```

```bash
sudo unzip -o -d /var/kasplex/api ./v1kpapi.zip
```

```bash
sudo chmod 0775 /var/kasplex/api/v1kpapi
```

- Configuration, config.json file in the directory (rename after editing config.sample.json).

```bash
cassandra: This part is the cluster db connection parameters.
host: connection address.
port: connection port, the default is 9142.
user: database user name.
pass: database password.
crt: connection certificate.
space: database name.
api: This part is the server parameters.
host: listening address.
port: listening port.
testnet: please set it to true when connecting to the TN10 node.
debug: debugging info level, optional range 0-3.
```

```bash
sudo mv config.sample.json config.json
```

- Run manually.

```bash
sudo /var/kasplex/api/v1kpapi
```

- Configure system services. Create /etc/systemd/system/kasplex-api.service file.

```bash
[Unit]
	Description=Kasplex Api Service
	After=network.target
[Service]
	WorkingDirectory=/var/kasplex/api
	ExecStart=/var/kasplex/api/v1kpapi
	Restart=always
	RestartSec=10s
	User=root
	Group=root
[Install]
	WantedBy=multi-user.target
```

```bash
sudo nano /etc/systemd/system/kasplex-api.service
```

- Update service configuration.

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl enable kasplex-api.service
```

- Start service.

```bash
sudo systemctl start kasplex-api.service
```

4. Select table.

```bash
DESCRIBE KEYSPACES;
USE kpdb01mainnet;
DESCRIBE TABLES;
DESCRIBE TABLE table1;
SELECT * FROM stbalance WHERE column_name = 'value';
```

- Get all holder's address and balance according to special tick.

```bash
SELECT * FROM stbalance WHERE tick  = 'NACHO'  ALLOW FILTERING;
```

## Notes

- Status monitoring (manual).

You can use cqlsh or the query tool provided by the AWS console.

// value3: The current DaaScore completed.
// value1: Synchronization status ('1' means synchronized).

```bash
SELECT * FROM yourdbname.runtime WHERE key='H_CBLOCK_LAST';
SELECT * FROM yourdbname.runtime WHERE key='ST_SYNCED';
```

- Use the latest daascore for accurate results.

- If you need to reset all data (manually), please stop the service first, then delete the relevant data tables and records (CQL is as follows), and then restart it.

```bash
DELETE FROM yourdbname.runtime WHERE key='H_ARCHIVE_LAST';
DELETE FROM yourdbname.runtime WHERE key='H_BLOCK_LAST';
DELETE FROM yourdbname.runtime WHERE key='H_CBLOCK_LAST';
DELETE FROM yourdbname.runtime WHERE key='ST_SYNCED';
DROP TABLE yourdbname.block;
DROP TABLE yourdbname.transaction;
DROP TABLE yourdbname.vspc;
```
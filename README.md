# hive - Ethereum end-to-end test harness

Hive is a system for running integration tests against Ethereum clients.

Ethereum Foundation maintains a public Hive instance to check for consensus, p2p and
blockchain compatibility. You can find the latest test results [here][hive-prod].

The hive project is licensed under the [GNU General Public License v3.0][gpl]. You can

## Setup

```bash
sudo setfacl --modify user:$USER:rw /var/run/docker.sock
apt install golang-go
go build
./hive --sim devp2p --client go-ethereum-bissias_latest --results-root /tmp/hive_results --loglevel 4
```

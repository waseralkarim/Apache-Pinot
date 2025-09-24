## Apache Pinot Default Ports

| Component | Port | Purpose |
| --- | --- | --- |
| **Controller** | **9000** | REST API & Web UI (cluster management, schema/table configs, ingestion, queries) |
| **Broker** | **8099** | Query endpoint (Superset, CLI, APIs connect here) |
| **Server** | **8098** | Data serving (internal between Broker ↔ Server for query execution) |
| **Minion** | **9514** | Minion tasks (optional, background jobs like compaction) |
| **Zookeeper** | **2181** | Cluster coordination & metadata (Pinot requires this, unless using Helix alternatives) |
- Pinot **doesn’t expose a traditional JDBC/ODBC protocol** like MySQL or Postgres.
- Instead, Pinot has its own **SQL-like query engine** and provides **REST APIs** for queries.

# Pre-Requisite

## **Step 1: Installing Java in Linux**

If you do not have **Java** installed on your system, you can download and install it from the [**official Oracle website**](https://www.oracle.com/java/technologies/downloads/).

For [**most Linux distributions**](https://www.tecmint.com/top-most-popular-linux-distributions/), you can use the package manager to install **Java**. For example, on [**Debian-based systems**](https://www.tecmint.com/debian-based-linux-distributions/), you can use the following command.

```
sudo apt-get install default-jdk
```

After the installation is complete, you can verify the **Java** version by running the following command.

```
java -version

```

## **Step 2: Installing Zookeeper in Linux**

**Zookeeper** is required by **Apache Pinot** for cluster management, so install it using the following command.

```
sudo apt install zookeeperd         [OnDebian-based Systems]
```

Once installed, start, enable, and verify the status of the **Zookeeper** service.

```
sudo systemctl start zookeeper
sudo systemctl enable zookeeper
sudo systemctl status zookeeper
```

# **Install and Run Pinot**

### Step 1: Download and navigate the files

```bash
# Download Pinot
wget https://archive.apache.org/dist/pinot/apache-pinot-1.1.0/apache-pinot-1.1.0-bin.tar.gz

tar -xvzf apache-pinot-1.1.0-bin.tar.gz
cd apache-pinot-1.1.0-bin
```
### Step 2: Create the systemd files for Pinot Components

Before running the systemd commands the configuration files can be found inside apache-pinot-1.1.0-bin folder

### Controller

```bash
sudo tee /etc/systemd/system/pinot-controller.service > /dev/null <<EOF
[Unit]
Description=Apache Pinot Controller
After=network.target

[Service]
User=devops
WorkingDirectory=/home/devops/apache-pinot-1.1.0-bin
ExecStart=/home/devops/apache-pinot-1.1.0-bin/bin/pinot-admin.sh StartController -config /home/devops/apache-pinot-1.1.0-bin/conf/pinot-controller.conf
Restart=always
RestartSec=5
StandardOutput=append:/home/devops/apache-pinot-1.1.0-bin/logs/systemd/controller.log
StandardError=append:/home/devops/apache-pinot-1.1.0-bin/logs/systemd/controller-error.log

[Install]
WantedBy=multi-user.target
EOF
```

### Broker

```bash
sudo tee /etc/systemd/system/pinot-broker.service > /dev/null <<EOF
[Unit]
Description=Apache Pinot Broker
After=pinot-controller.service

[Service]
User=devops
WorkingDirectory=/home/devops/apache-pinot-1.1.0-bin
ExecStart=/home/devops/apache-pinot-1.1.0-bin/bin/pinot-admin.sh StartBroker -config /home/devops/apache-pinot-1.1.0-bin/conf/pinot-broker.conf
Restart=always
RestartSec=5
StandardOutput=append:/home/devops/apache-pinot-1.1.0-bin/logs/systemd/broker.log
StandardError=append:/home/devops/apache-pinot-1.1.0-bin/logs/systemd/broker-error.log

[Install]
WantedBy=multi-user.target
EOF
```

### Server

```bash
sudo tee /etc/systemd/system/pinot-server.service > /dev/null <<EOF
[Unit]
Description=Apache Pinot Server
After=pinot-controller.service

[Service]
User=devops
WorkingDirectory=/home/devops/apache-pinot-1.1.0-bin
ExecStart=/home/devops/apache-pinot-1.1.0-bin/bin/pinot-admin.sh StartServer -config /home/devops/apache-pinot-1.1.0-bin/conf/pinot-server.conf
Restart=always
RestartSec=5
StandardOutput=append:/home/devops/apache-pinot-1.1.0-bin/logs/systemd/server.log
StandardError=append:/home/devops/apache-pinot-1.1.0-bin/logs/systemd/server-error.log

[Install]
WantedBy=multi-user.target
EOF
```

### Minion

```bash
sudo tee /etc/systemd/system/pinot-minion.service > /dev/null <<EOF
[Unit]
Description=Apache Pinot Minion
After=pinot-controller.service

[Service]
User=devops
WorkingDirectory=/home/devops/apache-pinot-1.1.0-bin
ExecStart=/home/devops/apache-pinot-1.1.0-bin/bin/pinot-admin.sh StartMinion -config /home/devops/apache-pinot-1.1.0-bin/conf/pinot-minion.conf
Restart=always
RestartSec=5
StandardOutput=append:/home/devops/apache-pinot-1.1.0-bin/logs/systemd/minion.log
StandardError=append:/home/devops/apache-pinot-1.1.0-bin/logs/systemd/minion-error.log

[Install]
WantedBy=multi-user.target
EOF
```

---

### Reload & Enable all

**Before starting:**

```bash
mkdir -p /home/devops/apache-pinot-1.1.0-bin/logs/systemd
chown -R devops:devops /home/devops/apache-pinot-1.1.0-bin

```

```bash
sudo systemctl daemon-reload
sudo systemctl enable pinot-controller pinot-broker pinot-server pinot-minion
```

Then start them:

```bash
sudo systemctl start pinot-controller pinot-broker pinot-server pinot-minion
```

Make sure the log directories exist, otherwise systemd may fail to write logs.


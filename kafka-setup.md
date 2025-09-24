## **Kafka Installation**

**Download Kafka 3.5.1**

```jsx
wget https://archive.apache.org/dist/kafka/3.5.1/kafka_2.13-3.5.1.tgz
```

---

**Extract the Archive**

```jsx
tar -xzf kafka_2.13-3.5.1.tgz
```

---

**Navigate to the Kafka Directory**

```jsx
cd kafka_2.13-3.5.1
```

---

**Start Zookeeper**

Kafka requires Zookeeper to manage distributed brokers. Start Zookeeper using the provided configuration:

```jsx
bin/zookeeper-server-start.sh config/zookeeper.properties
```

---

## **Remote access**

If you want to connect **from outside the server**:

Edit config/server.properties

This file can be found inside kafka_2.13-3.5.1 folder

listeners=PLAINTEXT://0.0.0.0:9092

advertised.listeners=PLAINTEXT://<your-server-ip>:9092

---

Restart Kafka:

```jsx
sudo systemctl restart kafka
```

---

### **Kafka Systemd**

```jsx
sudo tee /etc/systemd/system/kafka.service > /dev/null <<'EOF'

[Unit]

Description=Apache Kafka Service

After=network.target zookeeper.service

[Service]

User=devops

Group=devops

WorkingDirectory=/home/devops/kafka_2.13-3.5.1

ExecStart=/home/devops/kafka_2.13-3.5.1/bin/kafka-server-start.sh /home/devops/kafka_2.13-3.5.1/config/server.properties

ExecStop=/home/devops/kafka_2.13-3.5.1/bin/kafka-server-stop.sh

Restart=on-abnormal

LimitNOFILE=100000

[Install]

WantedBy=multi-user.target

EOF
```

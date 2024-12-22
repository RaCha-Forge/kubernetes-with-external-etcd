STEPS TO SETUP ETCD CLUSTER

# Step by Step
- [Step 1: Download the etcd binary](#step-1-download-the-etcd-binary)
- [Step 2: Copy the generated certificates](#step-2-copy-the-generated-certificates)
- [Step 3: Setup etcd as systemctl process](#step-3-setup-etcd-as-systemctl-process)
- [Step 4: Testing the cluster](#step-4-testing-the-cluster)


-----------------------------------------------------------
-----------------------------------------------------------
## Step 1: Download the etcd binary

Download the binary of the etcd in each of the server.

```bash
cd ~
wget https://github.com/etcd-io/etcd/releases/download/v3.3.13/etcd-v3.3.13-linux-amd64.tar.gz
tar -zxvf etcd-v3.3.13-linux-amd64.tar.gz
cd etcd-v3.3.13-linux-amd64/
sudo mv etcd etcdctl /usr/bin/
cd ~
rm -rf etcd-v3.3.13-linux-amd64*
```

-----------------------------------------------------------
-----------------------------------------------------------
## Step 2: Copy the generated certificates

Copy the respective generated certificated to respective node.
```bash
mkdir -p /etc/etcd/pki

cp node1-peer.pem node1-peer-key.pem node1.pem node1-key.pem client.pem client-key.pem new-etcd-root-ca.pem /etc/etcd/pki/
```

-----------------------------------------------------------
-----------------------------------------------------------
## Step 3: Setup etcd as systemctl process

```bash
export NODE_FOR_CERT_NAME=node1
export NODE_1_IP=""
export NODE_2_IP=""
export NODE_3_IP=""
export NODE_1_NAME=""
export NODE_2_NAME=""
export NODE_3_NAME=""

cat > /etc/systemd/system/etcd.service <<EOF
[Unit]
Description=etcd

[Service]
Type=notify
ExecStart=/usr/bin/etcd \\
  --name ${NODE_1_NAME} \\
  --trusted-ca-file=/etc/etcd/pki/new-etcd-root-ca.pem \\
  --cert-file=/etc/etcd/pki/${NODE_FOR_CERT_NAME}.pem \\
  --key-file=/etc/etcd/pki/${NODE_FOR_CERT_NAME}-key.pem \\
  --peer-trusted-ca-file=/etc/etcd/pki/new-etcd-root-ca.pem \\
  --peer-cert-file=/etc/etcd/pki/${NODE_FOR_CERT_NAME}-peer.pem \\
  --peer-key-file=/etc/etcd/pki/${NODE_FOR_CERT_NAME}-peer-key.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${NODE_1_IP}:2380 \\
  --listen-peer-urls https://${NODE_1_IP}:2380 \\
  --advertise-client-urls https://${NODE_1_IP}:2379 \\
  --listen-client-urls https://${NODE_1_IP}:2379,https://127.0.0.1:2379 \\
  --initial-cluster-token etcd-cluster \\
  --initial-cluster ${NODE_1_NAME}=https://${NODE_1_IP}:2380,${NODE_2_NAME}=https://${NODE_2_IP}:2380,${NODE_3_NAME}=https://${NODE_3_IP}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable etcd
systemctl start etcd.service
systemctl status etcd.service

```

Create etcd.service file in same way mentioned above in all nodes and execute mentioned steps after creating etcd.service file

-----------------------------------------------------------
-----------------------------------------------------------
## Step 4: Testing the cluster


In order to test the cluster, use the generated certs. Replace the NODE_IP with the specific node ip and execute the below command.

```bash
etcdctl  --endpoints=https://${NODE_IP}:2379 --ca-file /etc/etcd/pki/new-etcd-root-ca.pem --cert-file /etc/etcd/pki/client.pem --key-file /etc/etcd/pki/client-key.pem  cluster-health
```

If you get the below output, the cluster is working fine.

```bash
cluster is healthy
```
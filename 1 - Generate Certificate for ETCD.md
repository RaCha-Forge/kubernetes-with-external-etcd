STEPS TO GENERATE CERTIFICATES FOR ETCD CLUSTER

## Step by Step
- [Step 1: Download and setup cfssl](#step-1-download-and-setup-cfssl)
- [Step 2: Create a Certificate Authority (CA)](#step-2-create-a-certificate-authority-ca)
- [Step 3: Create TLS certificates](#step-3-create-tls-certificates)
- [Step 4: Copy the certificates to etcd nodes](#step-4-copy-the-certificates-to-etcd-nodes)



## Step 1: Download and setup cfssl

```bash
mkdir ~/bin
curl -s -L -o ~/bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
curl -s -L -o ~/bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
chmod +x ~/bin/{cfssl,cfssljson}
export PATH=$PATH:~/bin
```


## Step 2: Create a Certificate Authority (CA)

> We then use this CA to create other TLS certificates

```json

cat > ca-config.json <<EOF
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "etcd": {
                "expiry": "87600h",
                "usages": ["signing","key encipherment","server auth","client auth"]
            }
        }
    }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "etcd cluster",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IN",
      "L": "Bengaluru",
      "O": "Kubernetes",
      "OU": "ETCD-CA",
      "ST": "Karnataka"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca


```

## Step 3: Create TLS certificates

```json

ETCD1_IP="172.16.16.221"
ETCD2_IP="172.16.16.222"
ETCD3_IP="172.16.16.223"

cat > etcd-csr.json <<EOF
{
  "CN": "etcd",
  "hosts": [
    "localhost",
    "127.0.0.1",
    "${ETCD1_IP}",
    "${ETCD2_IP}",
    "${ETCD3_IP}"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IN",
      "L": "Bengaluru",
      "O": "Kubernetes",
      "OU": "etcd",
      "ST": "Karnataka"
    }
  ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd etcd-csr.json | cfssljson -bare etcd


```

## Step 4: Copy the certificates to etcd nodes

```bash
{

declare -a NODES=(172.16.16.221 172.16.16.222 172.16.16.223)

for node in ${NODES[@]}; do
  scp ca.pem etcd.pem etcd-key.pem root@$node: 
done

}
```

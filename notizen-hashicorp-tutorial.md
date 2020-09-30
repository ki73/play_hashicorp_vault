# Notizen Hashicorp Vault HA Cluster Raft Storage Tutorial

Tutorial: https://learn.hashicorp.com/tutorials/vault/raft-storage

Git URL: https://github.com/hashicorp/vault-guides/tree/master/operations/raft-storage/local

```bash
####################
demo_home="$(pwd)"
script_name="$(basename "$0")"
os_name="$(uname -s | awk '{print tolower($0)}')"
#################### 
$ ./cluster.sh create network
sudo ip addr add 127.0.0.2/8 dev lo label lo:0
sudo ip addr add 127.0.0.3/8 dev lo label lo:1
sudo ip addr add 127.0.0.4/8 dev lo label lo:2

#################### 
$ ./cluster.sh create config
rm -f config-vault_2.hcl
rm -rf "$demo_home"/raft-vault_2
mkdir -pm 0755 "$demo_home"/raft-vault_2

tee "$demo_home"/config-vault_2.hcl 1> /dev/null <<EOF
...
EOF

#################### 
$ ./cluster.sh setup all
# Einzelner Aufruf der setup Funktionen für vault_1..4

### Vault 1
# Start Vault
$VAULT_TOKEN VAULT_API_ADDR=http://127.0.0.1..4:8200 vault server -log-level=trace -config "$vault_config_file" > "$vault_log_file" 2>&1 &

# Setup
export VAULT_ADDR=http://127.0.0.1:8200
vault operator init -format=json -key-shares 1 -key-threshold 1
# Herauslesen von Unseal key und Root token (VAULT_TOKEN)
vault operator unseal $UNSEAL_KEY
vault login $VAULT_TOKEN
vault write -f transit/keys/unseal_key

### Vault 2
# Start Vault 2
export VAULT_ADDR=http://127.0.0.2:8200
vault operator init -format=json -recovery-shares 1 -recovery-threshold 1
# Herauslesen vom RECOVERY_KEY und VAULT_TOKEN
vault login $VAULT_TOKEN
vault secrets enable -path=kv kv-v2
vault kv put kv/apikey webapp=4711
vault kv get kv/apikey

#################### 
$ ./cluster.sh vault_3 operator raft join http://127.0.0.2:8200
export VAULT_ADDR=http://127.0.0.3:8200
vault operator raft join http://127.0.0.2:8200
#################### 
$ ./cluster.sh vault_2 operator raft list-peers
export VAULT_ADDR=http://127.0.0.2:8200
vault operator raft list-peers

$ ./cluster.sh vault_3 operator raft list-peers
export VAULT_ADDR=http://127.0.0.3:8200
vault operator raft list-peers
#################### 
$ ./cluster.sh stop [vault_1|vault_2|vault_3|vault_4|all]
# --> kill :-) 
#################### 
$ ./cluster.sh start [vault_1|vault_2|vault_3|vault_4|all]
# s.o.
#################### 
$ ./cluster.sh vault_1 status
export VAULT_ADDR=http://127.0.0.1:8200
vault status
#################### 
```

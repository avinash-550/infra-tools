
## Docker 
```bash
# start docker container with vault server (access ui at http://localhost:8200/ui)
docker run --volume config:/vault/config.d --cap-add=IPC_LOCK -e 'VAULT_LOCAL_CONFIG={"storage": {"file": {"path": "/vault/file"}}, "listener": [{"tcp": { "address": "0.0.0.0:8200", "tls_disable": true}}], "default_lease_ttl": "168h", "max_lease_ttl": "720h", "ui": true}' -p 8200:8200 hashicorp/vault server
## tip: if facing error in windows, use gitbash terminal

# verify server status (inside container)
vault status
```


## Features

### Secrets
Vault comes with various pluggable components called secrets engines and authentication methods allowing you to integrate with external systems. The purpose of those components is to manage and protect your secrets in dynamic infrastructure (e.g. database credentials, passwords, API keys).

```bash
# write a secret (v2 secrets engine - supports versioning, soft deletion, metadata)
vault kv put -mount=secret hello foo=world
# KV v1 secrets engine and the path prefix syntax instead (e.g. vault kv get secret/foo)

# write multiple keys (secret version will update to 2)
vault kv put -mount=secret hello foo=world excited=yes

# read a secret
vault kv get -mount=secret -field=excited hello

# json output
vault kv get -mount=secret -format=json hello | jq -r .data.data.excited

# delete a secret 
vault kv delete -mount=secret hello

# secret is deleted but not destroyed recover using
vault kv undelete -mount=secret -versions=2 hello

```
### Secret engines
Secret engines in Vault are components that store, generate, or encrypt data. Each secret engine is responsible for managing a specific type of secret or set of secrets. Vault can manage a variety of secret engines, each designed for specific use cases, such as key/value storage, databases, cloud services, and more. Secret engines are enabled at a specific path, and once enabled, they can be configured and used to manage secrets in a way that is isolated from other secret engines. This allows Vault to be highly flexible and to securely manage a wide range of secret types and workflows.


```bash
# enable a new secrets engine
vault secrets enable -path=kv kv
# When a request comes to Vault, it matches the initial path part using a longest prefix match and then passes the request to the corresponding secrets engine enabled at that path. Vault presents these secrets engines similar to a filesystem.
#  Each path is completely isolated and cannot talk to other paths. 

# list all the secret engines
vault secrets list [-detailed]

# list existing keys at secret engine
vault kv list kv/

# disable secrets engine
vault secrets disable kv/
```

### Authentication
Vault provides a variety of [authentication methods](https://developer.hashicorp.com/vault/docs/auth) for the human operators and machines. Token authentication is automatically enabled.

```bash
# create a new token
vault token create

# revoke unused tokens
vault token revoke s.iyNUhq8Ov4hIAx6snw5mB2nL

```

### Authorization

```bash
# read a policy
vault policy read default

# create a new policy
vault policy write my-policy - << EOF
# Dev servers have version 2 of KV secrets engine mounted by default, so will
# need these paths to grant permissions:
path "secret/data/*" {
  capabilities = ["create", "update"]
}

path "secret/data/foo" {
  capabilities = ["read"]
}
EOF

# list all policies
vault policy list
```
## References
1. https://developer.hashicorp.com/vault/docs/concepts/dev-server
2. https://hub.docker.com/r/hashicorp/vault
3. https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-apis

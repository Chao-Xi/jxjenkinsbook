# 第二节(Extra) - Store Secrets using Hashicorp Vault with Docker

## 1.What is Hashicorp Vault

Vault is a tool for securely accessing secrets. A secret is anything that you want to tightly control access to, such as API keys, passwords, certificates, and more. Vault provides a unified interface to any secret, while providing tight access control and recording a detailed audit log. More details can be found at https://github.com/hashicorp/vault/


## 2.Configuration

The first step is to configure a Data Container to store the configuration for Vault.

```
$ touch vault.hcl

backend "consul" {
  address = "consul:8500"
  advertise_addr = "consul:8300"
  scheme = "http"
}
listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = 1
}
disable_mlock = true
```

The config defines three important properties. 

* Firstly, it sets Vault to use **Consul** to store the secrets. Using Consul enables high availability mode as **Consul manages to information and distribution to ensure HA.** 
* Secondly, it binds Vault to listen on all IP addresses, this is for use with the HTTP API. 
* Finally, for development purposes, **we disable TLS**.

### Create Data Container

To store the configuration we'll create a container. This will be used by Vault and Consul to read the required configuration files.

```
docker create -v /config --name config busybox; docker cp vault.hcl config:/config/;
```

## 3.Launch

With the configuration data container created we can launch the necessary processes to start Vault.

### 3-1 Launch Services

Launch a single Consul agent. In production, we'd want to have a cluster of 3 or 5 agents as a single node can lead to data loss.

```
docker run -d --name consul -p 8500:8500 consul:1.8.0 agent -dev -client=0.0.0.0

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                                NAMES
bf950a5631cc        consul:v0.6.4       "docker-entrypoint.s…"   14 seconds ago      Up 13 seconds       8300-8302/tcp, 8400/tcp, 8301-8302/udp, 8600/tcp, 8600/udp, 0.0.0.0:8500->8500/tcp   consul
```

Our Vault instance can now use Consul to store the data. All data stored within Consul will be encrypted.

```
docker run -d --name vault-dev \
  --link consul:consul \
  -p 8200:8200 \
  --volumes-from config \
   cgswong/vault:0.5.3 server -config=/config/vault.hcl
```

* `--detach , -d`: Run container in background and print container ID
* `--link`: Add link to another container
* `--publish , -p`	: Publish a container’s port(s) to the host
* `--volumes-from`: Mount volumes from the specified container(s)


## 4. Initialise

With a vault instance running, we can now configure our environment and initialise the Vault.

### 4-1 Configure Environment

This command will create an alias which will proxy commands to vault to the Docker container. As we're in development, **we need to define a non-HTTPS address for the vault.**

```
alias vault='docker exec -it vault-dev vault "$@"'
export VAULT_ADDR=http://127.0.0.1:8200
```

### 4-2 Initialise Vault


With the alias in place, we can make calls to the CLI. The first step is to initialise the vault using the ***init*** command.

```
vault init -address=${VAULT_ADDR} > keys.txt
```

We store the results in `cat keys.txt`

```
cat keys.txt
Key 1: 04028656e5a5726aa2995540a30fe31663150d86ba36924cc2e65a69aa8463ce01
Key 2: 7dcbdb1003a37395e23181eb27bde8a4a68165251f57014ba518466da646c2c202
Key 3: 1bd673dda7ec11327f66dbac5abd0f5608d7ebdc3fd280128906c334607b148503
Key 4: 3d3911be81097c494374e0573b8e23aeab7ffd31b2991b34347ee01b8c7e762604
Key 5: 5b24b97325461eeede23ba10468ec45c052973c8921c9a6d186065424a43a06105
Initial Root Token: b550b1e3-042b-f327-0c44-41f05dcac228

Vault initialized with 5 keys and a key threshold of 3. Please
securely distribute the above keys. When the Vault is re-sealed,
restarted, or stopped, you must provide at least 3 of these keys
to unseal it again.

Vault does not store the master key. Without at least 3 keys,
your Vault will remain permanently sealed.
```

The file contains valuable information about how to access the vault and seal/unseal. The output includes our master key and token. We'll use this in the next step when communicating with our running instance.

## 5. Unseal Vault

When a Vault server is started, it starts in a **sealed** state. The server knows how to communicate with the backend storage, but it does not know how to decrypt any of the contents. Unsealing is the process of constructing the master key necessary to read the decryption key to decrypt the data, allowing read access to the Vault.

### 5-1 Unseal Vault

To unseal with Vault server you need access to three of the five keys defined when the Vault was initialised. The following command will grep the first three keys and unseal the Vault.

```
$ vault unseal -address=${VAULT_ADDR} $(grep 'Key 1:' keys.txt | awk '{print $NF}')
Sealed: true
Key Shares: 5
Key Threshold: 3
Unseal Progress: 1
```

```
$ vault unseal -address=${VAULT_ADDR} $(grep 'Key 2:' keys.txt | awk '{print $NF}')
Sealed: true
Key Shares: 5
Key Threshold: 3
Unseal Progress: 2
```


```
$ vault unseal -address=${VAULT_ADDR} $(grep 'Key 3:' keys.txt | awk '{print $NF}')

Sealed: false
Key Shares: 5
Key Threshold: 3
Unseal Progress: 0
```

In production, these keys should be stored separately and securely. Vault uses an algorithm known as Shamir's Secret Sharing to split the master key into shards. Each of the five keys is part of the shard.


### 5-2 Status

You can view the status of the vault using 


```
vault status -address=${VAULT_ADDR}
```

```
$ vault status -address=${VAULT_ADDR}
Sealed: false
Key Shares: 5
Key Threshold: 3
Unseal Progress: 0

High-Availability Enabled: true
        Mode: active
        Leader: consul:8300
```

## 6. Vault Tokens

Tokens are used to communicate with the Vault. When the vault was initialised, a root token was outputted. Store this in a variable with the following command. We'll use it for future API calls. export 

```
VAULT_TOKEN=$(grep 'Initial Root Token:' keys.txt | awk '{print substr($NF, 1, length($NF)-1)}')
```

You can use this token to login to vault. 

```
vault auth -address=${VAULT_ADDR} ${VAULT_TOKEN}
```

```
$ vault auth -address=${VAULT_ADDR} ${VAULT_TOKEN}
Successfully authenticated!
token: b550b1e3-042b-f327-0c44-41f05dcac228
token_duration: 0
token_policies: [root]
```


The primary reason is to create new tokens using vault **`token-create`**.


## 7. Read/Write Data

The Vault CLI can be used to read and write data securely. Vault is a primarily a key/value store. Each pairing can have additional security configuration attached, such as policies on access, TTL, and the ability to be revoked.

### 7-1 Save Data

To store data, we use the write CLI command. In this case, we have a key named `secret/api-key` with the value `12345678`

```
$ vault write -address=${VAULT_ADDR} \
> secret/api-key value=12345678
Success! Data written to: secret/api-key
```

### 7-2 Read Data

Reading the key will output the value, along with other information such as the lease duration. 

```
$ vault read -address=${VAULT_ADDR} \
	secret/api-key
Key             Value
lease_duration  2592000
value           12345678
```

You can also use the **`-field`** flag to extract the value from the secret data

```
$ vault read -address=${VAULT_ADDR} \
> -field=value secret/api-key
12345678
```

## 8. HTTP API


```
$ curl -H "X-Vault-Token:$VAULT_TOKEN" -XGET http://127.0.0.1:8200/v1/secret/api-key
{"lease_id":"","renewable":false,"lease_duration":2592000,"data":{"value":"12345678"},"warnings":null,"auth":null}
```

Using the command like tool **jq** we can parse the data and extract the value for our key.

```
curl -s -H  "X-Vault-Token:$VAULT_TOKEN" \
	 -XGET http://127.0.0.1:8200/v1/secret/api-key \
	| jq -r .data.value
12345678
```

This could be used to launch additional processes, such as MySQL, or access external services like AWS. Using this as part of a Docker Entrypoint will be discussed in future scenarios.


# Quick-guide-Hashicorp-vault
# Hashicorp Vault

We will deep dive into the basics of Vault and its different  use cases.

It is a popular tool it has a lot of use cases like -

- Secrets - Key/Secrets management.
- Data Encryption -  Encryption for our applications.
- Identity -  Roles and User management.

**Initialize the Vault -**

- Download the binary

```json
wget [https://releases.hashicorp.com/vault/1.3.0/vault_1.3.0_linux_amd64.zip](https://releases.hashicorp.com/vault/1.3.0/vault_1.3.0_linux_amd64.zip) && unzip vault_1.3.0_linux_amd64.zip && mv vault /usr/local/bin
```

- Pre-setup for Vault CLI `dev`mode

```json
echo "complete -o nospace -C /usr/bin/vault vault" >> ~/.bashrc

vault server -dev
```

- Set up Interaction with Vault API

```json
export VAULT_ADDR='[http://127.0.0.1:8200](http://127.0.0.1:8200/)'
```

- Basic CLI Functions

```json
vault status
```

- Vault `path-help`

```json
vault path-help secret/data/put/
```

- Vault Key-Value Secret Backend - Store a secret

```json
vault kv put secret/password value=h4ck9992002
```

- Retrieve KV Secret

```json
vault kv get secret/password
```

- List KV Secrets
    
    ```json
    vault kv list secret
    ```
    
- Update a KV Secret (without overwriting)

```json
vault kv patch secret/password value=HelloWorld
```

- Notice how the secret has been created as `v2` of the same secret
- Vault has the option of version managing secrets as well
- Retrieve version of the secret
    
    ```json
    vault kv get -version 1 secret/password
    ```
    
- Metadata for a secret

```
vault kv metadata get secret/passwordCopy
```

### **Adding options while creating key**

- Options include:
    - Max number of versions
    - delete time for secret => Ephemeral secrets that delete after X time

```
vault kv metadata put -max-versions 4 -delete-version-after="1h" secret/passwordCopy
```

- Enable Transit Backend
    
    ```json
    vault secrets enable transit
    ```
    
- create a Key to use for encrypt/decrypt operations
    
    ```json
    vault write -f transit/keys/my-crypto-key
    ```
    
- encrypt some data with this key
    
    ```json
    vault write transit/encrypt/my-crypto-key plaintext=$(base64 <<< "super secret data")
    ```
    
- • Notice that the version of this encrypted ciphertext is `v1` if the key is rotated and the data is "re-wrapped", this becomes `v2`
- Decrypt the data
    
    ```json
    vault write transit/decrypt/my-crypto-key ciphertext=<your-new-ciphertext>
    
    echo "<base64-value>" | base64 -d
    ```
    
- Rotating Keys
    
    ```json
    vault write -f transit/keys/my-crypto-key/rotate
    ```
    
- Enabling the Database Backend
    
    ```json
    vault secrets enable database
    ```
    
- Start a mongoDB Database instance with docker on port `27017`
    
    ```json
    docker run --rm -d -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=mongoadmin -e MONGO_INITDB_ROOT_PASSWORD=strongPass mongo:4.4
    ```
    
- Connect Mongo with Vault and add a role for Vault to generate credentials for mongo
    
    ```json
    vault write database/config/my-mongodb-database \
        plugin_name=mongodb-database-plugin \
        allowed_roles="my-role" \
        connection_url="mongodb://{{username}}:{{password}}@localhost:27017/admin" \
        username="mongoadmin" \
        password="strongPass"
    ```
    
- Add a role called `my-role` that will authorize vault to generate credentials for the MongoDB Instance.
    
    ```json
    vault write database/roles/my-role \
        db_name=my-mongodb-database \
        creation_statements='{ "db": "admin", "roles": [{ "role": "readWrite" }, {"role": "read", "db": "foo"}] }' \
        default_ttl="1h" \
        max_ttl="24h"
    ```
    
- Generate some dynamic creds for a user to MongoDB
    
    ```json
    vault read database/creds/my-role
    ```
    
- Enable User-Login Authentication
    
    ```json
    vault auth enable userpass
    ```
    
- Create a user
    
    ```json
    vault write auth/userpass/users/subhajit password=C0leisG0aTTT policies=admins
    ```
    
- Login
    
    ```json
    vault login -method=userpass username=subhajit password=C0leisG0aTTT
    ```
    
- Audit
    
    ```json
    vault audit enable file file_path=/var/log/vault_audit.log
    ```

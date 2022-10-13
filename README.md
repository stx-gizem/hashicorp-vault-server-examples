## Hashicorp Vault Examples

Run docker compose and install vault-server for development environment
*This setup is for testing Hashicorp Vault in development environment only.*

	cp .env.example .env
	
Set VAULT_DEV_ROOT_TOKEN in .env file

    docker compose up -d

Go to http://localhost:8200. You can login with your vault dev root token. Steps done using vault cli can also be done via ui.


Go to vault-server container bash

    docker exec -it vault-server sh

Login vault cli with root token and create Key/Value (KV) secrets group

    vault-server> vault login
    vault-server> vault secrets enable -version=1 -path=sample-app kv

Here, kv was used as the secret engine. [But the vault allows for many options.](https://www.vaultproject.io/docs/secrets)

 Create a policy that has read access to the secret engine.
 There is a sample policy in the .volumes/policies/sample-app.hcl file. It allows reading all key values in the sample-app directory. [You can define more features in vault policy](https://www.vaultproject.io/docs/concepts/policies)

    vault-server> vault policy write sampleapp /policies/sample-app.hcl


Create a key in sample-app secrets engine

    vault-server> vault kv put sample-app/development/config PG_USER=gizem

Read keys

    vault-server> vault kv get sample-app/development/config

Put key-values from json file

    vault-server> vault kv put -mount=sample-app mongo @/config/sample-app.json

Create a token with sampleapp policy

    vault-server> vault token create -policy=sampleapp

Clients with the token we created can now read keys, values in the sample-app/* directory.

Test the token, like a vault client 

    curl --location --request GET 'http://localhost:8200/v1/sample-app/development/config' \
    --header 'X-Vault-Token: <CREATED_TOKEN>'

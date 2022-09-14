# Install Vault

`brew install vault`

First, install the HashiCorp tap, a repository of all our Homebrew packages.

`brew tap hashicorp/tap`

Now, install Vault with hashicorp/tap/vault.

`brew install hashicorp/tap/vault`

To update to the latest, run

`brew upgrade hashicorp/tap/vault`

After installing Vault, verify the installation worked by opening a new terminal session and checking that the vault binary is available. By executing vault, you should see help output similar to the following:

`vault`

Copy and run the export VAULT_ADDR ... command from the terminal output. This will configure the Vault client to talk to the dev server.

`export VAULT_ADDR='http://127.0.0.1:8200'`

Set the VAULT_TOKEN environment variable value to the generated Root Token value displayed in the terminal output.
Root Token: `hvs.dB8zlL5K3oKJnHCWf4s12jGU`
`export VAULT_TOKEN="hvs.dB8zlL5K3oKJnHCWf4s12jGU"`

Save the unseal key somewhere. Don't worry about how to save this securely. For now, just save it anywhere.

Unseal Key: `+shFQnDqAogcv+ugQ0cNkuU4/eNf4iTb/o8BX+Nrsns=`

Verify the server is running by running the `vault status` command. If it ran successfully, the output should look like the following:

When running Vault in dev mode, Key/Value v2 secrets engine is enabled at secret/ path. Key/Value secrets engine is a generic key-value store used to store arbitrary secrets within the configured physical storage for Vault. Secrets written to Vault are encrypted and then written to backend storage. Therefore, the backend storage mechanism never sees the unencrypted value and doesn't have the means necessary to decrypt it without Vault.

Key/Value secrets engine has version 1 and 2. The difference is that v2 provides versioning of secrets and v1 does not.

Use the vault kv <subcommand> [options] [args] command to interact with K/V secrets engine.

`vault kv -help`

Now, write a key-value secret to the path hello , with a key of foo and value of world, using the vault kv put command against the mount path secret, which is where the KV v2 secrets engine is mounted. This command creates a new version of the secrets and replaces any pre-existing data at the path if any.

`vault kv put -mount=secret hello foo=world`

As you might expect, secrets can be retrieved with vault kv get.

`vault kv get -mount=secret hello`

To print only the value of a given field, use the -field=<key_name> flag.

`vault kv get -mount=secret -field=excited hello`

Now that you've learned how to read and write a secret, let's go ahead and delete it. You can do so using the vault kv delete command.

`vault kv delete -mount=secret hello`



Try to read the secret you just deleted.

`vault kv get -mount=secret hello`

`vault kv undelete -mount=secret -versions=2 hello`



![vaultFlowChart](images/vaultFlowChart.png)



Enable a secrets engine
To get started, enable a new KV secrets engine at the path kv. Each path is completely isolated and cannot talk to other paths. For example, a KV secrets engine enabled at foo has no ability to communicate with a KV secrets engine enabled at bar.

`vault secrets enable -path=kv kv`

Success! Enabled the kv secrets engine at: kv/



The path where the secrets engine is enabled defaults to the name of the secrets engine. Thus, the following command is equivalent to executing the above command.

`vault secrets enable kv`

`vault secrets list`


Create secrets at the kv/my-secret path.

`vault kv put kv/my-secret value="s3c(eT"`
Success! Data written to: kv/my-secret



Read the secrets at kv/my-secret.

`vault kv get kv/my-secret`

==== Data ====
Key      Value
---      -----
value    s3c(eT



Delete the secrets at kv/my-secret.

`vault kv delete kv/my-secret`
Success! Data deleted (if it existed) at: kv/my-secret



List existing keys at the kv path.

`vault kv list kv/`

Keys
----
hello



Disable a secrets engine
When a secrets engine is no longer needed, it can be disabled. When a secrets engine is disabled, all secrets are revoked and the corresponding Vault data and configuration is removed.

`vault secrets disable kv/`

Success! Disabled the secrets engine (if it existed) at: kv/


Vault behaves similarly to a virtual filesystem. The read/write/delete/list operations are forwarded to the corresponding secrets engine, and the secrets engine decides how to react to those operations.

This abstraction is incredibly powerful. It enables Vault to interface directly with physical systems, databases, HSMs, etc. But in addition to these physical systems, Vault can interact with more unique environments like AWS IAM, dynamic SQL user creation, etc. all while using the same read/write interface.

You learned the basics of the vault secrets command. This is important knowledge to move forward and explore other secrets engines.



Unlike the kv secrets where you had to put data into the store yourself, dynamic secrets are generated when they are accessed. Dynamic secrets do not exist until they are read, so there is no risk of someone stealing them or another client using the same secrets. Because Vault has built-in revocation mechanisms, dynamic secrets can be revoked immediately after use, minimizing the amount of time the secret existed.
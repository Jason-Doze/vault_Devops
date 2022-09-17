# Install Vault

`brew install vault`

First, install the HashiCorp tap, a repository of all our Homebrew packages.

`brew tap hashicorp/tap`

## Now, install Vault with hashicorp/tap/vault.

`brew install hashicorp/tap/vault`

## To update to the latest, run

`brew upgrade hashicorp/tap/vault`

<br>

After installing Vault, verify the installation worked by opening a new terminal session and checking that the vault binary is available. By executing vault, you should see help output similar to the following:

`vault`

<br>

Copy and run the export VAULT_ADDR ... command from the terminal output. This will configure the Vault client to talk to the dev server.

`export VAULT_ADDR='http://127.0.0.1:8200'`

<br>

Set the VAULT_TOKEN environment variable value to the generated Root Token value displayed in the terminal output.

<br>

Root Token: 

`hvs.dB8zlL5K3oKJnHCWf4s12jGU`
`export VAULT_TOKEN="hvs.dB8zlL5K3oKJnHCWf4s12jGU"`

<br>

Save the unseal key somewhere. Don't worry about how to save this securely. For now, just save it anywhere.

<br>

Unseal Key: 

Verify the server is running by running the `vault status` command. If it ran successfully, the output should look like the following:

When running Vault in dev mode, Key/Value v2 secrets engine is enabled at secret/ path. Key/Value secrets engine is a generic key-value store used to store arbitrary secrets within the configured physical storage for Vault. Secrets written to Vault are encrypted and then written to backend storage. Therefore, the backend storage mechanism never sees the unencrypted value and doesn't have the means necessary to decrypt it without Vault.

<br>

Key/Value secrets engine has version 1 and 2. The difference is that v2 provides versioning of secrets and v1 does not.

<br>

Use the vault kv <subcommand> [options] [args] command to interact with K/V secrets engine.

`vault kv -help`

<br>

Now, write a key-value secret to the path hello , with a key of foo and value of world, using the vault kv put command against the mount path secret, which is where the KV v2 secrets engine is mounted. This command creates a new version of the secrets and replaces any pre-existing data at the path if any.

`vault kv put -mount=secret hello foo=world`

<br>

As you might expect, secrets can be retrieved with vault kv get.

`vault kv get -mount=secret hello`

<br>

To print only the value of a given field, use the -field=<key_name> flag.

`vault kv get -mount=secret -field=excited hello`

<br>

Now that you've learned how to read and write a secret, let's go ahead and delete it. You can do so using the vault kv delete command.

`vault kv delete -mount=secret hello`

<br>

Try to read the secret you just deleted.

`vault kv get -mount=secret hello`

`vault kv undelete -mount=secret -versions=2 hello`

<br>

![vaultFlowChart](images/vaultFlowChart.png)

<br>

Enable a secrets engine

---

To get started, enable a new KV secrets engine at the path kv. Each path is completely isolated and cannot talk to other paths. For example, a KV secrets engine enabled at foo has no ability to communicate with a KV secrets engine enabled at bar.

`vault secrets enable -path=kv kv`

Success! Enabled the kv secrets engine at: kv/

<br>



The path where the secrets engine is enabled defaults to the name of the secrets engine. Thus, the following command is equivalent to executing the above command.

`vault secrets enable kv`

`vault secrets list`

<br>

Create secrets at the kv/my-secret path.

`vault kv put kv/my-secret value="s3c(eT"`
Success! Data written to: kv/my-secret

<br>

Read the secrets at kv/my-secret.

`vault kv get kv/my-secret`
```
==== Data ====
Key      Value
---      -----
value    s3c(eT
```

<br>

Delete the secrets at kv/my-secret.

`vault kv delete kv/my-secret`
Success! Data deleted (if it existed) at: kv/my-secret

<br>

List existing keys at the kv path.

`vault kv list kv/`
```
Keys
----
hello
```

<br>

Disable a secrets engine
When a secrets engine is no longer needed, it can be disabled. When a secrets engine is disabled, all secrets are revoked and the corresponding Vault data and configuration is removed.

`vault secrets disable kv/`

Success! Disabled the secrets engine (if it existed) at: kv/

<br>


Vault behaves similarly to a virtual filesystem. The read/write/delete/list operations are forwarded to the corresponding secrets engine, and the secrets engine decides how to react to those operations.

This abstraction is incredibly powerful. It enables Vault to interface directly with physical systems, databases, HSMs, etc. But in addition to these physical systems, Vault can interact with more unique environments like AWS IAM, dynamic SQL user creation, etc. all while using the same read/write interface.

You learned the basics of the vault secrets command. This is important knowledge to move forward and explore other secrets engines.

<br>

Unlike the kv secrets where you had to put data into the store yourself, dynamic secrets are generated when they are accessed. Dynamic secrets do not exist until they are read, so there is no risk of someone stealing them or another client using the same secrets. Because Vault has built-in revocation mechanisms, dynamic secrets can be revoked immediately after use, minimizing the amount of time the secret existed.

<br>

Enable the AWS secrets engine
Unlike the kv secrets engine which is enabled by default, the AWS secrets engine must be enabled before use. This step is usually done via a configuration management system.

`vault secrets enable -path=aws aws`
Success! Enabled the aws secrets engine at: aws/

<br>

`export AWS_ACCESS_KEY_ID=<aws_access_key_id>`


<br>

Next, set your secret key.

`export AWS_SECRET_ACCESS_KEY=<aws_secret_key>`

<br>

Your keys must have the IAM permissions listed in the Vault documentation to perform the actions on the rest of this page.

<br>

Configure the AWS secrets engine.

`vault write aws/config/root \
    access_key=$AWS_ACCESS_KEY_ID \
    secret_key=$AWS_SECRET_ACCESS_KEY \
    region=us-east-1`

Success! Data written to: aws/config/root

<br>

Create a role

---
The next step is to configure a role. Vault knows how to create an IAM user via the AWS API, but it does not know what permissions, groups, and policies you want to attach to that user. This is where roles come in - a role in Vault is a human-friendly identifier to an action.

For example, here is an IAM policy that enables all actions on EC2, but not IAM or other AWS services.
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1426528957000",
      "Effect": "Allow",
      "Action": ["ec2:*"],
      "Resource": ["*"]
    }
  ]
}
```

<br>

You need to map this policy document to a named role. To do that, write to aws/roles/:name where :name is your unique name that describes the role (such as aws/roles/my-role):

<br>

```
vault write aws/roles/my-role \
        credential_type=iam_user \
        policy_document=-<<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1426528957000",
      "Effect": "Allow",
      "Action": [
        "ec2:*"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
EOF
```

Success! Data written to: aws/roles/my-role

<br>

You just told Vault:

When I ask for a credential for "my-role", create it and attach the IAM policy { "Version": "2012..." }.

<br>

Generate the secret

---

Now that the AWS secrets engine is enabled and configured with a role, you can ask Vault to generate an access key pair for that role by reading from aws/creds/:name, where :name corresponds to the name of an existing role:

`vault read aws/creds/my-role`

<br>

```
Key                Value
---                -----
lease_id           aws/creds/my-role/ZHc0pyjoEkgz1crEaIazHhRD
lease_duration     768h
lease_renewable    true
access_key         AKIA6QFXKASVRSOVUTXM
secret_key        
security_token     <nil>
```

<br>

Success! The access and secret key can now be used to perform any EC2 operations within AWS. Notice that these keys are new, they are not the keys you entered earlier. If you were to run the command a second time, you would get a new access key pair. Each time you read from aws/creds/:name, Vault will connect to AWS and generate a new IAM user and key pair.

Copy the full path of this lease_id value found in the output. This value is used for renewal, revocation, and inspection.


Revoke the secret

---
Vault will automatically revoke this credential after 768 hours (see lease_duration in the output), but perhaps you want to revoke it early. Once the secret is revoked, the access keys are no longer valid.

To revoke the secret, use vault lease revoke with the lease ID that was outputted from vault read when you ran it.

Example:

`vault lease revoke aws/creds/my-role/0bce0782-32aa-25ec-f61d-c026ff22106`

Success! Revoked lease: aws/creds/my-role/0bce0782-32aa-25ec-f61d-c026ff22106e

<br>

Done! If you login to your AWS account, you will see that no IAM users exist. If you try to use the access keys that were generated, you will find that they no longer work.

<br>

With the secrets engine enabled, learn about it with the vault path-help command:

`vault path-help`

`vault path-help aws`

<br>

Token authentication

---
Token authentication is automatically enabled. When you started the dev server, the output displayed a root token. The Vault CLI read the root token from the $VAULT_TOKEN environment variable. This root token can perform any operation within Vault because it is assigned the root policy. One capability is to create new tokens.

Create a new token.

`vault token create`

```
Key                  Value
---                  -----
token                hvs.bOuvZxW5XpyTyv1vVN9zLR7n
token_accessor       0xddC9HdDQTm8gm9xtZe7bOY
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

jasondoze@jd vault % 

<br>

Press Ctrl+C to terminate the dev server that is running at http://127.0.0.1:8200 before starting.

<br>

Also, unset the VAULT_TOKEN environment variable.

`unset VAULT_TOKEN`

<br>

By default, Homebrew will install all packages on Mac in the directory /usr/local/Cellar/, and also creates symbolic links at /usr/local/opt/ and /usr/local/bin/ (for executable files).

`ls -lsa /usr/local/Cellar`

`brew info vault`

```
jasondoze@jd vault % brew info vault
==> vault: stable 1.11.3 (bottled), HEAD
Secures, stores, and tightly controls access to secrets
https://vaultproject.io/
/usr/local/Cellar/vault/1.11.3 (8 files, 187.4MB) *
  Poured from bottle on 2022-09-13 at 19:08:51
From: https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/vault.rb
License: MPL-2.0
==> Dependencies
Build: go ✘, gox ✘, node ✘, python@3.10 ✘, yarn ✘
==> Options
--HEAD
      Install HEAD version
==> Caveats
To restart vault after an upgrade:
  brew services restart vault
Or, if you don't want/need a background service you can just run:
  /usr/local/opt/vault/bin/vault server -dev
==> Analytics
install: 14,115 (30 days), 42,716 (90 days), 135,154 (365 days)
install-on-request: 13,544 (30 days), 41,209 (90 days), 131,015 (365 days)
build-error: 0 (30 days)
```
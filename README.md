# Secrets
Local [Vault](https://www.vaultproject.io) setup, whose data is stored on [Consul](https://www.consul.io), with a demo showing how to manage secrets from an AWS back-end server.

It's based on [Michael Herman's](https://testdriven.io/authors/herman/) article which can be found [here](https://testdriven.io/blog/managing-secrets-with-vault-and-consul/).

## Usage

Create images running:
```
docker-compose up -d
```

### Vault:
Since Vault looks based on `alpine` images, we can't `bash` on its container. So you need to use `sh`:
```
docker exec -it CONTAINER sh
```

Once you're connected to its container, you need to init Vault:
```
vault operator init
```

You can access Vault by the [UI](http://0.0.0.0:8002/ui/vault) or inside its container, via CLI.

#### Features
You can enable audit by running:
```
vault audit enable file file_path=/path/to/store/logs
```   

You can create policies to manage access control on Vault. There's a JSON file on the `vault/policies` directory with a 
policy example. You can enable it by running:
```
vault policy write app ./vault/policies/app-policy.json

vault token create -policy=app
```

#### Transit
It's possible to use Transit, what Hashicorp calls "Encryption as a Service", running:
```
vault secrets enable transit
```

Then you can:
- Create a named encryption key
```
vault write -f transit/keys/foo
```

- Encrypt data
```
vault write transit/encrypt/foo plaintext=$(base64 <<< "my precious")
```

- Decrypt data (you need to decode the resulting data)
```
vault write transit/decrypt/foo ciphertext=vault:v1:/Tun95IT+dVTvDfYiCHdI5rGPSAxgvPcFaDDtneRorQCyBOg (Decrypt)
base64 -d <<< "bXkgcHJlY2lvdXMK" (Decode)
```

#### AWS Back-end Secrets
You can enable AWS back-end secrets by running:
```
vault secrets enable -path=aws aws
```

Then you need to authenticate:
```
vault write aws/config/root access_key=foo secret_key=bar
```

To create a role:
```
vault write aws/roles/ec2-read arn=arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
```

To create a new credential group, which can be seen in the "Users" section of the IAM Console:
```
vault read aws/creds/ec2-read
```

To change the lease interval to 30 minutes, for example. Read more [here](https://www.vaultproject.io/guides/identity/lease.html).
```
vault write aws/config/lease lease=1800s lease_max=1800s
```
        
To revoke a credential:
```
vault lease revoke aws/creds/modify/{lease_id} (lease_id é obtido ao rodar o comando "vault read aws/creds/foo")
```
        
To revoke all AWS credentials:
```
vault revoke -prefix aws/
```

### Consul:
`consul-worker` in the docker-compose YML file is an example of another Consul server configured on this architecture.
# Secrets

Setup local de um Vault, cujos dados são guardados num Consul, com demonstração de gerenciamento de segredos em um servidor back-end AWS. Todo este setup é uma síntese do artigo publicado por Michael Herman, no link: https://testdriven.io/managing-secrets-with-vault-and-consul

### Setup:

* Ter o Consul e o Docker instalado

* Alterar os Dockerfile para ajustar as versões do Consul e do Vault conforme a necessidade

* Alterar o docker-compose de acordo conforme a necessidade do ambiente em que serão inseridos

* Dar build no Docker Compose, executando o comando:
    ```
    docker-compose up -d --build
    ```

### Vault:
* Para executar o bash do vault, executar o comando:
    ```
    docker-compose exec vault bash
    ```

* É necessário iniciar o vault uma vez, com o comando:
    ```
    vault operator init
    ```

* Pode-se ativar auditoria com o comando:
    ```
    vault audit enable file file_path=/path/para/guardar/os/logs
    ```   

* É possível navegar pelo vault pelo bash da imagem, por API ou pela UI (http://{ip_adress}:{vault_port}/ui/vault)

* É possível criar políticas de interação com a API com o seguinte comando:
    ```
    vault policy write app /path/to/policy/policy_file_name.json
    ```

    Na pasta vault/policies há um arquivo JSON de demonstração, após isso criar token com o comando:
    ```
    vault token create -policy=app
    ```
        
    Guardar este token para acessar a API com esta política

* É possível utilizar o Transit, aquilo que a HashiCorp chama de "Encryption as a Service" com o comando:
    ```
    vault secrets enable transit
    ```
        
    Outros comandos:
    ```
    vault write -f transit/keys/foo (Configurar uma chave nomeada de encriptação)
    vault write transit/encrypt/foo plaintext=$(base64 <<< "my precious") (Encriptar)
    vault write transit/decrypt/foo ciphertext=vault:v1:/Tun95IT+dVTvDfYiCHdI5rGPSAxgvPcFaDDtneRorQCyBOg (Decriptar)
    base64 -d <<< "bXkgcHJlY2lvdXMK" (Decode)
    ```
        

* Para habilitar segredos de um back-end AWS, executar o comando:
    ```
    vault secrets enable -path=aws aws
    ```

    Para autenticar:
    ```
    vault write aws/config/root access_key=foo secret_key=bar
    ```

    Para criar role:
    ```
    vault write aws/roles/ec2-read arn=arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
    ```

    Para criar um novo grupo de credenciais, que pode ser visto na seção Users do IAM Console:
    ```
    vault read aws/creds/ec2-read
    ```

    Para alterar o período de conceção para 30 minutos, por exemplo. Para mais informações acessar https://www.vaultproject.io/guides/identity/lease.html:
    ```
    vault write aws/config/lease lease=1800s lease_max=1800s
    ```
            
    Para revogar uma credencial:
    ```
    vault lease revoke aws/creds/modify/{lease_id} (lease_id é obtido ao rodar o comando "vault read aws/creds/foo")
    ```
            
    Para revogar todas as credenciais do AWS:
    ```
    vault revoke -prefix aws/
    ```

### Consul:
* O consul-worker no docker-compose.yml é um exemplo de outro servidor consul configurado nessa arquitetura
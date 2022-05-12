# aws-secret-manager-ansible-vault
## Description
In this article, we'll go over how to encrypt an ansible yaml file that contains sensitive data like passwords using "AWS Secrets Manager" and "Ansible vault"

## Pre-Requestes
  - AWS CLI installed on your server.
  - jq installed on your server

### Now get into our task!

Suppose you've a yaml file with name "main.yml" and you need to encrypt the file using ansible vault

```sh
# ansible-vault encrypt main.yml
New Vault password:
Confirm New Vault password:
Encryption successful
```
I've encrypted the file using a password called 'test@123' and if we want to run the file the file needs to decrypt and we can use the below command 

```sh
# ansible-playbook --ask-vault-pass main.yml
```

The password we used to encrypt the file 'main.yml' with Anible Vault must be saved in a file, an environment variable, or typed in directly. What if you also want to encypt that? It will loop, and we must store the master password ie, the vault password in a secure location and retrieve them while running the ansible playbook.

Here we use AWS Secret Manager. AWS Secrets Manager is a security service to centrally manage sensitive information and  eliminate the need to hard-code that information into an application. You can work with Secrets Manager in any of the following ways :

   - AWS Management Console
   - AWS Command Line Tools
   - AWS SDKs
   - Secrets Manager HTTPS Query API

Here I'm using AWS CLI to create secret in AWS secret manager. Now we need to create a IAM Role and policy and attach the instances to have access for aws secrets manager. Ref: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html

Now create a file as secret.json with the content of our vault password. You can either store the files in a git private repo or remove the json file after creating the AWS secret.
```sh
{
  "ansible_dev_vault_passwd": "vault@123"
}
```
Run the following commands after we have assigned the IAM roles to the instance.

```sh
aws secretsmanager create-secret --name ansible/Vault_Password --description "vault password" --secret-string file://secret.json --region ap-south-1


{
    "VersionId": "d04f008c-fr45-4398-1233-3e8f94b71d0a", 
    "Name": "ansible/vaultpassword", 
    "ARN": "arn:aws:secretsmanager:ap-south-1:3702343430737204:secret:ansible/Vault_Password-38cA1C"
}
```

Now we need to create a script, let's name that as 'retrive-secrets.sh' that will fetch the password from AWS secrets manager and pass to Ansible vault for decrypting the password used in property file.

```sh
#!/bin/bash

secret=$(aws secretsmanager get-secret-value --secret-id ansible/Vault_Password --region ap-south-1 | jq -r '.SecretString' | jq -r '.ansible_vault_passwd')
echo ${secret}
```
The script needs 'jq' to be installed on the server and to install it in Amazon Linux and other CentOs servers, you can use below command:
```sh
yum install jq -y
```

Now let's run our play-book using below command.

```sh
ansible-playbook secrets.yml --vault-password-file ./retrive-secrets.sh
```

![WhatsApp Image 2022-04-27 at 5 43 15 AM](https://user-images.githubusercontent.com/65948438/168184583-92df6f45-b638-4464-a22b-fd29ffb5679c.jpeg)

## Conclusion
We talked about using "ansible-vault" to encrypt ansible files with sensitive information and keeping the vault password in AWS secretsmanager. One of the main advantages of the above exercise is no AWS access key and Secrets are maintained in the host so. it is more secure that way. Thank you for your time !

# ssh-remote-server-setup
https://roadmap.sh/projects/ssh-remote-server-setup

The goal of this project is to learn and practice the basics of Linux. You are required to setup a remote linux server and configure it to allow SSH connections.

## Requirements
You are required to setup a remote linux server and configure it to allow SSH connections.

- Register and setup a remote linux server on any provider e.g. a simple droplet on DigitalOcean which gives you $200 in free credits with the link. Alternatively, use AWS or any other provider.
- Create two new SSH key pairs and add them to your server.
- You should be able to connect to your server using both SSH keys.
- You should be able to use the following command to connect to your server using both SSH keys.

```bash
ssh -i <path-to-private-key> user@server-ip
```

Also, look into setting up the configuration in ~/.ssh/config to allow you to connect to your server using the following command.

```bash
ssh <alias>
```

The only outcome of this project is that you should be able to SSH into your server using both SSH keys. Future projects will cover other aspects of server setup and configuration.

Stretch goal: install and configure fail2ban to prevent brute force attacks.

## Solution

1. Deploy new Linux Server (RHEL8) to Azure:

```bash
SUB_ID='<sub id>'
RG='dantcsetest'
VM_NAME='dantrhel81'
ADMIN_USERNAME='azureuser'
SECURE_PASSWORD='<secure pw>'
LOCATION='swedencentral'

az login
az account set -s $SUB_ID

az vm create \
  --resource-group $RG \
  --name $VM_NAME\
  --image RedHat:RHEL:8-lvm-gen2:latest \
  --admin-username $ADMIN_USERNAME \
  --admin-password $SECURE_PASSWORD \
  --authentication-type password \
  --size Standard_B2s \
  --location $LOCATION
```
2. Generate two new key pairs:
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_first
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_second
```

3. Copy the two new public keys to the VM:
```bash
PUB_KEY=$(cat ~/.ssh/id_rsa_first.pub)

az vm run-command invoke  \
  --resource-group $RG \
  --name $VM_NAME \
  --command-id RunShellScript \
  --scripts "echo '$PUB_KEY' >> /home/azureuser/.ssh/authorized_keys"

ssh azureuser@135.225.75.211 -i ../.ssh/id_rsa_first

<Repeat for key 2...>
```
4. Test access to the VM:
```bash
ssh azureuser@<IP> -i ../.ssh/id_rsa_second
```

### Stetch Goal:

Add the following config to ~/.ssh/config on the client machine:

```bash
Host rhel81
  HostName <IP>           # IP or domain of the server
  User azureuser                     # The SSH user
  IdentityFile ~/.ssh/id_rsa_second  # Path to the private key (optional, if you use one)
  Port 22
```

Test connection using alias:
```bash
 ssh rhel81
```

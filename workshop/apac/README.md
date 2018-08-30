# Instructions to bring up the topology for workshop

1. Login to server (Server details are given in google doc)

2. Go to vagrant directory

```bash
cd cfm-vagrant/cfm-1x1-vqfx-8srv-mcloud/
```

3. Login to vagrant VM

```bash
vagrant ssh s-srv1
sudo -i
```

4. Start ssh-agent for your session

```bash
eval `ssh-agent -s`
```

5. Go to contrail-multi-cloud directory

```bash
cd contrail-multi-cloud/
```

6. Run deployer.sh container script to bring up deployer container

```bash
./deployer.sh -r hub.juniper.net/contrail -t 5.0.1-0.214 -v $PWD:/root/multicloud
```

**Below is a sample output which you will see after running the above command**

```bash
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Using default tag: latest
latest: Pulling from sanjuabraham/contrail-multicloud-deployer
Digest: sha256:b59b654986bd643767558c0fd83b8fab0fafc2b3f76c0754c90f14de0f6aafc9
Status: Image is up to date for sanjuabraham/contrail-multicloud-deployer:latest

 contrail-multicloud-deployer docker is already running.

 The generated ssh key can be found at - keys/contrail-multicloud-key-24728

 Please ensure ssh key is added to ssh-agent. Check it using command: ssh-add -l


 Use "ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no -A root@127.0.0.1 -p 2222" to log into docker

 Default password is - multicloud

 run ./deploy.sh from one-click-deployer inside docker
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
```

7. Copy ssh command from output of above command and run it. Default password is **multicloud**

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no -A root@127.0.0.1 -p 2222
```

8. Every time you login to container, you need to make sure that ssh-agent is up and running

**Output** of command, if ssh-agent is up and running and key has been added to to agent
```bash
ae50ee0906ca:~# ssh-add -l
2048 SHA256:jHDjgpWMyZwcy3mJU3Q1tZVJcKwy3eWdi6IMm7UOk5s keys/contrail-multicloud-key-22559 (RSA)
```
**With the above output you don't need to run any command below. Go to step#9**

**Output** of command, if ssh-agent is not running
```bash
b8e8795b7903:~# ssh-add -l
Could not open a connection to your authentication agent.
```
If you are seeing above output, then your ssh-agent is not running. So run below commands to bring up ssh-agent and to add keys
**Note** Every system will have have a different key prefix. In the below example we are using 22559
```bash
eval `ssh-agent -s`
ssh-add multicloud/keys/contrail-multicloud-key-22559
```

9. Once logged in to container, change directory to multicloud/one-click-deployer

```bash
cd multicloud/one-click-deployer/
```

10. Login to azure account

```bash
az login
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code BG22JY7X9 to authenticate.
```

11. Login to URL given above and enter the code given in the above output command. After successful login, you will see json output of your subscriptions (Something like below)

```bash
27e376826e01:~/multicloud# az login
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code BG22JY7X9 to authenticate.
[
  {
    "cloudName": "AzureCloud",
    "id": "0d8225a3-adc6-4ed0-8372-885da35c523f",
    "isDefault": true,
    "name": "Free Trial",
    "state": "Enabled",
    "tenantId": "bea78b3c-4cdb-4130-854a-1d193232e5f4",
    "user": {
      "name": "mnayakbomman@juniper.net",
      "type": "user"
    }
  },
  {
    "cloudName": "AzureCloud",
    "id": "985d432a-f151-4b24-b18f-996b58329fc8",
    "isDefault": false,
    "name": "Pay-As-You-Go",
    "state": "Enabled",
    "tenantId": "bea78b3c-4cdb-4130-854a-1d193232e5f4",
    "user": {
      "name": "mnayakbomman@juniper.net",
      "type": "user"
    }
  }
]
```

12. Check default subscription account, using command `az account list --output table`

```bash
27e376826e01:~/multicloud# az account list --output table
Name           CloudName    SubscriptionId                        State    IsDefault
-------------  -----------  ------------------------------------  -------  -----------
Free Trial     AzureCloud   0d8225a3-adc6-4ed0-8372-cf5da35c523f  Enabled  True
Pay-As-You-Go  AzureCloud   985d432a-f151-4b24-b18f-456b58329fc8  Enabled  False

```

13. Set Pay-As-You-Go subscription account as your default account

```bash
az account set --subscription <subscription_id>
```

14. Check how many quota limit you have on CPU’s. Make sure that you have entered the right region.
For example purposes we are using `WESTUS`

__Note: If your limit is less than 30 then, then you will not be able to create multicloud topology__

```bash
az vm list-usage --location "WestUS" --query "[?name.localizedValue == 'Standard F Family vCPUs'].{name: name, currentValue: currentValue, limit: limit}"
```
**Below is a sample output which you will see after running the above command**

```bash
[
  {
    "currentValue": "0",
    "limit": "100",
    "name": {
      "localizedValue": "Standard F Family vCPUs",
      "value": "standardFFamily"
    }
  }
]

```

15. If you have enough quota limit on other any other Region, then change the topology.yaml file to point the cloud to correct region. Edit `/root/multicloud/topology.yml`

```text
Expectable values for region are
'SouthAfricaWest', 'SouthAfricaNorth', 'UAECentral', 'UAENorth', 'SoutheastAsia',
'EastAsia', 'AustraliaEast', 'AustraliaSoutheast', 'AustraliaCentral',
'AustraliaCentral2', 'ChinaEast', 'ChinaNorth', 'ChinaEast2', 'ChinaNorth2',
'CentralIndia', 'WestIndia', 'SouthIndia', 'JapanEast', 'JapanWest', 'KoreaCentral',
'KoreaSouth', 'USSecEast', 'USSecWest', 'USGovVirginia', 'USGovIowa', 'USGovArizona',
'USGovTexas', 'USDoDEast', 'USDoDCentral', 'GermanyNorth', 'GermanyWestCentral',
'SwitzerlandNorth', 'SwitzerlandWest', 'NorwayEast', 'NorwayWest', 'NorthEurope',
'WestEurope', 'FranceCentral', 'FranceSouth', 'UKWest', 'UKSouth', 'GermanyCentral',
'GermanyNortheast', 'EastUS', 'EastUS2', 'CentralUS', 'NorthCentralUS',
'SouthCentralUS', 'WestCentralUS', 'WestUS', 'WestUS2', 'CanadaEast', 'CanadaCentral',
'BrazilSouth'
```

16. Make sure that you have resource group name `workshop`

```bash
az group exists --name workshop
```

17. [Skip this step if workshop resource group already exists]
Create `workshop` resource group. Replace `WESTUS` with your region which has the required quota for this workshop

```bash
az group create --name workshop --location WESTUS
```

18. It always safe to run `az login` once again, to make sure that your azure authentication token is not expired. It expires every one hour

19. Copy keys to all nodes in onprem cloud

```bash
# password for all servers: c0ntrail123
ssh-copy-id 192.168.2.13
ssh-copy-id 192.168.2.11
ssh-copy-id 192.168.2.14
ssh-copy-id 192.168.2.15
```

20. Run deploy.sh script

```bash
./deploy.sh
```

# [Exercises for contrail-multi-cloud](../exercise/README.md)

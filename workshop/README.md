# Instructions to bring up the topology for workshop

1. Login to the instance with internet connectivity

2. Export your juniper username

```bash
export USERNAME="abcdef"
```

3. Change directory to ${USERNAME}/contrail-multi-cloud

```bash
cd ${USERNAME}/contrail-multi-cloud
```

4. Start ssh-agent for your session

```bash
eval `ssh-agent -s`
```

5. Run deployer.sh container script to bring up deployer container

```bash
./deployer.sh -r sanjuabraham -v $PWD:/root/multicloud -n ${USERNAME}

Using default tag: latest
latest: Pulling from sanjuabraham/contrail-multicloud-deployer
Digest: sha256:b59b654986bd643767558c0fd83b8fab0fafc2b3f76c0754c90f14de0f6aafc9
Status: Image is up to date for sanjuabraham/contrail-multicloud-deployer:latest

# This will give the user a pre-built image with all the

#
 contrail-multicloud-deployer docker is already running.

 The generated ssh key can be found at - keys/contrail-multicloud-key-24728

 Please ensure ssh key is added to ssh-agent. Check it using command: ssh-add -l


 Use "**ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no -A root@127.0.0.1 -p 2222**" to log into docker

 Default password is - multicloud

 run ./deploy.sh from one-click-deployer inside docker

```

6. Copy ssh command from output of above command and run it. Default password is **multicloud**

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no -A root@127.0.0.1 -p <port-no>
```

7. Once logged in to container, change directory to multicloud/one-click-deployer

```
bash
cd multicloud/one-click-deployer/
```

8. Login to azure account

```bash
az login
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code BG22JY7X9 to authenticate.
```

9. Login to URL given above and enter the code given in the above output command. After successful login, you will see json output of your subscriptions (Something like below)

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

10. Check default subscription account, using command `az account list --output table`

```bash
27e376826e01:~/multicloud# az account list --output table
Name           CloudName    SubscriptionId                        State    IsDefault
-------------  -----------  ------------------------------------  -------  -----------
Free Trial     AzureCloud   0d8225a3-adc6-4ed0-8372-cf5da35c523f  Enabled  True
Pay-As-You-Go  AzureCloud   985d432a-f151-4b24-b18f-456b58329fc8  Enabled  False

```

11. Set Pay-As-You-Go subscription account as your default account

```bash
az account set --subscription <subscription_id>
```

12. Check how many quota limit you have on CPU’s.
__Note: If your limit is less than 100 then, then you will not be able to create multicloud topology__

```bash
az vm list-usage --location "West US" --query "[?name.localizedValue == 'Standard F Family vCPUs'].{name: name, currentValue: currentValue, limit: limit}"

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

13. If you have enough quota limit on other any other Region, then change the topology.yaml file to point the cloud to correct region. Edit `../topology.yaml`

```yaml
- provider: azure
  organization: Juniper
  project: contrail-multicloud
  regions:
    - name: WestUS
      resource_group: contrail-multicloud-training
      clouds:

# Expectable values are
#'SouthAfricaWest', 'SouthAfricaNorth', 'UAECentral', 'UAENorth', 'SoutheastAsia',
#'EastAsia', 'AustraliaEast', 'AustraliaSoutheast', 'AustraliaCentral',
#'AustraliaCentral2', 'ChinaEast', 'ChinaNorth', 'ChinaEast2', 'ChinaNorth2',
#'CentralIndia', 'WestIndia', 'SouthIndia', 'JapanEast', 'JapanWest', 'KoreaCentral',
#'KoreaSouth', 'USSecEast', 'USSecWest', 'USGovVirginia', 'USGovIowa', 'USGovArizona',
#'USGovTexas', 'USDoDEast', 'USDoDCentral', 'GermanyNorth', 'GermanyWestCentral',
#'SwitzerlandNorth', 'SwitzerlandWest', 'NorwayEast', 'NorwayWest', 'NorthEurope',
#'WestEurope', 'FranceCentral', 'FranceSouth', 'UKWest', 'UKSouth', 'GermanyCentral',
#'GermanyNortheast', 'EastUS', 'EastUS2', 'CentralUS', 'NorthCentralUS',
#'SouthCentralUS', 'WestCentralUS', 'WestUS', 'WestUS2', 'CanadaEast', 'CanadaCentral',
#'BrazilSouth'
```

14. Make sure that you have resource group name `multicloud`

```bash
az group exists --name multicloud
```

15. [Skip this step if multicloud resource group already exists]
Create `multicloud` resource group. Replace `WESTUS` with your region which has the required quota for this workshop

```bash
az group create --name multicloud --location WESTUS
```

16. It always safe to run `az login` once again, to make sure that your azure authentication token is not expired. It expires every one hour

17. Run deploy.sh script

```bash
./deploy.sh
```

# [Exercises for contrail-multi-cloud](../exercise/README.md)

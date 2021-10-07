# Azure-Cloud-fundamentals
Start at the basics using Azure CLI 

## prerequisite
After [connect to subscription](https://www.microsoftazurepass.com) and enter the promo code,
Download the [azure CLI](https://docs.microsoft.com/he-il/cli/azure/install-azure-cli-windows?tabs=azure-cli#azure-cli-current-version) and install.

## Login
Open a terminal and type
```
az login
```
the output will be prompt you to login and it will print to the screen all of the subscription you have on your account

## Create a Resource Group

`name` Resource group name
For choosing the region you need to keep in mind to factors: latency and pricing 
Chekout what is the [pricing](https://azure.microsoft.com/en-us/pricing/details/virtual-machines/linux/) for a specific location,
and what is the [latency](http://azurespeedtest.azurewebsites.net/) from your braweser to all regions. 
I choose `switzerlandnorth` location for good latency and good pricing.

To see the name of the reference for your location you can excute the command `az account list-locations -o table` and search for your location.
You can use `-l` or `--location` and `-n` or `--name` fro the command
The `--tags` is key value pair for more metadata 
```
az group create -n webAppDemoRG -l switzerlandnorth --tags Owner=Matan-TAl purpose=exercise 
```
The output will be in jason format.
```jason
 "id": "/subscriptions/YOUR_KEY",
  "location": "switzerlandnorth",
  "managedBy": null,
  "name": "webAppDemoRG",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": {
    "Owner": "Matan-TAl",
    "purpose": "exercise"
  },
  "type": "Microsoft.Resources/resourceGroups"
}
```

## Create Virtual Network

keep in mind the IP addresses and the range. we choose 65531 addresses by choosing subnet mask of 16
```
az network vnet create --name vnet --resource-group webAppDemoRG --location switzerlandnorth --address-prefixes 10.0.0.0/16
```
explaination
`--name` for the vertual network name for your chouce
`--resource-goup` or `-g` attach to, in our case the one we just created webAppDemoRG
`--location` switzerlandnorth
`--address-prefixes 10.0.0.0/16` is the amount of addresses you will have in the VNet.

output will be
```
{
  "newVNet": {
    "addressSpace": {
      "addressPrefixes": [
        "10.0.0.0/16"
      ]
    },
    "bgpCommunities": null,
    "ddosProtectionPlan": null,
    "dhcpOptions": {
      "dnsServers": []
    },
    "enableDdosProtection": false,
    "enableVmProtection": null,
    "etag": "W/\"1041f685-af42-45ea-8526-30893cf93a4c\"",
    "extendedLocation": null,
    "flowTimeoutInMinutes": null,
    "id": "/subscriptions/YOUR_SUB_KEY",
    "ipAllocations": null,
    "location": "switzerlandnorth",
    "name": "vnet",
    "provisioningState": "Succeeded",
    "resourceGroup": "webAppDemoRG",
    "resourceGuid": "238227c5-5ba3-4ce7-bbe8-d34825d6d43c",
    "subnets": [],
    "tags": {},
    "type": "Microsoft.Network/virtualNetworks",
    "virtualNetworkPeerings": []
  }
}
```

## Create Subnets

Create the subnet and attached them to the resource group we created and our virtual network 
I choose subnet mask of 24, and i have 251 addresses in each  subnet.

`-g` - resource group name
`--vnet-name` VNet
`-n` name of the subnet
`--address-prefixes` how many IP addresses will be for this subnet.
1. Public subnet
```
az network vnet subnet create --vnet-name VNet -g webAppDemoRG -n public --address-prefixes 10.0.1.0/24
```

2. Private subnet
```
az network vnet subnet create --vnet-name VNet -g webAppDemoRG -n private --address-prefixes 10.0.2.0/24
```
After we excute those comand we can list all of the subnet in our VNet and with `-o table` in table formt 
```
az network vnet subnet list -g webAppDemoRG --vnet-name VNet -o table
```


## Create NSG to DataBase and WebApp

This is created with a default ruls
```
az network nsg create -g webAppDemoRG -n publicWebTierNSG -l switzerlandnorth --tags Owner=Matan-TAl purpose=exercise
```

```
az network nsg create -g webAppDemoRG -n privateDataTierNSG -l switzerlandnorth --tags Owner=Matan-TAl purpose=exercise
```

## attach the subnet to the NSG

1. private subnet

We will attach the premake NSG dataTierNSG to the private subnet
```
az network vnet subnet update --resource-group webAppDemoRG --vnet-name vnet -n private --network-security-group privateDataTierNSG
```

2. public subnet

We will attach the premake NSG webTierNSG to the public subnet
```
az network vnet subnet update --resource-group webAppDemoRG --vnet-name vnet -n public --network-security-group publicWebTierNSG
```



## adding rule to the NSG

This is how to config rules
```
az network nsg rule create --name
                           --nsg-name
                           --priority
                           --resource-group
                           [--access {Allow, Deny}]
                           [--description]
                           [--destination-address-prefixes]
                           [--destination-asgs]
                           [--destination-port-ranges]
                           [--direction {Inbound, Outbound}]
                           [--protocol {*, Ah, Esp, Icmp, Tcp, Udp}]
                           [--source-address-prefixes]
                           [--source-asgs]
                           [--source-port-ranges]
                           [--subscription]
```


#### Allow from specific IP address ranges on 22.
```
az network nsg rule create -g webAppDemoRG --nsg-name publicWebTierNSG -n SSH_Allow --priority 100  --source-port-ranges '*' --destination-address-prefixes 10.0.0.0/16 --destination-port-ranges 22   --access Allow --protocol Tcp --direction Inbound --description "Allow from specific IP address ranges on 22." --source-address-prefixes YOUR_IP_ADDRESS/SUBNET_MASk
```

#### Allow from Any IP address ranges on 80
```
az network nsg rule create -g webAppDemoRG --nsg-name publicWebTierNSG  -n HTTP_Allow --priority 300 --source-address-prefixes '*'  --source-port-ranges '*' --destination-address-prefixes 10.0.0.0/16 --destination-port-ranges 80  --access Allow --protocol Tcp --direction Inbound --description "Allow from Any IP address ranges on 80"
```

#### Allow from Any IP address ranges on 8080
```
az network nsg rule create -g webAppDemoRG --nsg-name publicWebTierNSG  -n Port_8080_Allow --priority 200 --source-address-prefixes '*'  --source-port-ranges '*' --destination-address-prefixes 10.0.0.0/16 --destination-port-ranges 8080  --access Allow --protocol Tcp --direction Inbound --description "Allow from Any IP address ranges on 8080"
```

#### Allow from Any IP address ranges on 443
```
az network nsg rule create -g webAppDemoRG --nsg-name publicWebTierNSG -n HTTPS_Allow --priority 400 --source-address-prefixes '*'  --source-port-ranges '*' --destination-address-prefixes 10.0.0.0/16 --destination-port-ranges 443  --access Allow --protocol Tcp --direction Inbound --description "Allow from Any IP address ranges on 443."

```
### List all rules in publicWebTierNSG

```
az network nsg rule list --nsg-name publicWebTierNSG --resource-group webAppDemoRG -o table
```

## add rules to the private NSG

#### Allow from specific IP address ranges on 22.

in our case we eant to give access from the web server to the database server only.

```
az network nsg rule create -g webAppDemoRG --nsg-name privateDataTierNSG -n SSH_Allow --priority 100 --source-address-prefixes 10.0.1.0/24 --source-port-ranges '*' --destination-address-prefixes 10.0.2.0/24 --destination-port-ranges 22   --access Allow --protocol Tcp --direction Inbound --description "Allow from specific IP address ranges on 22."
```

#### Allow from specific IP address ranges on 5432 - postgresSQL.

```
az network nsg rule create -g webAppDemoRG --nsg-name privateDataTierNSG -n postgresSQL_Allow --priority 300 --source-address-prefixes 10.0.1.0/24 --source-port-ranges '*' --destination-address-prefixes 10.0.2.0/24 --destination-port-ranges 5432   --access Allow --protocol Tcp --direction Inbound --description "Allow from specific IP address ranges on 5432."
```

### List all rules in privateDataTierNSG

```
az network nsg rule list --nsg-name privateDataTierNSG --resource-group webAppDemoRG -o table
```

## Create the SSH private and public key for the VM's

```
ssh-keygen -m PEM -t rsa -b 2048
```
1. When you're prompted to "Enter a file in which to save the key," press Enter. This accepts the default file location.
`> Enter a file in which to save the key (/c/Users/you/.ssh/id_algorithm):[Press enter]`
2. At the prompt, type a secure passphrase.
`> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]`

Now print the result to the screen using `cat`
```
cat PATH_OF_YOUR_SSH_FILE/id_rsa
```
And the public key
```
cat PATH_OF_YOUR_SSH_FILE/id_rsa.pub
```

## Create the Virtual Machine

before you execute the command you need to know the following attributes
You can list all the  images, it will take time, so you can narow it to a spesifc distrebution. in my case i want Ubuntu 20.04 LTS

#### list the images for Ubunto 20.04 LTS
```
az vm image list -f Ubuntu  --publisher Canonical --sku 20_04-lts-gen2 --all
```
## For the public subnet

```
az vm create -n WebAppServer -g webAppDemoRG --vnet-name VNet  --image Canonical:0001-com-ubuntu-server-focal:20_04-lts:latest --location switzerlandnorth --subnet public --public-ip-address-allocation dynamic  --admin-username matan-tal --storage-sku Standard_LRS --os-disk-name wevServer-VN-Disk --os-disk-size-gb 40 --size Standard_B2s --ssh-key-value PATH_TO_YOUR_ID_RSA.pub 
```

## For the private subnet

In here we need to define `--public-ip-address '""'` so no one can access from outside, and define a private ip address for connection inside the VNet.
It's the same OS.
```
az vm create -n DataServer -g webAppDemoRG --vnet-name VNet --nsg-name '""'  --image Canonical:0001-com-ubuntu-server-focal:20_04-lts:latest --location switzerlandnorth --subnet private --public-ip-address '""'  private-ip-address 10.0.2.10 --admin-username MatanTal --storage-sku Standard_LRS --os-disk-name wevServer-VN-Disk --os-disk-size-gb 40 --size Standard_B2s --ssh-key-value PATH_TO_YOUR_ID_RSA.pub 
```

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
```
az group create -n webAppDemoRG -l switzerlandnorth
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
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
```

## Create Virtual Network
```
az network vnet create --name VNet--resource-group webAppDemoRG --location switzerlandnorth
```
explain
`--name` for the vertual network name for your chouce
`--resource-goup` or `-g` attach to, in our case the one we just created webAppDemoRG
`--location` switzerlandnorth

## Create Subnets

Create the subnet and attached them to the resource group we created and our virtual network 
`-g` - resource group name
`--vnet-name` VNet
`-n` name of the subnet
`--address-prefixes` how many IP addresses will be for this subnet.
1. Public subnet
```
az network vnet subnet create --vnet-name VNet -g webAppDemoRG -n public --address-prefixes 10.0.0.0/29
```

2. Private subnet
```
az network vnet subnet create --vnet-name VNet -g webAppDemoRG -n private --address-prefixes 10.0.1.0/29
```
After we excute those comand we can list all of the subnet in our VNet and with `-o table` in table formt 
```
az network vnet subnet list -g webAppDemoRG --vnet-name VNet -o table
```


## Create SNG to Data and Web

```
az network nsg create -g webAppDemoRG -n webTierNSG -l switzerlandnorth 
```

```
az network nsg create -g webAppDemoRG -n dataTierNSG -l switzerlandnorth 
```

## add rule to the NSG


heighest priority `--priority 100`
```
az network nsg rule create --name allow access from web only  -g webAppDemoRG --nsg-name dataTierNSG --priority 100 
```




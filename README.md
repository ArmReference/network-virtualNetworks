# virtualNetworks
Reference deployment
```
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "StorageAccountName": {
      "type": "string",
      "defaultValue": ""
    },
    "ContainerName": {
      "type": "string",
      "defaultValue": ""
    },
    "SasToken": {
      "type": "string",
      "defaultValue": ""
    }
  },
  "variables": {
    "VnetName": "VNET",
    "AddressSpace": "172.16.192.0/22",
    "DnsServers": [
      "172.16.192.1",
      "172.16.192.2"
    ],
    "Provider": "/Microsoft.Network",
    "Resource": "/virtualNetworks",
    "templateUri": "[concat('https://',parameters('StorageAccountName'),'.blob.core.windows.net/',parameters('ContainerName'),variables('Provider'),variables('Resource'))]"
  },
  "resources": [
    {
      "name": "BuildVirtualNetworksSubnets-DMZ-001",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('templateUri'), '/subnet.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "Name": {
            "value": "[concat(variables('VnetName'),'-DMZ-001')]"
          },
          "AddressPrefix": {
            "value": "172.16.192.0/24"
          },
          "NetworkSecurityGroupId": {
            "value": ""
          },
          "RouteTableId": {
            "value": ""
          },
          "ServiceEndpointId": {
            "value": [ ]
          }
        }
      }
    },
    {
      "name": "BuildVirtualNetworksSubnets-APP-001",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('templateUri'), '/subnet.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "Name": {
            "value": "[concat(variables('VnetName'),'-APP-001')]"
          },
          "AddressPrefix": {
            "value": "172.16.193.0/24"
          },
          "NetworkSecurityGroupId": {
            "value": ""
          },
          "RouteTableId": {
            "value": ""
          },
          "ServiceEndpointId": {
            "value": [ ]
          }
        }
      }
    },
    {
      "name": "BuildVirtualNetworksDhcpOptions-DNS-001",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('templateUri'), '/dhcpOptions.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "DnsServers": {
            "value": "[variables('DnsServers')[0]]"
          }
        }
      }
    },
    {
      "name": "BuildVirtualNetworksDhcpOptions-DNS-002",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('templateUri'), '/dhcpOptions.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "DnsServers": {
            "value": "[variables('DnsServers')[1]]"
          }
        }
      }
    },
    {
      "name": "BuildVirtualNetworksAddressSpace-ADDR-001",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [ ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('templateUri'), '/addressSpace.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "AddressPrefixes": {
            "value": "[variables('AddressSpace')]"
          }
        }
      }
    },
    {
      "name": "BuildVirtualNetworks-VNET-001",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2016-09-01",
      "dependsOn": [
        "[concat('Microsoft.Resources/deployments/','BuildVirtualNetworksAddressSpace-ADDR-001')]",
        "[concat('Microsoft.Resources/deployments/','BuildVirtualNetworksDhcpOptions-DNS-001')]",
        "[concat('Microsoft.Resources/deployments/','BuildVirtualNetworksDhcpOptions-DNS-002')]",
        "[concat('Microsoft.Resources/deployments/','BuildVirtualNetworksSubnets-DMZ-001')]",
        "[concat('Microsoft.Resources/deployments/','BuildVirtualNetworksSubnets-APP-001')]"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(variables('templateUri'), '/virtualNetworks.json', parameters('SasToken'))]",
          "contentVersion": "2017.09.01.0"
        },
        "parameters": {
          "Name": {
            "value": "[concat(variables('VnetName'),'-001')]"
          },
          "addressSpace": {
            "value": "[createArray(reference('BuildVirtualNetworksAddressSpace-ADDR-001').outputs.addressPrefixes.value)]"
          },
          "dhcpOptions": {
            "value": "[createArray(reference('BuildVirtualNetworksDhcpOptions-DNS-001').outputs.dnsServers.value,reference('BuildVirtualNetworksDhcpOptions-DNS-002').outputs.dnsServers.value)]"
          },
          "enableDdosProtection": {
            "value": false
          },
          "enableVmProtection": {
            "value": false
          },
          "subnets": {
            "value": "[createArray(reference('BuildVirtualNetworksSubnets-DMZ-001').outputs.subnet.value,reference('BuildVirtualNetworksSubnets-APP-001').outputs.subnet.value)]"
          },
          "virtualNetworkPeerings": {
            "value": [ ]
          }
        }
      }
    }

  ],
  "outputs": {
    "virtualNetworks": {
      "type": "object",
      "value": "[reference('BuildVirtualNetworks-VNET-001').outputs.virtualNetworks.value]"
    }
  }
}
```

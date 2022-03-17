// Set up parameters

@minLength(11)
@maxLength(12)
param VMName string

param AdminUser string
@secure()
param AdminPW string

param VMLocation string = resourceGroup().location
param VMIPAddress string
param VMInstanceSize string

// Get the location of the Resource Group to determine vNet for deployment

var VMvNet = (VMLocation == 'uksouth') ? 'AZ01' : 'AZ02'

@allowed([
  'DirectoryServices'
  'FileServices'
  'Management'
  'DatabaseServices'
])
param VMSubnet string

@allowed([
  '2016-Datacenter'
  '2019-Datacenter'
  '2022-datacenter-azure-edition'
])
param OSVersion string

@allowed([
  '128'
  '256'
  '512'
  '1024'
  '2048'
  '4096'
  '8192'
])
param DataDiskSize string

@allowed([
  'Standard_LRS'
  'Premium_LRS'
  'StandardSSD_LRS'
])
param DiskSku string

param CostCentreTag string
param FunctionTag string

// Get the ID of the subnet in the required vNet

resource SubnetID 'Microsoft.Network/virtualNetworks/subnets@2021-05-01' existing = {
  name: '${VMvNet}/${VMSubnet}'
  scope: resourceGroup('TM-RG-Networking')
}

// Create the NIC

resource NewNic 'Microsoft.Network/networkInterfaces@2021-05-01' = {
  name: '${VMName}_NIC01'
  location: VMLocation
  properties: {
    ipConfigurations:[
      {
        name: 'ipconfig1'
        properties:{
          privateIPAllocationMethod: 'Static'
          privateIPAddress: VMIPAddress
          subnet:{
            id: SubnetID.id
          }
          primary: true
        }
      }
    ]
  }
}

// Create the VM

resource NewVM 'Microsoft.Compute/virtualMachines@2021-11-01' = {
  name: VMName
  location: VMLocation
  tags: {
    CostCentre: CostCentreTag
    function: FunctionTag
  }
  properties: {
    hardwareProfile: {
      vmSize: VMInstanceSize
    }
    osProfile:{
      computerName: VMName
      adminUsername: AdminUser
      adminPassword: AdminPW
    }
    storageProfile: {
      imageReference: {
        publisher: 'MicrosoftWindowsServer'
        offer: 'WindowsServer'
        sku: OSVersion
        version: 'latest'
      }
      osDisk: {
        createOption: 'FromImage'
        managedDisk: {
          storageAccountType: DiskSku
        }
      }
      dataDisks:[
        {
          name:'${VMName}_Data_01'
          diskSizeGB: int(DataDiskSize)
          lun: 0
          createOption: 'Empty'
        }
      ]    
  }
  networkProfile:{
    networkInterfaces:[
      {
        id:NewNic.id
      }
    ]
  }
}
}

# VMSS Failed Status via Application Health Extension Demo 

**Goal**: The purpose of this demo is to understand how Application Health Extension displays health statuses of instances and understand how the health extension pings an application on the request path to assess health of th application. 

1. Deploy a new VMSS with the application health extension using a custom ARM template. Please use the below 'Deploy to GitHub' Button and enter the required parameters and deploy: 

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fkmccullagh19%2FLabBoxProjects%2Fmain%2FHealthExtensionDemo)

2. Once you have successfully deployed the VMSS, you should see that the instances Health Status on the overview page in the portal as "Unhealthy". This is because there is no application at the root for the health extension to ping. To resolve this, please use the [custom script extension to install IIS.htm](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/tutorial-install-apps-powershell) at the root and then update the request path to look there. Once CSE is installed and IIS.htm is running at the root of all instances and the health application request path is configured, the instances should report a 200 response for Healthy. 

From the article, 

## Install the extension 

```powershell
#### Get information about the scale set
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

##### Add the Custom Script Extension to install IIS and configure basic website
$vmss = Add-AzVmssExtension `
  -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.9 `
  -Setting $customConfig

#### Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
``` 
```powershell
  ## Allow communicaion 
 #### Create a rule to allow traffic over port 80
$nsgFrontendRule = New-AzNetworkSecurityRuleConfig `
  -Name myFrontendNSGRule `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 200 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access Allow

#### Create a network security group and associate it with the rule
$nsgFrontend = New-AzNetworkSecurityGroup `
  -ResourceGroupName  "myResourceGroup" `
  -Location EastUS `
  -Name myFrontendNSG `
  -SecurityRules $nsgFrontendRule

$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName  "myResourceGroup" `
  -Name myVnet

$frontendSubnet = $vnet.Subnets[0]

$frontendSubnetConfig = Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name mySubnet `
  -AddressPrefix $frontendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgFrontend

Set-AzVirtualNetwork -VirtualNetwork $vnet
  ```
  
  3. Confirm that the Instances are now showing in a healthy State. 

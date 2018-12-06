

# Azure VM Encryption Lab
## Create Windows Server 2016 VM 

0. Run the following template.  https://portal.azure.com/microsoft.onmicrosoft.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdmitriilezine%2FSingle-VM%2Fmaster%2FSingle%20VM%2Fazuredeploy.json
1. Create new Resource Group, name it 'alias-ADELAB', where 'alias' is your alias
2. For location select "West US"
3. Admin password is created as "Subscription#YOURSUBSCRIPTIONID". You can replace it with your own strong password
4. Replace Source Client IP with Public IP of your device, or put "*" to allow any source
5. Under "Windows OS Type" parameter select WinServer2016
6. Virtual Machine Size Standard_DS12_v2 to have speedy lab experienses 
7. Click "Purchase"
8. VM will be deployed in about 5 minutes. This VM is deployed with autoshutdown scheduled for 11PM EST.

## Verify VM encryption status
Open Cloud Shell and run the following PowerShell code to see current status of VM encryption. Replace 'alias' with your alias
	
	$rgName = 'alias-ADELAB'
	$vmName = 'JumpBox'
	Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName $rgName -VMName $vmName

* What is the status of VM encryption?

## Create Azure Key Vault
In the Cloud Shell run the following code. Replace 'alias' with your alias. Replace 'domain' with domain that will complete your account for Azure AD, as shown in Azure portal in the top right corner.
Make sure that the location is the same as was used to create VM, ie West US

	$alias = 'alias'
	$logonname = $alias+'@domain.com'
	$location ="West US"
	$keyVaultName =$alias+'-akvade'

Create AKV

	New-AzureRmKeyVault -VaultName $keyVaultName -ResourceGroupName $rgName -Sku Standard -Location $location

Enable it for ADE

	Set-AzureRmKeyVaultAccessPolicy -VaultName $keyVaultName -EnabledForDiskEncryption
 

## Encrypt VM
After AKV is ready, we can encrypt the VM. Verify parameters for accuracy. 
In the Cloud Shell run the following code.

	$rgName = 'alias-ADELAB'
	$vmName = 'JumpBox'
	$KeyVaultName = $alias+'-akvade'
	$KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname
	$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri
	$KeyVaultResourceId = $KeyVault.ResourceId

Enable disk encryption on a VM

	Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType All -Force


## Verify VM encryption status
In the Cloud Shell run the following

	Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName $rgName -VMName $vmName

* Did it enable encryption on both drives? 
* Why do you think the data drive is not encrypted?


## Intialize and format data disk via Run Command

0. In Azure Portal, click on the VM, scroll down in the left pane and click on "Run Command"
1. Click on the RunPowerShellScript
2. Copy/paste the following PowerShell code into the window and click Run.
3. Wait for execution of the commands against the VM.
###
	Initialize-Disk -Number 3 -PartitionStyle MBR
	New-Partition -DiskNumber 3 -UseMaximumSize -DriveLetter G
	Format-Volume -DriveLetter G -FileSystem NTFS

#### At this time, the data disk should be initialized and formatted on the VM and we should be able to encrypt it.
#### Run the disk encryption command again and see if it will encrypt the second disk

## Check AKV for secret material related to ADE

1. In AKV, see if you have access to the Secrets. Why not?
2. Grant yourself access to the Secrets. Why is it possible for you to do this?
3. How many secrets are present?

# Disable encryption on VM
In the Cloud Shell run the following

	Disable-AzureRmVMDiskEncryption -ResourceGroupName $rgNam -VMName $vmName


## Verify VM encryption status
In the Cloud Shell run the following

	Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName $rgName -VMName $vmName


# Enable encryption with KEK
In the Cloud Shell run the following

	$keyEncryptionKeyName = 'MySuperSecretKeyEncryptionKey'
    Add-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName -Destination 'Software'
    $keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid
	Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType All -Force

## Verify VM encryption status
In the Cloud Shell run the following

	Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName $rgName -VMName $vmName

## Check AKV for secret material related to ADE

1. In AKV, see if you have access to Keys. Why not?
2. Grant yourself access to the Secrets. Why is it possible for you to do this?
3. How many Keys are present?

# References
Azure Disk Encryption https://docs.microsoft.com/en-us/azure/security/azure-security-disk-encryption-prerequisites



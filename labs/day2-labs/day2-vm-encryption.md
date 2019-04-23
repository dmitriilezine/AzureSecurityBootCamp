

# Azure VM Encryption Lab
## Create Windows Server 2016 VM 

For this lab you need to have ADDS lab deployment from Day 1. 
It takes about one hour to deploy the ADDS and should have been done before start of this lab.

VMs in the lab are automatically shutdown every evening. Before starting this lab, power them up.

0. In Azure Portal select resource group to which it was delopyed (**'alias-ADELAB'**)
1. Select DC1, DC2, AS1 and JumpBox (JumpBox only if you plan to get OS level access into environment), and click on Start, confirm Yes if asked.
2. Wait till VM status shown as Running.

## Verify VM encryption status
Open Cloud Shell and run the following PowerShell code to see current status of VM encryption. Replace 'alias' with your alias
	
	$rgName = 'alias-ADELAB'
	$vmName1 = 'DC1'
	$vmName2 = 'DC2'
	Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName $rgName -VMName $vmName1
	Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName $rgName -VMName $vmName2

* What is the status of VM encryption for DC1 and DC2?

## Create Azure Key Vault
In the Cloud Shell run the following code. Replace **'alias'** with your alias. Replace **'domain'** with domain that will complete your account for Azure AD, as shown in Azure portal in the top right corner.
Make sure that the location is the same as was used to create VMs, ie if VMs are created in **West US** then specify **West US**, or the location that you used to create those VMs. Azure Key Vault used for disk encryption must be in the same region as the VMs being encrypted.

	$alias = 'alias'
	$logonname = $alias+'@domain.com'
	$location ="West US"
	$keyVaultName =$alias+'-akvade'

Create AKV

	New-AzureRmKeyVault -VaultName $keyVaultName -ResourceGroupName $rgName -Sku Standard -Location $location

Enable it for ADE

	Set-AzureRmKeyVaultAccessPolicy -VaultName $keyVaultName -EnabledForDiskEncryption
 

## Encrypt DC1 VM
After AKV is ready, we can encrypt the VM. Important: per Azure ADE guideance, for any critical environment, make a backup of a VM before enabling ADE.
Verify parameters for accuracy. 
In the Cloud Shell run the following code.

	$rgName = 'alias-ADELAB'
	$vmName = 'DC1'
	$KeyVaultName = $alias+'-akvade'
	$KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname
	$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri
	$KeyVaultResourceId = $KeyVault.ResourceId

Enable disk encryption on a DC1 VM

	Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType All -Force


## Verify DC1 VM encryption status
In the Cloud Shell run the following

	Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName $rgName -VMName $vmName

* Did it enable encryption on both drives? 

## Encrypt DC2 VM with KEK
Grant your account access to keys in the key vault. In Cloud Shell:

	$alias = 'alias'
	$KeyVaultName = $alias+'-akvade'
	$logonname = $alias+'@domain.com'
	Set-AzureRmKeyVaultAccessPolicy -VaultName $keyVaultName -UserPrincipalName $logonname -PermissionsToKeys decrypt,encrypt,unwrapKey,wrapKey,verify,sign,get,list,update,create,import,delete,backup,restore,recover,purge -PassThru

In the Cloud Shell run the following code to create KEK. Verify parameters for accuracy

	$rgName = 'alias-ADELAB'
	$vmName = 'DC2'
	$KeyVaultName = $alias+'-akvade'
	$KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname
	$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri
	$KeyVaultResourceId = $KeyVault.ResourceId
	$keyEncryptionKeyName = 'MySuperSecretKeyEncryptionKey'
    Add-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName -Destination 'Software'
    $keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid

Enable disk encryption on a DC2 VM

	Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType All -Force


## Verify DC2 VM encryption status
In the Cloud Shell run the following

	Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName $rgName -VMName $vmName

* Did it enable encryption on both drives? 

## Check AKV for secret material related to ADE

1. In AKV, see if you have access to the Secrets or Keys. Why not?
2. Grant yourself access to the Secrets and Keys. Why is it possible for you to do this?
3. How many secrets are present?
4. How many keys are present?

## Encrypt Application Server (AS1) VM
In the Cloud Shell run the following code.

	$rgName = 'alias-ADELAB'
	$vmName = 'AS1'
	$KeyVaultName = $alias+'-akvade'
	$KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname
	$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri
	$KeyVaultResourceId = $KeyVault.ResourceId

Enable disk encryption on a AS1 VM

	Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType All -Force


## Verify AS1 VM encryption status
In the Cloud Shell run the following

	Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName $rgName -VMName $vmName

* Did it enable encryption on both drives? 
* Why do you think the data drive is not encrypted?

## Intialize and format data disk via Run Command
### Second disk on app server VM is not initialized during VM deployment and must be initialized 

0. In Azure Portal, click on the VM, scroll down in the left pane and click on "Run Command"
1. Click on the RunPowerShellScript
2. Copy/paste the following PowerShell code into the window and click Run.
3. Wait for execution of the commands against the VM.
###
	Initialize-Disk -Number 3 -PartitionStyle MBR
	New-Partition -DiskNumber 3 -UseMaximumSize -DriveLetter G
	Format-Volume -DriveLetter G -FileSystem NTFS

#### At this time, the data disk should be initialized and formatted on the VM and we should be able to encrypt it.
#### Run the disk encryption command again (in Cloud Shell) and see if it will encrypt the second disk


# Disable encryption on DC1 VM
In the Cloud Shell run the following

	$rgName = 'alias-ADELAB'
	$vmName = 'DC1'
	Disable-AzureRmVMDiskEncryption -ResourceGroupName $rgName -VMName $vmName
	Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName $rgName -VMName $vmName

Verify that both disks are not encrypted.


## Check AKV for secret material related to ADE

1. In AKV, see if you have access to the Secrets or Keys. Why not?
2. Grant yourself access to the Secrets and Keys. Why is it possible for you to do this?
3. How many secrets are present?
4. How many keys are present?

# Stretch Goal
## Extract data disks from each DC and attempt to extract NTDS.DIT
## Which disk allowed you to do this?

0. Shut down both DC VMs. In Azure Portal, select DC1 and DC2 VMs and click on Shutdown. This will deallocate the disks from a VM.
1. Next you need to export disk to a location where you can mount it. 
2. Since disks are very large size (data disks in this case are 2GB, OS disks are 127GB), you do not want copy it down from Azure to your device. Instead you will need to use Azure VM to access it.
3. In Azure VM, logon to Azure portal, select the disk "DC10-data-disk1" for DC1 and "DC20-data-disk1" for DC2, click Disk Export, click Generate URL and then click to download the disk to your VM. Mount the disk.
4. You can also use the following PowerShell to copy disk to your storage account (you'll need to create one)

In Cloud Shell:

	$rgName = 'alias-ADELAB'
	$disk = 'DC10-data-disk1'
	$saName = 'nameoftargetstorageaccount'
	$saKey = 'storageaccountkey'
	$saContainer = 'containername'

	$sas = Grant-AzureRmDiskAccess -ResourceGroupName $rgName -DiskName $disk -DurationInSecond 600 -Access Read
	$destContext = New-AzureStorageContext –StorageAccountName $saName -StorageAccountKey $saKey
	$copyBlob = start-AzureStorageBlobCopy -AbsoluteUri $sas.AccessSAS -DestContainer $saContainer -DestContext $destContext -DestBlob disk.vhd
	$copyBlob | Get-AzureStorageBlobCopyState

Above steps should copy disk into storage account. You can download it to your Azure VM, mount it and attempt access data.

# References
Azure Disk Encryption https://docs.microsoft.com/en-us/azure/security/azure-security-disk-encryption-prerequisites



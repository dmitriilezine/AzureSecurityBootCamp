
# VM Preparation for Azure Labs
## Create Windows 10 Pro 1809 VM 

0. Run the following template.  https://portal.azure.com/microsoft.onmicrosoft.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdmitriilezine%2FSingle-VM%2Fmaster%2FSingle%20VM%2Fazuredeploy.json
1. Create new Resource Group, name it 'alias-WIN10PRO', where 'alias' is your alias
2. For location select "West US"
3. Admin password is created as "Subscription#YOURSUBSCRIPTIONID". You can replace it with your own strong password
4. Replace Source Client IP with Public IP of your device, or put "*" to allow any source
5. Under "Windows OS Type" parameter select Win10Pro1809
6. Virtual Machine Size Standard_DS12_v2 to have speedy lab experienses 
7. Click "Purchase"
8. VM will be deployed in about 6 to 8 minutes. This VM is deployed with autoshutdown scheduled for 11PM EST.

## Create Windows 10 VM with Visual Studio 2017

0. Run the following template.  https://portal.azure.com/microsoft.onmicrosoft.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdmitriilezine%2FSingle-VM%2Fmaster%2FSingle%20VM%2Fazuredeploy.json
1. Create new Resource Group, name it 'alias-WIN10VS2017', where 'alias' is your alias
2. For location select "West US"
3. Admin password is created as "Subscription#YOURSUBSCRIPTIONID". You can replace it with your own strong password
4. Replace Source Client IP with Public IP of your device, or put "*" to allow any source
5. Under "Windows OS Type" parameter select Win10WithVSC2017
6. Virtual Machine Size Standard_DS12_v2 to have speedy lab experienses 
7. Click "Purchase"
8. VM will be deployed in about 6 to 8 minutes. This VM is deployed with autoshutdown scheduled for 11PM EST.
9. Logon to this VM via RDP
10. Start Visual Studio and initiate update to get it up to date with latest releases

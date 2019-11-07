# Hybrid Identity

## Task 1 - Deploy a Domain Controller
You will setup an IaaS VM with Active Directory via a JSON template from GitHub.  Although this domain controller is the in the cloud, we’ll use it to simulate an on-premises domain controller.

| Username  | Password         |
|-----------|------------------|
| adadmin | Azuret1workshop! |

### Cloud Shell

Declare the necessary variables and create the Resource Group for the lab.

```bash
location=<selected region>
name_suffix="<your student id>"
group_name=identitylab
az group create -l $location -n $group_name
```

Now create the deployment of the AD instance.

```bash
template_uri=https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/active-directory-new-domain/azuredeploy.json

az group deployment create --resource-group $group_name --name domaincontroller \
  --template-uri $template_uri\
  --parameters \
    adminUsername=adadmin \
    adminPassword=Azuret1workshop! \
    domainName=azureworkshop.io \
    dnsPrefix=$name_suffix

echo "Public FQDN: ${name_suffix}.${location}.cloudapp.azure.com"
```

### Portal

1. Login to your Azure subscription.
2. Go to <https://azure.microsoft.com/en-us/resources/templates/active-directory-new-domain/>
3. Select **Deploy to Azure**. The Azure Portal will open and a template will appear.
4. Enter the following information:
    * Resource Group: Create New: **identitylab**
    * Location: Pick a supported location
    * Admin Username: Enter **adadmin** 
    * Admin Password: Enter **Azuret1workshop!**
    * Domain name:  azureworkshop.io
    * DNS Prefix: **`<your student id>`**
5. Scroll down and select  **I agree to the terms and conditions stated above** and then **Purchase**.  

    > Monitor the deployment by clicking on the “Deploying Template deployment” tile within the Azure Portal.
    > Confirm that you don’t have any validation errors.  If you do, correct them before moving forward.
    > If the deployment fails, examine the logs to see what the root cause is, and then delete the Resource Group and start again.

_NOTE: The deployment and build of the VM can take ~30 minutes to provision and install DNS + Domain Controller.  Please return to the instructor’s presentation._

## Task 2 - Connect to Domain Controller and Create a Domain User

| Username  | Password         |
|-----------|------------------|
| adadmin | Azuret1workshop! |

### Portal

1. In the Search (top of the portal), type "adVM" and select the resource from the results
2. Click on **Download RDP File**.
3. Logon with  _adadmin@azureworkshop.io_ credentials

### RDP Session

4. If prompted, click **No** on the Network Discovery blade.
5. Within Server Manager, click **Tools** and then **Active Directory Users and Computers**.
6. Expand the tree and select the **Users Container**.
7. On the toolbar click the icon to create a new user in the current container.  
8. Create a New User with the following information:
    * First Name: **On**
    * Last Name: **Prem**
    * Full Name: **On Prem**
    * User Logon Name: **onprem**
9. Click **Next** and set the password to *Azuret1workshop!*
10. Uncheck **User must change password at next logon** and set the **Password never expires** checkbox.
11. Click **Next** then **Finish**

#### Powershell (Run as administrator)
The following can also be used to create the user using PowerShell.

```powershell
# create the new user
Import-Module ActiveDirectory

$firstName="On"
$lastName="Prem"

New-ADUser -Name "$firstName $lastName" -GivenName $firstName -Surname $lastName `
    -SamAccountName "onprem" -UserPrincipalName "onprem@azureworkshop.io" `
    -Path "CN=Users,DC=azureworkshop,DC=io" `
    -AccountPassword (ConvertTo-SecureString "Azuret1workshop!" -AsPlainText -Force) -PasswordNeverExpires `
    -Enabled $true
```

## Task 3 - Create an AD Connect Server

### Cloud Shell

```bash
group_name=$(az group list --query "[?contains(name, 'tiw')].name" -o tsv)
location=$(az group list --query "[?contains(name, 'tiw')].location" -o tsv)
vm_name=ADConnect

az vm create -g $group_name -n $vm_name \
    --image win2016datacenter \
    --admin-username azureuser \
    --admin-password "Azuret1workshop!" \
    --size Standard_DS2_v2 \
    --nsg-rule RDP
```

We are creating a small VM to be used later to host Azure AD Connect.

1. Return to the Azure portal and click the **Create a Resource** button (the Plus) found on the upper left-hand corner of the Azure portal.
2. Select **Compute** then select **Virtual machine**.
3. On the Basics tab complete the following:
    * Resource Group: **AZDCRG**
    * Virtual machine name: **ADConnect**
    * Region: Choose the same region as your domain controller
    * Availability options: No infrastructure redundancy required
    * Image: Windows Server 2016 Datacenter
    * Size: Choose **DS2_v2**
    * Username: **ADAdmin**
    * Password: `Complex.Password`
    * Confirm Password: `Complex.Password`
    * Public inbound ports: **Allow selected ports**
    * Select inbound ports: **RDP (3389)**
4. Click **Review + create** and then **Create**.   After validation passes, monitor your deployment status. It should take less than 10 minutes to spin up the VM.

## Task 4 - Join the ADConnect VM to the domain

1. Connect to the **ADConnect** virtual machine and logon as ADAdmin. **Microsoft Azure / Resource Groups / AZDCRG / ADConnect / Connect.**
2. If prompted, click **No** on the Network discovery blade.
3. Depending on which region you chose for setup, the ADConnect VM may or may not have the DNS server set to a value we need.
4. The DNS Server on ADCONNECT may not be set to see the domain controller (adVM), so we need to check that setting.  
5. Open a **Command prompt** and enter *ipconfig /all*.  If the DNS Server is set to 10.0.0.4 (the private IP address of adVM), close the Command Prompt window and then continue to **Task 5 - Join the Domain**, otherwise proceed to the **Configure DNS** set of tasks.

### Configure DNS

1. Within **Server Manager**, click on **Local Server**.
2. Click on **IPv4 address assigned by DHCP, IPv6 enabled setting** for the Ethernet connection.
3. Right-click on the network adapter and choose **Properties**.
4. Select **Internet Protocol Version 4 (TCP/IPv4)** and then click **Properties**.
5. Select the radio button for **Use the following DNS Server addresses:** and Set the DNS server to **10.0.0.4** and click **OK** and then **Close**.
6. You will then lose connection to the ADConnect VM, this is expected. Once you are back at the Microsoft Azure Portal, click **Restart** to restart the ADConnect VM.
7. Once the VM is successfully restarted, connect to the ADConnect VM and logon as ADAdmin.

## Task 5 - Join the Domain

1. Within **Server Manager**, click on **Local Server**.
2. Click on **WORKGROUP**, then **Change** to rename this computer or join it to a domain.
3. Click the radio button for **Domain**, enter your fully-qualified domain name, such as mydomainname.com, and click **OK**.
4. In the Windows Security box enter the AD Domain Admin credentials you specified in the template.
5. Click **Ok** on the Welcome screen, **Ok** on the Computer Name/Domain Changes window, **Close**, then **Restart Now**.

## Task 6 - Create an Azure Active Directory Tenant

1. In the Azure Portal, click  **+Create a resource** and then select **Identity**, then **Azure Active Directory**.
2. Enter the following on the **Create directory tab**:
    * Organization name (e.g. *MyOrg*)
    * Initial domain name (e.g. your initials plus last four of your cellphone)
        * Ensure validation passes as your namespace needs to be unique within the *.onmicrosoft.com namespace.  We often see students choosing a domain name that already exists.  
        * _You should write this initial domain  and directory name down._
3. Click **Create**.  It will take several minutes for the directory to be created.
4. Once complete, select Click **here** to manage your new directory.

## Task 7 - Create a Sync Account

We are going to create an account that AD Connect will use to perform the synchronization process bethween the on-prem domain controller and Azure Active Directory.

1. In Azure Active Directory, under **Manage** choose **Users** and then under **All users** click on **+New User** and enter the following:
    * User name: **adsync**
    * Name: **AD Sync Account**
    * Click on **Show Password** and then copy the password.
    * Under **Roles** click **User**.  Search for and select a directory role named: **Global administrator**. Select than and then click **Select**.
2. Click **Create**.
3. Open an InPrivate or Incognito browser and surf to <https://portal.azure.com.>
4. Login as the AD Sync Account you just created using the temporary password.
5. Change your password to *Complex.Password* and then click **Sign in**.
6. Close your inprivate or incognito browser.

## Task 8 - Sync Azure AD with Windows Server AD (AD DS)

### Install Azure Active Directory Connect

1. Connect to the ADConnect VM and logon as your previously created **domain account** (i.e. **domainname\username**, not adadmin which is a local account).  If you don’t see the VM, you must switch from the new Azure Active Directory  you just created to the **Default Directory** associated with your subscription.  Click in the upper right-hand corner of the screen to change directories.
2. When Server Manager opens select Local Server and turn off IE Enhanced Security Configuration for Administrators and Users.
3. Open Internet Explorer, accept the defaults, and surf to <http://go.microsoft.com/fwlink/?LinkId=615771>
4. Click **Download**, then **Run** when prompted.
Close Internet Explorer.

## Task 9 -  Configure Azure Active Directory Connect

1. On the Welcome to Azure AD Connect screen select **I agree** then **Continue**.
2. Review the screen and select **Use express settings**.
3. On the **Connect to Azure AD** screen enter your **Azure AD Credentials**.  This would be the *adsync@yourdirectoryname.onmicrosoft.com*  account you created.  Click **Next** and then confirm the credential are validated.
4. On the **Connect to AD DS screen**, enter the Active Directory Domain Services domain administrator credentials. This would be the account you created in the original template (i.e. mydomain\myusername). Click **Next** and confirm the credential are validated.  
    * If you get an error about the current security context is not associated with an Active Directory domain or forest, you more than likely didn’t logon with a domain account but rather a local account.  You can verify this by opening a command prompt and entering **whoami**.  Logout and login with a domain account and then restart at step 1 in this section.
5. On the **Azure AD sign-in configuration** screen, select the checkbox for **Continue without any verified domains** and click **Next**.
    * Since this is a temporary lab environment we are not going use a validated custom domain.
6. On the **Ready to Configure** screen click **Install**.
7. It may take 5-10 minutes for Azure AD Connect to complete installation. Read the **Configuration Complete** screen and then click **Exit**.
8. Minimize your RDP window.

## Task 10 - Validate Synchronization

1. Switch to the Azure portal and examine your Azure AD Directory by selecting the xxxx.onmicrosoft.com  Directory from the upper right hand corner of the portal.
2. Note that you should see accounts sourced from Active Directory that have synchronized to Azure Active Directory (e.g. On Prem).

### Congratulations!  Your are now synchronizing Active Directory to Azure Active Directory

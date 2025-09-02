# Active Directory Lab 

## Objective


In this project, I simulate a basic domain environment with Microsoft Active Directory. My goal throughout this lab is to understand and implement the fundamentals of Active Directory, how it is initially configured,
and understand the relationship between a Domain Controller and its clients within a LAN. We'll accomplish this by deploying all of the environment via virtual machines with Oracle VirtualBox. Windows Server 2022 will be configured as the Domain Controller, and Windows 11 as domain-joined clients. We will also automate the creation of users through a Powershell script. This lab, for me, was a fantastic introduction to 
Active Directory! 

### Skills Learned


- Utilizing Oracle Virtualbox to spin up virtual machines
- Configuring Active Directory Domain Services (AD DS) using Windows Server 2022
- Network simulation in a virtual NAT/Internal Network, managing IP addresses
- Troubleshooting connectivity and domain name resolution
- DHCP & DNS roles and setup
- User and Computer management
- Joining Windows clients to a domain
- Group Policy Objects (GPO)
- Resetting user passwords and configuring groups

### Environments Used


- Oracle VirtualBox
- Windows Server 2022
- Windows 11
- Microsoft Active Directory
- Windows CMD and Powershell

### On-prem Network Settings diagram 
For this lab, we will be using the following IP settings respective to our Domain Controller's External and Internal NICs and our Client's single, Internal NIC. 
The IP address scope was chosen because it is an open private IP range, allowing our Domain Controller's DHCP to assign our client an IP address in that range for use in the internal network.  

![ADLabDiagram](https://github.com/user-attachments/assets/fea4af53-2658-45ea-b3b8-c3fe1f587e18)

## Steps

### Step 1: Install VirtualBox

1. Download the updated version of [Oracle VirtualBox ](https://www.virtualbox.org/wiki/Downloads), according to your host OS.

2. Follow the installation instructions.
<br />

![Download VB](https://github.com/user-attachments/assets/51b227f3-fd3f-4d18-bdf4-b72800d3b06b)

### Step 2: Create and Install Windows Server 2022 VM

1. Download the [Windows Server 2022 ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022) from the MS Evaluation Center.

2. In VirtualBox, click on "New" and select the Server 2022 ISO. We will name it "DC01."

3. Check "Skip Unattended Installation." Next, allocate desired memory and processors- I chose 4gb and 2 cpu cores, with the default virtual hard disk setting (50gb).  Then, "Finish" the install.  
![Win2022VMInstall](https://github.com/user-attachments/assets/83c0e492-6e7e-4b8f-81ae-3f98631993d3)

4. In this VM's Settings, in the Network configuration, I enabled two adapters. Adapter 1 will be set to NAT. This allows the virtual machine to access the host's IP address for external internet connectivity. Adapter 2 is set
   to "Internal Network." This allows communication betweent the VMs- the domain controller and the client- while isolating the client from the external network. 
![Adapter1](https://github.com/user-attachments/assets/9f177d9b-a634-4c94-a18d-875319b7e82c)
![Adapter2](https://github.com/user-attachments/assets/317b04c5-f9f6-442c-8448-dc1ecea46e23)

5. Install Windows Server 2022. At the OS Setup options, we will opt for the "Standard Evaluation (Desktop Experience)", giving us a GUI instead of just a command line interface. Then, a custom installation specifying the virtual hard disk (50gb). We will also setup a local Adminstrator account. For the purpose of the homelab, we will set an easy-to-remember password. 
![Winserv2022](https://github.com/user-attachments/assets/51cffe98-31bc-49c1-b083-8cd6a9bbaa9f)
![Winserv2022Install](https://github.com/user-attachments/assets/96466c1f-93a9-4616-b148-0ec2dbded5b9)
![Winserv2022Login](https://github.com/user-attachments/assets/5cdb542a-846f-4129-97ff-322e49328901)

### Step 3: Configuring the Network Adapters
1. Once logged in, we will open "Network and Internet settings", then "Change adapter options."
   ![ChangeAdapter](https://github.com/user-attachments/assets/b72c681a-2e40-475b-b74f-014cf200739a)

2. To determine which ethernet adapter is internet facing and which is internal, I chose to run CMD and use ipconfig. Ethernet 1 has the valid home IP address (10.x.x.x), while Ethernet 2 has an APIPA address (169.254.x.x).
   This means that Eth 1 is internet (external) facing, while Eth 2 is internal facing. The same IP settings can also be found Ethernet>Options>Status>Details>IPv4 address. 
![CMDipconfig](https://github.com/user-attachments/assets/917d6065-8537-4be2-a4cb-a20575ffa453)

3. Ethernet 1 is renamed to "_INTERNET_" and Ethernet 2 is renamed to "_INTERNAL_". This distinction will be important for proper network routing and setting up the IP address.  
   ![RenameNIC](https://github.com/user-attachments/assets/d12b076d-7845-4ee3-997a-02c35e68becc)

4. We will set the Internal adapter settings per the diagram. We will assign it an IP address of 172.16.0.1 (as part of a free private IP address range). Subnet mask will be 255.255.255.0. The domain controller functions as the
   default gateway, so it will be left blank. The DNS server will be set as 127.0.0.1, which is the loopback address, because Active Directory will automatically configure DNS.
   This server will use its own IP address as the DNS server.
   The Internet adapter can be left alone, since it automatically retrieves its settings from the host network. 

![InternalNIC](https://github.com/user-attachments/assets/74f985e2-65ac-47e9-96ad-bbf37da08c2e)


### Step 4: Configuring Windows Server as a Domain Controller and Adding a Domain Admin Account

1. First, we will rename the PC itself to an appropriate title, as "DC01." Restart the PC to apply the changes. 
![RenamePC](https://github.com/user-attachments/assets/aafbe82f-4328-4e44-afc9-c3a23239b442)

2. In Server Manager, we will navigate to "Add roles and features." In the wizard, choose "Role-based or feature-based installation," then select our only available server, "DC01." 
![AddRoles](https://github.com/user-attachments/assets/ba263fed-62f4-480c-8064-4e801602e012)

We will select Active Directory Domain Services (AD DS) and follow the wizard to install it.
![ADDS](https://github.com/user-attachments/assets/2be2d8ea-570e-4e4a-b7c8-4074357d6a21)
![ADDS2](https://github.com/user-attachments/assets/b0dbf917-628a-4420-9055-38f9038c102f)
![ADDS3](https://github.com/user-attachments/assets/783fea44-54a0-4921-9abc-b04b4ddbf8b1)

3. Once AD DS is installed, navigate to the yellow flag notification at the top right, and "Promote this server to a domain controller."
![DCpromotion](https://github.com/user-attachments/assets/8a28dbb6-39b7-4d27-9b00-819a5b2a1904)

4. In the Deployment Configuration Settings, we will select "Add a new forest" and specify the root domain name. For this lab, I chose "skunkworks.local"

![domainname](https://github.com/user-attachments/assets/2ffe8d35-03b3-4741-8091-8499ab6f001b)

Follow the wizard to install. We will set a Directory Services Restore Mode (DSRM) password, and navigate "Next" through the wizard to install. 
After installation, the server will restart as a promoted domain controller with our assigned name and AD Domain Services installed! 
<br>
We can now login into the domain using the built-in administrator account. 

![DomainLogin](https://github.com/user-attachments/assets/11491073-8a95-4eea-8ede-6cd07291bc61)

5. First, we will create a dedicated "Admins" OU (Organizational Unit) in our domain tree, and later create a dedicated domain admin account. To do this, we will navigate to "Tools" at the upper right corner, and select "Active Directory Users and Computers."

![CreateAdmin](https://github.com/user-attachments/assets/40685747-2e22-4bb2-a65f-d1d1b62a1072)

Right-click our domain name to New>Organizational Unit. 

![CreateAdminOU_1](https://github.com/user-attachments/assets/f67f4bcc-b4b5-49fd-868c-b98a7a6e13f4)

Assign it the name- "ADMINS" 

![CreateAdminOU_2](https://github.com/user-attachments/assets/f39d73e2-0496-444c-a13e-5098ee659d7b)

6. In the ADMINS OU, right-click > New > User, and create a user according to your own preference, setting a password and assigning an appropriate username. 

![CreateAdminOU_3](https://github.com/user-attachments/assets/4daf503b-b08c-469e-b6a6-8164860b6fb5)

![CreateAdminOU_4](https://github.com/user-attachments/assets/dacff5aa-036f-417a-b1a3-258d69239d6e)

7. To make this account a domain administrator,  right-click on our user and select Properties, and navigate to the "Member Of" tab. Then "Add" the user to the "Domain Admins" group (using Check Names to ensure it resolves correctly.)
   
![CreateAdminUser](https://github.com/user-attachments/assets/85d93a38-aead-453b-85fa-cadee111a098)

Select Ok and Apply, ensuring that the user is now a member of the Domain Admins group. This user is now a domain administrator! We can now login as this admin user.

![CreateAdminUser](https://github.com/user-attachments/assets/9098b255-113a-45f7-b2a5-8453c9e3a0c9)


### Step 5: Install RAS/NAT 

We will now install RAS/NAT (Remote Access Service/Network Address Translation) on the domain controller. RAS and NAT allows our domain controller to facilitate network connectivity to the Windows 11 client, via the Internal adapter (which
we configured earlier). The client will operate within the private, virtual INTERNAL network, while still having internet access through the domain controller's INTERNET adapter. 

1. In the Server Manager, select "Add roles and features." Select "Role-Based Installation", and select the Domain Controller server (DC01.skunkworks.local). In Server Roles, select "Remote Access." On the next page, ensure "Routing"
   is checked. Navigate "Next" through the automatically applied settings to install.
   
![RASNAT](https://github.com/user-attachments/assets/bb6c1406-efa3-4314-85d9-dc82056525fb)
![RASNAT_2](https://github.com/user-attachments/assets/76b4be01-9ea3-40e3-b3cb-6de76ae7bf7b)

2. After installation is complete, navigate to "Tools" > "Routing and Remote Access." Right click on the Server and select "Configure and Enable Routing and Remote Access." Select "NAT." This option allows our internal clients to
   access the internet using the External NIC's IP address. 

![RASNAT_3](https://github.com/user-attachments/assets/4ef7e1ea-8b4d-4284-b454-a8404bd2fa97)

3. Select the adapter that we specified and renamed "_INTERNET_". This is why it was important to differentiate between the external and internal adapters. 

![RASNAT_4](https://github.com/user-attachments/assets/f4bd99b9-f432-49f2-87f9-bb49fb0b2063)

Once finished, our Server should now display connectivity. 

![RASNAT_5](https://github.com/user-attachments/assets/7ed61f02-6fa4-4901-869a-265f5618d776)


### Step 6: Install DHCP Server Role and scope out IP ranges

We will now install and configure the DHCP Server. Our clients (Windows 11) will obtain their private IP address through our Domain Controller's DHCP, automatically assigning them a static IP based on our selected IP address scope. 

1. In Server Manager, once again navigate to "Add Roles and Features." Select "Role-Based" and click Next. At Server Selection, select our server (DC01) and confirm.

2. Select "DHCP" and select Add Features, then click Next. Finish and install.

![DHCP](https://github.com/user-attachments/assets/dcadbd73-94fa-4a54-87ab-1fe2e7916400)


3. Once DHCP is installed, navigate to "Tools" > "DHCP". We are going to configure the automatic IP address assignment scope.
   Right click on "IPv4" and select "New Scope." For our scope name, we will assign it "172.16.0.100-200"

![DHCPScope](https://github.com/user-attachments/assets/58781f0a-8a1d-4144-88f7-00119af4da99)

4. Set the Start IP address as 172.16.0.100. Set the End IP address as 172.16.0.200. This means that clients on our internal network will be automatically assigned an IP between 172.16.0.100 - x.x.x.200.
   Set Length as "24" (as per CIDR notation). This sets our Subnet mask as 255.255.255.0.

![DHCPScope_2](https://github.com/user-attachments/assets/2270c530-55ba-479e-98f2-f1b0dd1ab956)

5. Click Next until the Router page. We will enter our Internal NIC's IP address: 172.16.0.1, and "Add." This means that our clients will use our Internal Adapter address as the Gateway, enabled by the Routing feature. 

![DHCPScope_3](https://github.com/user-attachments/assets/46a06962-f29a-437e-bd3b-4b2c85eca9b4)

6. Navigate Next until the end, and finalize the scope. Right click on our server ("dc01.skunkworks.local") and select "Authorize", then Refresh it to ensure it's active.
   Once the IPv4 icon turns green, the scope is now active. 
![DHCPScope_4](https://github.com/user-attachments/assets/c3e4e48e-941c-497b-b4d6-d5da7e81b133)


### Step 7: Generate sample users automatically using Powershell
We will utilize a Powershell script to automate user creation in Active Directory, simulating a populated corporate Domain Controller and granting us Users that we can manage in AD!

1. On the Desktop (or any preferred location), create a folder named `AD_User_Creation`. We will populate this folder with two files.
2. Create a txt file named `names.txt`. We will add the following sample names into it:
   
```
Bertram Gilfoyle
Elliot Alderson
Alva Cosey
Maxie Dickey
Carrol Tavarez
Socorro Mosbey
Paulette Thrash
Leilani Isenberg
Nickie Rodriquez
Vada Wilkin
Ali Rezendes
Edgardo Cornette
Jeffie Wirth
Jeannette Oboyle
Rima Dankert
```
I chose to randomize names and add a certain amount. You are able to add as many names as you want.

4. Using Powershell ISE, create a new .ps1 file and copy/paste the following code:

```
# ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password2025" 
$USER_FIRST_LAST_LIST = Get-Content .\names.txt
# ------------------------------------------------------ #

$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false

foreach ($n in $USER_FIRST_LAST_LIST) {
    $first = $n.Split(" ")[0].ToLower()
    $last = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()
    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $first `
               -Surname $last `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_USERS,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
}
```
 
![CreateUsers](https://github.com/user-attachments/assets/a78cd1a1-10d6-4ac2-9984-945571dca4dd)

Save it as `CREATE_USERS.ps1` in our `AD_User_Creation` folder. We should now have two files, the .ps1 file and the names.txt. 

5. Running the script: Open **Windows Powershell ISE** as Administrator. Click File > Open > then navigate to the `CREATE_USERS.ps1` files to execute the script. You may need to copy `names.txt` to C:\Users\Administrator path.
   Select the green "Run" arrow in the options toolbar.
![RunScript](https://github.com/user-attachments/assets/65ef3e3c-e976-46cd-86f6-285fc755a48a)

6. We will confirm the script's success. In the Server Manager, navigate to Tools > Active Directory Users and Computers. There should be a new OU created: `_USERS`. It is now populated with User accounts using the names
   we specified in `names.txt`. Each user now has a standardized username (`firstinitial-lastname`) and their own password. We are able to manage them, add them to Groups, configure access, and reset their passwords!

![UsersCreated](https://github.com/user-attachments/assets/dc740dd2-7835-4f4c-aecb-be1f53878726)


### Step 8: Creating and Configuring Windows 11 VM
We will be setting up a Windows 11 Enterprise client in a VM. 

1. Download the [Windows 11 Enterprise ISO Image](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise) from Microsoft's Evalutation Center. 
2. Click "New". Choose the Windows 11 Enterprise ISO. We will name it "Windows 11 Client." Click "Skip Unattended Installation." 

![Win11](https://github.com/user-attachments/assets/ed6bb0b9-e4df-4314-a422-a1f88268b854)

3. Similar to the Server 2022 VM, allocate CPU core count, RAM size, and virtual hard disk size.

4. In the VM's settings, navigate to the Network tab. Ensure that Adapter 1 is enabled, and that it is set to "Internal Network." 

![Intnet](https://github.com/user-attachments/assets/2a2429b9-387e-445c-b1d1-79487fa27daa)


5. Start the Windows 11 VM to install. Navigate through standard installation procedures, and let it install. 

![Win11Install](https://github.com/user-attachments/assets/ade4402f-8985-4c62-82c7-3e7138b4a7e9)

6. **Ensure Internal Routing Connectivity** : Once our local user account is configured and Windows 11 is installed, let's verify that our IP address is properly configured. We set this VM's Network Adapter as Internal, so it will
   use our Domain Controller as the default gateway, obtaining a valid IP address from our Domain Controller's DHCP scope.
   To verify, I use `ipconfig` command in Command Prompt.
   
![Win11](https://github.com/user-attachments/assets/cce59b53-1a08-414a-a212-1f2b495d4f05)

The default gateway is as expected: our Domain Controller's static IP. Our DNS is listed: skunkworks.local. Our assigned IP address is 172.16.0.100, which is within the scope. To verify internet connectivity via NAT, I `ping`
google.com, and see successful responses in outbound traffic. Success! This verifies our internal network and connection to the DC is functional. 

(**Troubleshooting**: Initially, the Client was not able to reach the DHCP server and acquire an IP address. `Ping` also returned with errors. I then ran `ipconfig /renew` in an effort to establish comms, and was also rejected. I restarted the client's network adapter. I then investigated the IP settings and ensured that the Client's DHCP and DNS were Automatic, and the Server's _Internal_ IP settings were correct. I then inspected the Advanced Sharing Settings of both the Client and the Server, and found that the Client's Private Network Discovery option was "off." Turning it on, allowing the PC to find and be found by other devices on the network, solved the issue.)

We can also check in our Domain Server under `Server Manager > Tools > DHCP > IPv4 > Scope > Address Leases` and see that our Client (with its default name) is visible and active! Later, when we fully change our Client name, this
will reflect the name change. This means that our client reached out to the DHCP server to request an IP address, and it was successfully configured. 

![Active](https://github.com/user-attachments/assets/9845604b-05b9-4a81-ad71-3ea67b05faf3)


### Step 9: Joining our Windows 11 Client to the Domain

We will rename our Client and join it to our Domain, allowing any verified user to log into their domain user account from this client.

1. Navigate `Start > About > Domain or Workgroup` and select "Change." 

![ChangeName](https://github.com/user-attachments/assets/41a9b95d-d9d0-485e-9df5-8aba8f3723c2)

2. Set the computer name to an appropriate title- I chose "WS01," meaning "Workstation 01."

3. Set the "Member of" section to our domain: "skunkworks.local" Select OK. 

![ChangeName_2](https://github.com/user-attachments/assets/0a08347b-a3e9-4952-806b-4516f7eda3d5)

4. You will be prompted to enter the Domain Admin account credentials. This account, created earlier, is authorized to join this computer to the domain. Follow the prompts. After clicking OK, we have a welcome message
   verifying that we have joined skunkworks.local domain!

![ChangeName_3](https://github.com/user-attachments/assets/a0a6b672-b421-4fd4-bae6-63d1299a19ab)

6. Restart Windows 11 to fully apply the changes and verify by logging in using any of the user credentials we created earlier.
![Login](https://github.com/user-attachments/assets/1849e267-5b9d-491d-9e01-738d17bfcf52)

7. **Verify**: By using `whoami` command in CMD, we can list our account name and connected domain. This demonstrates that our client is now operating within the domain and with the assigned privileges in AD. 

![Verify](https://github.com/user-attachments/assets/5e52b6f8-0bbe-438d-a161-24b920e072c8)

8. From here, we can manage our users extensively and complete tasks like unlocking account and resetting passwords...

![PasswordReset](https://github.com/user-attachments/assets/a615cf05-f000-417e-a8cf-debe3fb32fe3)

Create Groups and add users to specific Groups, assigning permissions...

![Groups](https://github.com/user-attachments/assets/c391a035-bdc6-4d91-bbaa-81b83e32d46b)

Creating GPOs (Group Policy Objects) 

![GPO](https://github.com/user-attachments/assets/9fb2447f-2078-4c77-a2bb-4ebbae2117ca)


## Conclusion

We have now successfully stood up a fully functioning Active Directory environment, with Windows Server 2022 as the Domain Controller and Windows 11 Enterprise as the client system, all within Oracle VirtualBox. I replicated the steps in 
a Windows 10 client as well, and joined it to the domain. Through this basic homelab, I was able to gain hands-on experience in:
- Installing and configuring Active Directory Domain Services;
- Configuring network settings and roles; DNS, DHCP for dynamic IP allocation, and NAT for routing, all for setting up internal and external communication;
- Troubleshooting network connectivity issues;
- Creating and automating user accounts;
- Practicing user administration tasks such as resetting passwords, creating GPOs and assigning users to Groups with specific security configurations;
- Joining Windows 11 and Windows 10 clients to the Domain and verifying connectivity.

This homelab furthered my knowledge of domain management, Active Directory, and allowed me to troubleshoot real-world IT scenarios, and allowed me to gain confidence and explore foundational IT practices!
Much credit to [Josh Madakor](https://www.youtube.com/watch?v=MHsI8hJmggI) for this introductory homelab.
Next will be learning ticketing systems (osTicket) and MS Entra ID/ Azure. 



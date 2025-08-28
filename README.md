# Active Directory Lab 

## Objective


In this project, I simulate a basic domain environment with Microsoft Active Directory. My goal throughout this lab is to understand and implement the fundamentals of Active Directory, how it is initially configured,
and understand the relationship between a Domain Controller and its clients within a LAN. We'll accomplish this by deploying all of the environment via virtual machines with Oracle VirtualBox. Windows Server 2022 will be configured as the Domain Controller, and Windows 11 (and, as a bonus, Windows 10) as domain-joined clients. We will also automate the creation of users through a Powershell script. This lab is very basic- yet extremely informative- introduction to 
Active Directory and networking. 

### Skills Learned


- Utilizing Oracle Virtualbox to spin up virtual machines
- Configuring Active Directory Domain Services (AD DS) using Windows Server 2022
- Network simulation in a virtual NAT/Internal Network, managing IP addresses
- Troubleshooting connectivity and domain name resolution
- DNS & DHCP roles and setup
- User and Computer management
- Joining Windows clients to a domain
- Group Policy Objects (GPO)
- Resetting user passwords and configuring groups

### Environments Used


- Oracle VirtualBox
- Windows Server 2022
- Windows 10 & 11
- Microsoft Active Directory
- Windows CMD and Powershell

## Steps

### Install VirtualBox

-Download [Oracle VirtualBox ](https://www.virtualbox.org/wiki/Downloads) and follow the installation instructions.
<br />
![Download VB](https://github.com/user-attachments/assets/51b227f3-fd3f-4d18-bdf4-b72800d3b06b)

### Create Windows Server 2022 VM

-Download the [Windows Server 2022 ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)
-In VirtualBox, click on "New" and select the Server 2022 ISO. We will name it "DC01." 
![Win2022 VM Install](https://github.com/user-attachments/assets/83c0e492-6e7e-4b8f-81ae-3f98631993d3)

- Check "Skip Unattended Installation." Next, allocate desired memory and processors- I chose 4gb and 2 cpu cores. 

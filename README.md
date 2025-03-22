# Basic Active Directory Deployment

This project concerns the basic creation and deployment of an Active Directory domain. We will create a new Windows Server 2022 virtual machine in VMware. Then, we will configure the VMware LAN segment IP configuration so that both PCs are able to communicate with each other with static IPs. Afterward, we will install Active Directory and create and join the Windows VM to the domain. Finally, we will use a PowerShell script to create many bogus users and log into those users using the Basic Windows VM.

## Environments and Technologies Used

- “Basic” Virtual Machine in VMware
- Windows Server 2022 Domain Controller
- [PowerShell Generate Users Script](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1)

## Operating Systems Used
-  Windows 10 (22H2)
- Windows Server 2022 Evaluation ISO
    - [Download From The Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)

## Project Walk-Through
First, we will need to create a new virtual machine with Windows Server 2022 installed on it. You should be familiar with the steps from the previous “Installing Virtual Machines in VMware” lab. However, make sure that you have Windows Server 2022 selected as the Version.

<img src="https://i.imgur.com/16qtNKP.png" height="70%" width="70%"/>

While installing, you will receive a popup that will let you select the operating system you want to install. Make sure you select the Windows Server 2022 Datacenter Evaluation (Desktop Experience) so you will have access to the Operating System GUI (Graphical User Interface). After that, continue the normal installation process (Custom Install, New Partition Setup, etc.).

<img src="https://i.imgur.com/Frj6t3K.png" height="70%" width="70%"/>

When given the option to customize your settings, make sure that the password is Password123! After that, don’t forget to install VMware tools.

<img src="https://i.imgur.com/rtIR8tn.png" height="70%" width="70%"/>

After successfully installing Windows Server 2022 and installing VMware tools, we will now be creating a LAN Segment so that the two VMs can communicate only with each other. 

Right-click the Windows Server 2022 VM > Settings > Network Adapter > LAN Segment > LAN Segments… > Add > Name: Private > OK > Change LAN Segment to Private. Change the “Basic” VM’s LAN Segment to Private as well. It will work when you notice that both VMs lose their internet connection.

<img src="https://i.imgur.com/nHXTCcT.png" height="70%" width="70%"/>

Now, we will manually configure some IP addresses. On the Windows Server VM, type network connections into the search bar > Right-Click Ethernet0 > Properties > Internet Protocol Version 4 > Properties > Use the following IP Address: Configure the VMs with the following IP addresses:

- Windows Server 2022 VM
    - IP Address: 192.168.10.101
    - Subnet Mask: 255.255.255.0
- “Base” Windows VM
    - IP Address: 192.168.10.102
    - Subnet Mask: 255.255.255.0

<img src="https://i.imgur.com/Vq6I0Sk.png" height="70%" width="70%"/>

To verify that everything is set properly, go to the Windows 10 VM, run Command Prompt as an administrator, and enter the following command:

    netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=in action=allow

Then, use ipconfig for both VMs' command prompts and ping the Windows 10 VM from the Windows Server VM, using the listed IPv4 Addresses. If the addresses are not 192.168… and are instead 169.254… then use the ipconfig /release command.

<img src="https://i.imgur.com/HM0VwYI.png" height="70%" width="70%"/>

After confirming connectivity, we will now use the Windows Server as the default gateway for the Windows VM. Right-click the Windows Server VM > Settings > Add… > Network Adapter.

<img src="https://i.imgur.com/UbylPhF.png" height="70%" width="70%"/>

Then, go back into the Windows Server VM > Ethernet 1 Properties from the Network Connections window > Sharing > Check the “Allow other network users to connect through this…” setting.

<img src="https://i.imgur.com/0lSAVST.png" height="70%" width="70%"/>

Then, go back into the Network Connections window and change the Ethernet0 IPv4 Address back to 192.168.10.101

<img src="https://i.imgur.com/kgsKw3y.png" height="70%" width="70%"/>

Go to the IPv4 settings of the “Basic” Windows VM and change the Default Gateway to 192.168.10.101 and the DNS server to the same IP. Although it might take a while, eventually, the Basic Windows VM should get internet access.

<img src="https://i.imgur.com/bfKcdUW.png" height="70%" width="70%"/>

Now, we will be configuring Active Directory Domain Services. 

From the Windows Server VM > Server Manager > Add Roles and Features > Next Until Server Roles (Enable **Active Directory Domain Services**) > Add Features > Next Until Confirmation (Restart The Destination Server) > Install

<img src="https://i.imgur.com/cVFlHwu.png" height="70%" width="70%"/>

Once installation is finished, click on the flag in Server Manager and click “Promote this server to a domain controller.” Select Add a new forest and name the Root domain name: mydomain.com. 

Additionally, configure the DRSM password as Password123! 

Also, **uncheck the DNS Delegation.** Continue the installation. This will take a while, and the VM will restart itself.

<img src="https://i.imgur.com/HAc8Hcu.png" height="70%" width="70%"/>

After the restart, notice that the Windows Server user is now MYDOMAIN\Administrator. Login again and open Active Directory Users and Computers from the search bar. Right-click mydomain.com > New > Organizational Unit > Create The Following Organizational Units:

- Name: _EMPLOYEES
- Name: _ADMINS

<img src="https://i.imgur.com/bfxxtvp.png" height="70%" width="70%"/>

In the _ADMINS folder, create a new user:

- Name: Jane Doe
- User Logon: janeadmin
- Password: Password123!
- Disable User must change password at next logon
- Enable Password never expires

<img src="https://i.imgur.com/kjdwowR.png" height="70%" width="70%"/>

After creating the Jane Doe account, add the account to the Domain Admins group. Right-click Jane > Properties > Member Of > Add… > Type Domain Admins + Check Names > OK

<img src="https://i.imgur.com/7pFTjv8.png" height="70%" width="70%"/>

Now, we will be joining the Basic VM to the domain. Type About Your PC in the Search Bar > Rename this PC (Advanced) > Change… > Domain: > mydomain.com. When prompted for a username and password, use the following parameters:

- User name: mydomain.com\janeadmin
- Password: Password123!

Then, you will be prompted to restart the computer to finalize the domain changes.

<img src="https://i.imgur.com/vUUk9g1.png" height="70%" width="70%"/>

Log back into the Windows Server VM. We will now be creating Users in Active Directory using the script. Open up PowerShell ISE as an Administrator. Press the arrow button to expand the Script pane and paste the script into the Script pane. Change the password to Password123! and change the number of users to 1000. After making the adjustments, press on the Green Play Button or F5 to run the script.

<img src="https://i.imgur.com/RyeY2eD.png" height="70%" width="70%"/>

After executing the script, notice that many users are being generated automatically. Go into the Active Directory Users and Computers and Right-Click the _EMPLOYEES folder > Refresh. When you do so, you will notice that the users have been generated. Go back into the Basic VM and try logging into one of the users. In my case, I will try logging into the lud.fin account that was generated automatically by the script.

<img src="https://i.imgur.com/lToBkHE.png" height="70%" width="70%"/>

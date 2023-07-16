<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop (Remmina)
- Active Directory Domain Services
- PowerShell (ISE)

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Step 1: Create Windows server and client VMs on same subnet within Azure
- Step 2: Install active directory on Windows Server and promote it as a Domain Controller
- Step 3: Create multiple "Organaization Units" and put users into their respective unit
- Step 4: Join Windows client to domain controller and allow access to non-admin users

<h2>Deployment and Configuration Steps</h2>


<p>
<img src="" width="80%"/>
</p>
<p>
We will begin by creating the the Windows Server that Active Directory(AD) will be installed on. So go within Microsoft Azure to virtual machine page and create a new virtual machine. We will create also create a new resource group for the server and client to grouped into so just create a new resource group. Configure the VM to use "Windows Server 2022 Datacenter". 2 CPU cores are sufficient for this server so select that then click "Review + create" then click create on the page. We will use the subnet that this VM will create for the client so that they are technically on the same network.
</p>

![Screenshot from 2023-07-12 10-41-59](https://github.com/jckaizen/azure-ad-port/assets/57122203/b9dd31bd-fa6e-45f4-9d48-f4ca08c9da29)

<p>
It will be exactly similar for the client only instead we will change the subnet and resource group to the same ones that the server uses. Also since this is just a client, we will use normal Windows 10.
</p>

![Screenshot from 2023-07-12 10-44-26](https://github.com/jckaizen/azure-ad-port/assets/57122203/82f77ea8-50ef-4be3-86cb-1e8e83282db4)

<p>
Now that both VMs are setup, we need the server's private IP to static for later use. Go to the server's NIC(network interface card), under Settings is IP configuration. In it, click to set the private IP to be static.</p>

![Screenshot from 2023-07-12 10-46-04](https://github.com/jckaizen/azure-ad-port/assets/57122203/b05dc7b2-b374-4c22-97b8-f04fa4ea13dc)

<p>Great. Now let's remote into the client. I will use Remmina since I'm on Linux. If you're on Windows, you can use the built-in remote desktop appliation. MacOS will need a third-party application as well.</p>

<p>Once logged in, open up the command prompt and perpetually ping <i>(ping x.x.x.x -t)</i> the private IP of the server. It should time out since, by default, Windows server has ICMP disable.</p>

![Screenshot from 2023-07-12 10-52-13](https://github.com/jckaizen/azure-ad-port/assets/57122203/18d0a577-37c3-4acd-9d36-295e028fe451)

<p>In order to enable it, we'll need to remote into the server and go into Windows Defender Firewall.</p>

![Screenshot from 2023-07-12 10-53-31](https://github.com/jckaizen/azure-ad-port/assets/57122203/06789b7d-4415-4e19-a239-97248b6c0cfd)

<p>Inside, go to the "Inbound rules" and enable ICMP Echo Request(ICMPv4-in); there should be two of them.</p>

![Screenshot from 2023-07-12 10-54-41](https://github.com/jckaizen/azure-ad-port/assets/57122203/48a9531e-d450-41f4-bf6c-3ac004d92a24)

<p>Once you switch back to your client, the pings should be working</p>

![Screenshot from 2023-07-12 10-55-44](https://github.com/jckaizen/azure-ad-port/assets/57122203/b82148f9-4602-48d5-aa8d-0dd36566e1cf)

<p>Switching to the server, go to "Server Manager" and click to add roles and features under "Manage".</p>

![Screenshot from 2023-07-12 10-57-49](https://github.com/jckaizen/azure-ad-port/assets/57122203/f99062b7-b5fc-41ad-a406-e7ac6ac81046)

<p>Click next until the "Server Roles" and check "Active Directory Domain Services" and click next. At the end, it will prompt you to install.</p>

![Screenshot from 2023-07-12 10-59-43](https://github.com/jckaizen/azure-ad-port/assets/57122203/1c5e728e-9252-4b40-98b1-39719908fbad)

<p>Once done, there should be a caution notice under the notications asking to promote the server to be a domain controller. Clikc on it.</p>

![Screenshot from 2023-07-12 11-02-25](https://github.com/jckaizen/azure-ad-port/assets/57122203/d30555d4-403a-434b-b692-4e02fe3c7791)

<p>Make a new forest and it can be any root domain name that you want. Note: In production, you would probably want to own the domain.</p>

![Screenshot from 2023-07-12 11-02-51](https://github.com/jckaizen/azure-ad-port/assets/57122203/bdf16d06-1699-4e17-a6f0-86637c045e91)

<p>Click through until you reach "Prerequisites Check" and click install. The server will restart. You will now have to use the domain to remote into the server VM, though give it a few minutes.</p>

![Screenshot from 2023-07-12 11-05-49](https://github.com/jckaizen/azure-ad-port/assets/57122203/8540f5f9-4693-4119-8b52-52bd840ce56c)

<p>Once back in, go into "Active Directory Users and Computers". Under the domain name, create two <a href="https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/understanding-the-active-directory-logical-model#active-directory-organizational-units">Organizational Units</a> named "_EMPLOYEES" and "_ADMINS". Inside employees, create a new user called "Jane Doe" with the username of "jane_admin" then add it to the "Domain Admins" Security Group.</p>

![Screenshot from 2023-07-12 11-25-19](https://github.com/jckaizen/azure-ad-port/assets/57122203/1906ac37-9b8a-4c56-9839-94627f36bebc)

<p>Disconnect from the server and try to log back in using the Jane account. It should work.</p>

![Screenshot from 2023-07-12 11-28-22](https://github.com/jckaizen/azure-ad-port/assets/57122203/0334205a-065f-42fa-a0f6-fccde9c2c098)

<p>With the domain controller setup, within Azure, go to the Windows client's network interface and set the DNS server to custom and to be the domain controller's private IP. Restart the client.</p>

![Screenshot from 2023-07-12 11-33-29](https://github.com/jckaizen/azure-ad-port/assets/57122203/2b42442e-3d39-4b35-8293-6f77de7b155e)

<p>Log into the client then go to system settings. Under the "About" section, click "Rename this PC (advanced)". Now click the change button to change the domain of the VM to that of the domain controller .</p>

![Screenshot from 2023-07-12 11-36-49](https://github.com/jckaizen/azure-ad-port/assets/57122203/f9315b19-0b2c-4385-aadd-8c17f01f43c0)

<p>If we switch back to the server, we can see that the client has joined the domain.</p>

![Screenshot from 2023-07-12 11-41-37](https://github.com/jckaizen/azure-ad-port/assets/57122203/996afd05-d4f2-4eca-86a3-b4dbee88adc2)

<p>Only admins can log into the client but we want non-admin to log in as well. So logging in as "jane_admin" on the client, go to system settings and then remote desktop. Click on "Select users that can remotely access this PC". Add "Domain Users". Normally you'll want to use group policy to allow you to do this to many systems at once, but this is just a quick way to do so.</p>

![Screenshot from 2023-07-12 11-49-08](https://github.com/jckaizen/azure-ad-port/assets/57122203/12ac46e9-96c7-46f5-90f5-3f9f063a6881)

<p>To see if this worked, we'll create many users using a powershell script. On the server, open "Powershell ISE" as an admin. Create a new file and paste this script in.</p>

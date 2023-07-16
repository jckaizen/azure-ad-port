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
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
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

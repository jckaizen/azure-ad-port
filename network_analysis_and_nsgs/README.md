<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines</h1>
In this tutorial, we observe various network traffic to and from Azure Virtual Machines with Wireshark as well as experiment with Network Security Groups.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Windows Command Prompt
- Various Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)
- Ubuntu Server 20.04 LTS

<h2>High-Level Steps</h2>

- Step 1: Create resource group and VMs within Azure
- Step 2: Remote into Windows client using RDP (remote desktop protocol)
- Step 3: Install wireshark
- Step 4: Observe various network protocol using wireshark

<h2>Actions and Observations</h2>

<p>
First, go to the Virtual Machine page of Azure and select to create a new VM. Since we can create a resource group from here, select "Create new" and give the resource group a name. This will hold all of the resources for the VMs. Then select Windows 10 and 2 CPU cores for the image and size boxes. Also create a username and password for it; make sure you remember it.
</p>

![Screenshot from 2023-07-11 11-29-22](https://github.com/jckaizen/azure-ad-port/assets/57122203/50137e05-eb3b-4210-b4e2-ef4d5ff573bd)


<p>You can skip straight to "Review + create", but keep in mind the name of the subnet that this VM will create for you.</p>

![Screenshot from 2023-07-11 11-29-07](https://github.com/jckaizen/azure-ad-port/assets/57122203/3f7d6f0e-7762-43f8-bbeb-92717df43c38)

<p>Next, create a new virtual machine. This VM will be an Ubuntu 20.04 LTS server. Put it under the same resource group as the Windows one and select 2 CPU cores for it as well. You also want to change "Authentication type" from "SSH public key" to "Password" since we aren't remoting into this machine using SSH. Make sure you remember the username and password as well (We will use SSH to remote into it but from the Windows client later on.)</p>

![Screenshot from 2023-07-11 11-31-43](https://github.com/jckaizen/azure-ad-port/assets/57122203/c0636964-9afb-446f-8d3e-56b3e7479d8a)

<p>Under "Networking", chagne the virtual network to be the same as the Windows client. This will put them in the same network. Afterwards, continue to "Review + create" and create the VM.</p>

![Screenshot from 2023-07-11 11-32-29](https://github.com/jckaizen/azure-ad-port/assets/57122203/9349b5fb-f684-4d7b-8513-7007e6eddb57)

<p>
We will remote into the Windows client using RDP or remote desktop protocol. Since I'm using Linux, I will use Remmina to remote into the VM. If you're on Windows, you can use the built-in remote desktop application. If on MacOS, you will have to use a third-party application to remote in.
</p>

![Screenshot from 2023-07-11 11-41-02](https://github.com/jckaizen/azure-ad-port/assets/57122203/0888cad0-550e-4f5f-950b-ab3ddc63181e)

<p>
Once logged in, go download Wireshark from the edge browser within the VM. Just keep everything default for the installation. Once done, open Wireshark.
</p>

![Screenshot from 2023-07-11 11-42-40](https://github.com/jckaizen/azure-ad-port/assets/57122203/f6ec16c9-365b-4499-8a39-0d1447eac9cb)

<p>You should see a blue shark fin that allows to start capturing packets to analyze. Click on it.</p>

<h3>ICMP</h3>

<p>ICMP is a specific packet when you're pinging a machine (mainly used to diagnose networks). So filter on "icmp" in the search bar. Then go to the command prompt and ping the private IP address of the Ubuntu server. It will be located within the network interface of Ubuntu VM within Azure.</p>

![Screenshot from 2023-07-11 11-48-06](https://github.com/jckaizen/azure-ad-port/assets/57122203/0e7195cc-f700-49d2-bf10-7ec14e46ba57)

<p>When you take a look at Wireshark, you will notice that 8 packets were detected (4 requests and 4 responses).</p>

![Screenshot from 2023-07-11 11-47-49](https://github.com/jckaizen/azure-ad-port/assets/57122203/36429bea-1d03-46c2-bf97-b6cf074a4ff6)

<p>You can also ping a normal website like www.google.com</p>

![Screenshot from 2023-07-11 11-48-45](https://github.com/jckaizen/azure-ad-port/assets/57122203/baee25d8-0487-4a73-a298-d987038bc060)

<p>You can also see a firewall blocking the IMCP. Ping the Ubuntu server perpetually <i>(ping x.x.x.x -t)</i> in the Windows VM. Then go to the network security group of the Ubuntu VM within Azure and under "Inbound security rules", create a rule to deny ICMP traffic.</p>

![Screenshot from 2023-07-11 11-50-13](https://github.com/jckaizen/azure-ad-port/assets/57122203/19d2293e-ffb1-4936-8551-bfe857a3ab13)

<p>You will start to notice requests timing out.</p>

![Screenshot from 2023-07-11 11-59-46](https://github.com/jckaizen/azure-ad-port/assets/57122203/f2324bfe-b5c3-4f0c-9cb3-b46807fbad84)

<p>If you delete the same rule or change it from deny to allow. You will notice the request starting to go through.</p>

![Screenshot from 2023-07-11 12-00-52](https://github.com/jckaizen/azure-ad-port/assets/57122203/df4770bb-836b-41af-8072-17cf89d0d081)

<h3>SSH</h3>

<p>SSH is mainly used to remote into servers that ish "headless" (no GUI) or you just need to use the terminal.</p>

<p>Filter on "ssh" in the search bar. Now we will ssh into our Ubuntu VM so type <i>ssh x.x.x.x</i>. The x's is the private IP address of the Ubuntu VM. Put in the username and password you created for that VM and enter. If you type any command on that machines you will notice all of the packets being captured.</p>

![Screenshot from 2023-07-11 12-05-43](https://github.com/jckaizen/azure-ad-port/assets/57122203/0e976f2a-ca09-4924-bf56-af954f639816)

![Screenshot from 2023-07-11 12-06-15](https://github.com/jckaizen/azure-ad-port/assets/57122203/1e9aa387-b378-4cf5-9e6a-ae815554516d)

<h3>DHCP</h3>

<p>Mainly found on consumer-grade routers, DHCP assigns private IP addresses to devices on the network.</p>

<p>Filter on "dhcp" in the search bar. The only way to observe this is if we ask for a new IP address from the DHCP server so in the command prompt, type "ipconfig /renew" and watch the packets being captured in Wireshark.</p>

![Screenshot from 2023-07-11 12-07-26](https://github.com/jckaizen/azure-ad-port/assets/57122203/73cb6809-bc90-464d-ad4e-1bf5447a4744)

<h3>DNS</h3>

<p>DNS is what allows us to use simple website names instead of the actual IP address of the website. Example: www.google.com is actually 142.250.189.4</p>

<p>Filter on "dns" in the search bar. Then in the command prompt, type "nslookup www.google.com". You will get back the actual IP (well one of the IP addresses) of google.</p>

![Screenshot from 2023-07-11 12-08-59](https://github.com/jckaizen/azure-ad-port/assets/57122203/a6aa8d78-13af-43c5-845e-b30b6c9b2d2c)

<p>And when you take a look at Wireshark you will notice queries for google.com</p>

![Screenshot from 2023-07-11 12-09-10](https://github.com/jckaizen/azure-ad-port/assets/57122203/815b6628-842d-4ede-af7a-cff048284029)

<h3>RDP</h3>

<p>RDP or remote desktop protocol is what allows you to remote into the desktop of a machine.</p>

<p>If you remote into your VM, filter on "tpc.port == 3389" (3389 is the port nubmer of RDP), you will notice a constant stream of packets since we are continuously connected the session. I can't show a screenshot it show my real IP address, but you get the point.</p>

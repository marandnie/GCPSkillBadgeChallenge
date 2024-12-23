## Task 1. Create the VPC network
Create a new VPC network called securenetwork.
Click Check my progress to verify the objective.
Create the VPC network.


Create a new VPC subnet inside securenetwork in the region region.
Click Check my progress to verify the objective.
Create the VPC subnet.


Once the network and subnet have been configured, configure a firewall rule that allows inbound RDP traffic (TCP port 3389) from the internet to the bastion host. This rule should be applied to the appropriate host using network tags.
Click Check my progress to verify the objective.
Create the firewall rule.


## Task 2. Deploy your Windows instances and configure user passwords
Deploy a Windows 2016 server (Server with Desktop Experience) instance called vm-securehost with two network interfaces in the zone zone.
Configure the first network interface with an internal only connection to the newly created VPC subnet.
The second network interface with an internal only connection to the default VPC network. This is the secure server.
Click Check my progress to verify the objective.
Create the vm-securehost instance.


Deploy a second Windows 2016 server (Server with Desktop Experience) instance called vm-bastionhost with two network interfaces in the zone zone.
Configure the first network interface to connect to the newly created VPC subnet with an ephemeral public (external NAT) address.
The second network interface with an internal only connection to the default VPC network. This is the jump box or bastion host.
Click Check my progress to verify the objective.
Create the vm-bastionhost instance.


Configure user passwords
After your Windows instances have been created, create a user account and reset the Windows passwords in order to connect to each instance.
NOTE: Copy the User name and Password of both instances for later use.
The following gcloud command creates a new user called app-admin and resets the password for a host called vm-bastionhost located in the placeholder zone:
gcloud compute reset-windows-password vm-bastionhost --user app_admin --zone "placeholder"
Copied!
The following gcloud command creates a new user called app-admin and resets the password for a host called vm-securehost located in the placeholder zone:
gcloud compute reset-windows-password vm-securehost --user app_admin --zone "placeholder"
Copied!
Alternatively, you can force a password reset from the Compute Engine console. You will have to repeat this for the second host as the login credentials for that instance will be different.

## Task 3. Connect to the secure host and configure Internet Information Server
To connect to the secure host, you have to RDP into the bastion host first. A Windows Compute Instance with an external address can be connected to via RDP using the RDP button that appears next to Windows Compute instances in the Compute Instance summary page.

Once you are connected to the bastion host using RDP session then open a new RDP session inside the bastion host to connect to the internal private network address of the secure host.

When connected to a Windows server, you can launch the Microsoft RDP client using the command mstsc.exe, or you can search for Remote Desktop Manager from the Start menu. This will allow you to connect from the bastion host to other compute instances on the same VPC even if those instances do not have a direct internet connection themselves.

Once you connect to the vm-securehost machine through RDP then configure Internet Information Server.

Once you log in to the vm-securehost machine, Open the Server Management window. And Configure the local server to Add roles and features.

Use the Role-based or feature-based installation to add the Web Server (IIS) role.

Click Check my progress to verify the objective.
Configure the IIS web server software.

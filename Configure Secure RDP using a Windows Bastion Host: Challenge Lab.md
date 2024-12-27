# Your challenge
Deploy the secure Windows machine that is not configured for external communication inside a new VPC subnet, then deploy the Microsoft Internet Information Server on that secure machine. For the purposes of this lab, all resources should be provisioned in the following region and zone:

- Region: region
- Zone: zone

```
gcloud auth list
gcloud config list project

export ZONE=zone
echo ${ZONE}

REGION="${ZONE%-*}"
echo ${REGION}

```

### Tasks
The key tasks are listed below. Good luck!

- Create a new VPC network with a single subnet.
- Create a firewall rule that allows external RDP traffic to the bastion host system.
- Deploy two Windows servers that are connected to both the VPC network and the default network.
- Create a virtual machine that points to the startup script.
- Configure a firewall rule to allow HTTP access to the virtual machine.


## Task 1. Create the VPC network

1. Create a new VPC network called `securenetwork`.

2. Create a new VPC subnet inside `securenetwork` in the `region` region.

3. Once the network and subnet have been configured, configure a firewall rule that allows inbound RDP traffic (TCP port 3389) from the internet to the bastion host. This rule should be applied to the appropriate host using network tags.

```
gcloud compute networks create securenetwork --project=$DEVSHELL_PROJECT_ID --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional

gcloud compute networks subnets create secure-subnet --project=$DEVSHELL_PROJECT_ID --range=10.0.0.0/24 --stack-type=IPV4_ONLY --network=securenetwork --region=$REGION

gcloud compute --project=$DEVSHELL_PROJECT_ID firewall-rules create secure-firewall --direction=INGRESS --priority=1000 --network=securenetwork --action=ALLOW --rules=tcp:3389 --source-ranges=0.0.0.0/0 --target-tags=rdp

gcloud compute firewall-rules create <FIREWALL_NAME> --network securenetwork --allow tcp,udp,icmp --source-ranges <IP_RANGE>
gcloud compute firewall-rules create <FIREWALL_NAME> --network securenetwork --allow tcp:22,tcp:3389,icmp

```

## Task 2. Deploy your Windows instances and configure user passwords

1. Deploy a Windows 2016 server (Server with Desktop Experience) instance called vm-securehost with two network interfaces in the `zone` zone.

- Configure the first network interface with an internal only connection to the newly created VPC subnet.

- The second network interface with an internal only connection to the default VPC network. This is the secure server.

```
gcloud compute instances create vm-securehost --project=$DEVSHELL_PROJECT_ID --zone=$ZONE --machine-type=e2-small --network-interface=stack-type=IPV4_ONLY,subnet=secure-subnet,no-address --network-interface=stack-type=IPV4_ONLY,subnet=default,no-address --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --tags=rdp --create-disk=auto-delete=yes,boot=yes,device-name=vm-securehost,image=projects/windows-cloud/global/images/windows-server-2016-dc-v20230510,mode=rw,size=150,type=projects/$DEVSHELL_PROJECT_ID/zones/$ZONE/diskTypes/pd-standard --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any
```

2. Deploy a second Windows 2016 server (Server with Desktop Experience) instance called vm-bastionhost with two network interfaces in the zone zone.

- Configure the first network interface to connect to the newly created VPC subnet with an ephemeral public (external NAT) address.

- The second network interface with an internal only connection to the default VPC network. This is the jump box or bastion host.

```
gcloud compute instances create vm-bastionhost --project=$DEVSHELL_PROJECT_ID --zone=$ZONE --machine-type=e2-small --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=secure-subnet --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --tags=rdp --create-disk=auto-delete=yes,boot=yes,device-name=vm-securehost,image=projects/windows-cloud/global/images/windows-server-2016-dc-v20230809,mode=rw,size=150,type=projects/$DEVSHELL_PROJECT_ID/zones/$ZONE/diskTypes/pd-standard --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any
```

### Configure user passwords

1. After your Windows instances have been created, create a user account and reset the Windows passwords in order to connect to each instance.

NOTE: Copy the User name and Password of both instances for later use.

2. The following gcloud command creates a new user called app-admin and resets the password for a host called vm-bastionhost located in the placeholder zone:

```
gcloud compute reset-windows-password vm-bastionhost --user app_admin --zone $ZONE
```

3. The following gcloud command creates a new user called app-admin and resets the password for a host called vm-securehost located in the placeholder zone:
```
gcloud compute reset-windows-password vm-securehost --user app_admin --zone $ZONE
```
- Alternatively, you can force a password reset from the Compute Engine console. You will have to repeat this for the second host as the login credentials for that instance will be different.


## Task 3. Connect to the secure host and configure Internet Information Server
1. To connect to the secure host, you have to RDP into the bastion host first. A Windows Compute Instance with an external address can be connected to via RDP using the RDP button that appears next to Windows Compute instances in the Compute Instance summary page.

2. Once you are connected to the bastion host using RDP session then open a new RDP session inside the bastion host to connect to the internal private network address of the secure host.

3. When connected to a Windows server, you can launch the Microsoft RDP client using the command mstsc.exe, or you can search for Remote Desktop Manager from the Start menu. This will allow you to connect from the bastion host to other compute instances on the same VPC even if those instances do not have a direct internet connection themselves.

Once you connect to the vm-securehost machine through RDP then configure Internet Information Server.

4. Once you log in to the vm-securehost machine, Open the Server Management window. And Configure the local server to Add roles and features.

5. Use the Role-based or feature-based installation to add the Web Server (IIS) role.



Troubleshooting

- Unable to connect to the Bastion host: Make sure you are attempting to connect to the external address of the bastion host. If the address is correct you may not be able to connect to the bastion host if the firewall rule is not correctly configured to allow TCP port 3389 (RDP) traffic from the internet, or your own system's public IP-address, to the network interface on the bastion host that has an external address. Finally, you might have issues connecting via RDP if your own network does not allow access to internet addresses via RDP. If everything else is definitely OK, you will need to talk to the owner of the network you are connected to the internet with to open up port 3389 or connect using a different network.
- Unable to connect to the Secure Host from the Bastion host: If you can successfully connect to the bastion host but are unable to make the internal RDP connection using Microsoft Remote Desktop Connection application, check that both instances are connected to the same VPC network.

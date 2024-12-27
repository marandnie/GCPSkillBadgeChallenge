# Challenge scenario
Your company is ready to launch a brand new product! Because you are entering a totally new space, you have decided to deploy a new website as part of the product launch. The new site is complete, but the person who built the new site left the company before they could deploy it.

## Your challenge
Your challenge is to deploy the site in the public cloud by completing the tasks below. You will use a simple Apache web server as a placeholder for the new site in this exercise. Good luck!

## Running a basic Apache web server
A virtual machine instance on Compute Engine can be controlled like any standard Linux server. Deploy a simple Apache web server (a placeholder for the new product site) to learn the basics of running a server on a virtual machine instance.

```
gcloud auth list
gcloud config list project

export ZONE=zone
echo ${ZONE}

REGION="${ZONE%-*}"
echo ${REGION}

export INSTANCE_NAME=
echo ${INSTANCE_NAME}

```

## Task 1. Create a Linux VM instance
Create a Linux virtual machine, name it Instance name and specify the zone as Compute zone.

```
gcloud compute firewall-rules create http-ingress --allow=tcp:80 --source-ranges 0.0.0.0/0 --target-tags http-server --network default

gcloud compute instances create $INSTANCE_NAME --zone=$ZONE --machine-type=e2-medium --tags=http-server,https-server --image=projects/debian-cloud/global/images/debian-10-buster-v20220406 --metadata=startup-script=\#\!\ /bin/bash$'\n'apt-get\ update$'\n'apt-get\ install\ apache2\ -y$'\n'service\ --status-all$'\n'

gcloud compute instances create $INSTANCE_NAME --project=$DEVSHELL_PROJECT_ID --zone=$ZONE --machine-type=e2-medium --tags=http-server,https-server --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=867917564241-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=instance-20241227-183615,image=projects/debian-cloud/global/images/debian-12-bookworm-v20241210,mode=rw,size=10,type=pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any
```

## Task 2. Enable public access to VM instance
While creating the Linux instance, make sure to apply the appropriate firewall rules so that potential customers can find your new product.

## Task 3. Running a basic Apache Web Server
A virtual machine instance on Compute Engine can be controlled like any standard Linux server.

- Deploy a simple Apache web server (a placeholder for the new product site) to learn the basics of running a server on a virtual machine instance.

```
gcloud compute ssh $INSTANCE_NAME
sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl status apache2

gcloud compute instances describe $INSTANCE_NAME --format='get(networkInterfaces[0].accessConfigs[0].natIP)'

```

## Task 4. Test your server
- Test that your instance is serving traffic on its external IP.
You should see the "Hello World!" page (a placeholder for the new product site).


Troubleshooting
- Receiving a Connection Refused error:
- Your VM instance is not publicly accessible because the VM instance does not have the proper tag that allows Compute Engine to apply the appropriate firewall rules, or your project does not have a firewall rule that allows traffic to your instance's external IP address.
- You are trying to access the VM using an https address. Check that your URL is http:// EXTERNAL_IP and not https:// EXTERNAL_IP.

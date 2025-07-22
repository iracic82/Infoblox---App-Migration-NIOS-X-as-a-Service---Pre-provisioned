---
slug: nios-xaas-and-dtc
id: 9pf2ea1ikpak
type: challenge
title: Configure NIOS-XaaS, Deploy Application in AWS and Perform Migration
teaser: Configure NIOS-XaaS authoritative DNS, deploy the cloud application in AWS
  using Terraform + IPAM, and set up DNS Traffic Control to shift traffic from on-prem
  to the cloud.
notes:
- type: video
  url: https://videos.infoblox.com/watch/TYegXjnJ2ocSZZyyaVAqTo?
tabs:
- id: 4sufxvwpvbku
  title: Shell
  type: terminal
  hostname: shell
- id: 9gudn6m9yz7o
  title: Editor
  type: code
  hostname: shell
  path: /root/infoblox-lab
- id: cx0uzoqrjyga
  title: AWS Console
  type: browser
  hostname: aws-console
- id: yqmrz88ehgnk
  title: Infoblox Portal
  type: browser
  hostname: infoblox
- id: fkhndw1fyydo
  title: Windows Client
  type: service
  hostname: rdpclient
  path: /#/client/c/WindowsClient?username=instruqt&password=Passw0rd!
  port: 8080
- id: zemexgoavf5c
  title: Lab Diagram
  type: browser
  hostname: lab
difficulty: ""
timelimit: 0
lab_config:
  default_layout_sidebar_size: 0
enhanced_loading: null
---

# 🔍 Overview

You’ve confirmed the on-premises app is running and DNS is in place. Now it’s time to:
- Extend authoritative DNS into AWS using Infoblox NIOS-XaaS — a SaaS solution with zero infrastructure to manage
- Deploy the cloud application using Terraform and IPAM automation,
- Use DNS Traffic Control (DTC) to migrate traffic without breaking anything.

This is the full cloud migration experience — from deployment to DNS-based traffic steering — delivered with policy alignment, automation, and zero infrastructure baggage.


⸻

# 📘 Use Case Story: Zero-Touch Cloud Expansion

Alex is ready. Terraform configs are prepped, and IPAM integration is dialed in. Now, the final steps are:
- Deploy DNS services using NIOS-X-as-a-Service — fully managed, no infrastructure required
- Launch the app in AWS with automated IP allocation and authoritative DNS
- Use DNS Traffic Control (DTC) to gradually and safely steer user traffic to the cloud

No downtime. No broken links. Full visibility. Just smooth migration with SaaS-native DNS at the core.

⸻

# ⚙️ What You’ll Do in This Step

In this challenge, you’ll:
- 🔹 Step 1: Configure NIOS-X-as-a-Service to act as the authoritative DNS (no infra to manage)
- 🔹 Step 2: Deploy the application in AWS using Terraform + IPAM automation
- 🔹 Step 3: Set up DNS Traffic Control (DTC) and perform a safe, controlled traffic shift

All with SaaS simplicity, zero disruption, and full DNS observability.

⸻
# 💼 Business Value

This step demonstrates the full-stack migration workflow, powered by Infoblox Universal DDI and NIOS-X-as-a-Service:
- ✅ Authoritative DNS deployed instantly via SaaS — no infrastructure to manage
- ✅ IPAM-integrated app provisioning using Terraform for consistency and speed
- ✅ Intelligent traffic steering with DTC for seamless, policy-driven cutovers
- ✅ End-to-end visibility and control that follows the workload across environments

You’re not just shifting workloads — you’re evolving how apps are delivered, governed, and secured in the cloud era.

🔹 Step 1: Configure NIOS-XaaS for Authoritative DNS
==

🎯 Goal

Automate the full setup of NIOS-X-as-a-Service as your authoritative DNS and security platform for cloud workloads — including VPN connectivity to Infoblox’s regional PoP and enabling DNS and route propagation.

You’re not clicking through GUIs — you’re treating infrastructure like code and DNS like a cloud-native service.

🛠 Actions

1️⃣ Launch and Validate NIOS-XaaS Instance Health

Run the health-check script to validate that your NIOS-XaaS instance is provisioned and operational:

```run
cd  /root/infoblox-lab/app-migration-lab/terraform/scripts/
python3 infoblox_vpn_configure.py
```

![Screenshot 2025-07-22 at 11.08.24.png](https://play.instruqt.com/assets/tracks/prsxwy2uwmxg/8a3bfd79d66e27194e84213a89d26a41/assets/Screenshot%202025-07-22%20at%2011.08.24.png)


> [!IMPORTANT]
> ⚠️ Important Before Proceeding

Before running Step 2, wait ~5 minutes for Service IPs for the Infoblox PoP to become available.

You can confirm this in the Infoblox Portal UI under:
Configure → Service Deployment → As-A-Service → Cloud → AWS → [Region] → SITE

![Screenshot 2025-07-22 at 11.24.37.png](https://play.instruqt.com/assets/tracks/prsxwy2uwmxg/e2384910ce5ad0ff6d550f7e2a845491/assets/Screenshot%202025-07-22%20at%2011.24.37.png)


Look for the Cloud Service IP, when those show up please proceed with the Step2.


2️⃣ Discover Required Cloud Service Details

```run
cd  /root/infoblox-lab/app-migration-lab/terraform/scripts/
python3 get_cnames.py
```

![Screenshot 2025-07-22 at 11.09.22.png](https://play.instruqt.com/assets/tracks/prsxwy2uwmxg/cd08df0874f76aa3df4beeb8ab793b23/assets/Screenshot%202025-07-22%20at%2011.09.22.png)

📄 Fetches and saves Servcie IPs  (e.g., used later for VPN or DNS assignment).

3️⃣ Create the Cloud VPN to Infoblox PoP

```run
cd  /root/infoblox-lab/app-migration-lab/terraform/scripts/
python3 create_aws_vpn.py
```

![Screenshot 2025-07-22 at 11.09.31.png](https://play.instruqt.com/assets/tracks/prsxwy2uwmxg/63a69133f4fe9b7a28c70765a6324f08/assets/Screenshot%202025-07-22%20at%2011.09.31.png)

🌍 Automatically creates two VPN tunnels to the Infoblox SaaS PoP using CGW+VGW configuration:
- Creates CGWs (Customer Gateways)
- Creates VPN Connections for both tunnels


4️⃣ Extract Tunnel IPs

```run
cd  /root/infoblox-lab/app-migration-lab/terraform/scripts/
python3 extract_tunnels.py
```


![Screenshot 2025-07-22 at 11.09.40.png](https://play.instruqt.com/assets/tracks/prsxwy2uwmxg/8d14a8d8e3b464b05d4d0708b70a9c5c/assets/Screenshot%202025-07-22%20at%2011.09.40.png)

📥 Saves all IPs (Tunnel 1 & 2 per VPN) for later configuration with Infoblox UDDI or routing.


5️⃣ Update UDDI with the Tunnel IP

```run
cd  /root/infoblox-lab/app-migration-lab/terraform/scripts/
python3 extract_tunnels.py
```
![Screenshot 2025-07-22 at 11.09.48.png](https://play.instruqt.com/assets/tracks/prsxwy2uwmxg/e2b972eb690af7fdb6da14cac7ee6f15/assets/Screenshot%202025-07-22%20at%2011.09.48.png)


🚀 Updates the UDDI config to point to the correct Tunnel IPs


6️⃣ Enable Route Propagation in VPC

```run
cd  /root/infoblox-lab/app-migration-lab/terraform/scripts/
python3 enable_propagation_vpc.py
```

![Screenshot 2025-07-22 at 11.09.54.png](https://play.instruqt.com/assets/tracks/prsxwy2uwmxg/1276598291149487b0d392c58d03b571/assets/Screenshot%202025-07-22%20at%2011.09.54.png)


📡 Ensures AWS route tables are updated with the VGW for traffic routing — essential for DNS and security inspection to flow via Infoblox.


✅ What You Now Have
- 🟢 A running NIOS-X-as-a-Service instance
- 🛡️ Connected securely to Infoblox’s regional PoP
- 🌐 Route propagation enabled for your VPC
- 📘 Ready to enable DNS, Security, Application Visibility, or DTC policies — all SaaS-native

# 🔹 Step 2: Activate NIOS-XaaS Service for your current infrastructure
==

🧭 Goal

Configure NIOS-XaaS as the authoritative DNS for your private zone (infolab.com) and delegate traffic control away from Route 53 — enabling unified DNS governance from Infoblox.

🛠️ Steps to Assign Authority to NIOS-XaaS


🔹 Step 1: Navigate to the DNS Zone
1.	Go to the Infoblox CSP Portal
2.	Navigate to:

```
Configure → Networking → DNS → Views → default
```

![Screenshot 2025-07-22 at 11.36.39.png](https://play.instruqt.com/assets/tracks/prsxwy2uwmxg/e848f2e6f82ccd61cbd4771d365a3256/assets/Screenshot%202025-07-22%20at%2011.36.39.png)

3.	Find your zone: infolab.com

![Screenshot 2025-07-22 at 11.36.46.png](https://play.instruqt.com/assets/tracks/prsxwy2uwmxg/c493c33f7066adc4d6d395ef7f7ff309/assets/Screenshot%202025-07-22%20at%2011.36.46.png)

🔹 Step 2: Edit the Zone and Assign Authoritative Server

1.	Click on the infolab.com zone → Edit
2.	Scroll down to the section: Authoritative DNS Servers

![Screenshot 2025-07-22 at 11.36.58.png](https://play.instruqt.com/assets/tracks/prsxwy2uwmxg/0f31b0aaa09cedc4afe2200057d92bcb/assets/Screenshot%202025-07-22%20at%2011.36.58.png)

4.	In the Available Service Instances column, locate your NIOS-XaaS instance
- It will be listed under Type: NIOS-X (BloxOne DDI)
5.	Move it to the right-hand column to assign it

✅ This action declares that this Infoblox SaaS instance is now serving the zone as an authoritative DNS responder


🔹 Step 3: Save and Confirm
1.	Click Save & Close
2.	Back in the zone list, you’ll now see:

![Screenshot 2025-07-22 at 11.37.09.png](https://play.instruqt.com/assets/tracks/prsxwy2uwmxg/ee9aa1881227bf69928dcf155e59c559/assets/Screenshot%202025-07-22%20at%2011.37.09.png)


- DNS Provider: BloxOne DDI
- External Provider Zone Type: Private
- DNS View: default
- Access View: Unassigned


🔹 Step 3: Deploy App in AWS via Terraform + IPAM
==

Now it’s time to deploy the application stack to the new AWS environment, using IPAM automation from Infoblox Universal DDI.

Alright, here’s where the magic kicks in.

Alex and Sam already did the heavy lifting by carving out the right Federated Realms and Address Blocks mapped strictly to AWS — no IP overlaps, no policy confusion, just clean, cloud-native boundaries.

And because they’re not into click-ops, everything’s wired up with Terraform + Universal DDI. So yeah — no IP spreadsheets, no DNS tickets, no hand-holding.

Your job? Just trigger the Terraform stack and let Universal DDI take care of:
- Subnet selection ✅
- Next-available IP assignment ✅
- EC2 provisioning ✅
- DNS registration ✅

```run
cd  /root/infoblox-lab/app-migration-niosx/terraform/app-lab/
terraform init
```

Lets run apply after initilization of the providers.

```run
cd  /root/infoblox-lab/app-migration-niosx/terraform/app-lab/
terraform apply --auto-approve
```


We’re deploying the same app stack from on-prem into AWS, but the right way — no clunky lift-and-shift, no IP collisions, and none of the drama that comes with live VM moves.

This is that active/active swim lane model — the real MVP of app migrations.

🚫 We’re not faking IPs.
🚫 We’re not forcing legacy constraints.
✅ We’re replicating the app with clean policy and full DNS visibility.

(Yeah, storage replication is still a thing — but that’s for another lab.)


# ✅ Validation

Once your A record is created and pointing to your deployed app in AWS, the final test is from the Windows client. You’ll manually configure the DNS settings to use Infoblox and then test the app’s URL.

⸻

🎯 Goal
- Manually set the DNS server on your Windows client to 10.10.10.3 (your NIOS-XaaS Service IP)
- Resolve and access app-cloud.infolab.com
- Verify the app responds from the new cloud location


# 	🛠 Instructions

✅ Step 1: Confirm Network Connection

![Screenshot 2025-07-21 at 11.17.11.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/02aa49cccb302077e21c4a04c9e6ad80/assets/Screenshot%202025-07-21%20at%2011.17.11.png)

Make sure your Windows instance shows it’s connected to the Internet.

✅ Step 2: Go to Network Settings

![Screenshot 2025-07-21 at 11.17.28.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/c428d5d9f458db3bdd1823709df7b321/assets/Screenshot%202025-07-21%20at%2011.17.28.png)

Under Status, click Change adapter options.


✅ Step 3: Modify Adapter Settings

![Screenshot 2025-07-21 at 11.17.45.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/88ea44b390b1766e3895f2f6f0c30390/assets/Screenshot%202025-07-21%20at%2011.17.45.png)


1.	Right-click on your active Ethernet adapter (e.g., Ethernet 3)
2.	Select Properties


✅ Step 4: Set Infoblox DNS IP

![Screenshot 2025-07-21 at 11.17.55.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/79a0c8af809d465909b5b099d4e1cfa2/assets/Screenshot%202025-07-21%20at%2011.17.55.png)

![Screenshot 2025-07-21 at 11.18.05.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/87baeba773a88b8416abb0d395171426/assets/Screenshot%202025-07-21%20at%2011.18.05.png)


1. Select Internet Protocol Version 4 (TCP/IPv4) → click Properties
2. Choose:
✅ Obtain IP address automatically
✅ Use the following DNS server address
Enter:
- Preferred DNS: 10.10.10.3

> [!IMPORTANT]
> DNS Server IP: 10.10.10.3

- Leave alternate empty
- Click OK, then Close

✅ Step 5: Launch the Browser & Access App


Open Microsoft Edge or any browser and go to:

```
http://app-cloud.infolab.com:5080
```
![Screenshot 2025-07-21 at 11.18.20.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/f366be302b2e460c4bdd72a6e0e5e2e7/assets/Screenshot%202025-07-21%20at%2011.18.20.png)
![Screenshot 2025-07-21 at 11.19.39.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/bed5b7bd544ef5e5e3dd8457ffaa3b02/assets/Screenshot%202025-07-21%20at%2011.19.39.png)


You should see the “Hi folks!” portal confirming:

- Hostname: new-location
- IP Address: 10.100.0.10

> [!NOTE]
> 🎉 Boom — this confirms DNS is resolving via Infoblox and the app is running from the AWS deployment!

🔹 Step 4: Configure DTC and Perform Traffic Shift
==

🎯 Goal

Create a DTC pool with both app endpoints and gradually shift traffic from the on-prem IIS app to the AWS-deployed instance, ensuring no user disruption.


🛠 Actions

### ✅ Step 1: Access the DNS Traffic Control Section

From the Infoblox UI:
	•	Navigate to DTC : Configure→ Networking →DNS → DTC

![Screenshot 2025-07-21 at 11.39.55.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/aeb3121c24af8fb6f3097cc74648c11b/assets/Screenshot%202025-07-21%20at%2011.39.55.png)


### ✅ Step 2: Create a DTC Servers

1.	Click Create DTC Server

![Screenshot 2025-07-21 at 11.42.29.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/dc3e0ec297cbad3a643f973d9f44ff14/assets/Screenshot%202025-07-21%20at%2011.42.29.png)

2.	Click on “Create DTC Server,” assign a name, and leave the remaining configuration settings as shown in the screenshot below.

![Screenshot 2025-07-21 at 11.43.23.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/e5a978ed898d50251a46d935c247dfe5/assets/Screenshot%202025-07-21%20at%2011.43.23.png)

3. Clikc "Save&Close"

4. Repeat the previous steps ( 1. - 3. )  to create another DTC Server.
Just make sure to assign a new name and keep the rest of the configuration consistent with image below.

![Screenshot 2025-07-21 at 11.45.21.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/b55a6e250792b9913eb3981f1e162011/assets/Screenshot%202025-07-21%20at%2011.45.21.png)


![Screenshot 2025-07-21 at 11.46.31.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/8261cdaa2c65aa5e025799c362257b94/assets/Screenshot%202025-07-21%20at%2011.46.31.png)


### ✅ Step 3: Create a DTC Pool

Select DTC Pools under DTC option

![Screenshot 2025-07-21 at 11.48.09.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/9d9dc986b77411cc913c774f882126a3/assets/Screenshot%202025-07-21%20at%2011.48.09.png)

1.	Click Create Pool
2.	Give it a name
```
Lab
```
3.	Set the Load Balancing Method to **Ratio**

![Screenshot 2025-07-21 at 11.50.03.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/8e7fe8e9375d625bbb7435b220759689/assets/Screenshot%202025-07-21%20at%2011.50.03.png)

4.	Under Servers, click Add Server twice:
- Member 1: Srv1 (e.g., 10.0.0.10) - weight 100
- Member 2: Srv2 (e.g., 10.100.0.10) - weight 1

👉 This means: almost all DNS responses will return the IP of Srv1 ( which is on-prem application ) from this pool — unless Srv1 is down or fails health checks.

![Screenshot 2025-07-21 at 12.31.15.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/6fbb0868aba54e8a66eaf3c646ce31c1/assets/Screenshot%202025-07-21%20at%2012.31.15.png)


5. Click Next
6. Select HTTP health checks and move it to the right

![Screenshot 2025-07-21 at 11.54.13.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/f11e268c3f616ce8a1410c51df6a4df4/assets/Screenshot%202025-07-21%20at%2011.54.13.png)

7. Click Next
8. Click Save&Close

### ✅ Step 4: Add a Health Check  ( EDIT)
1.	Go to DTC → DTC Health Checks
2.	Edit the existing HTTP check

![Screenshot 2025-07-21 at 11.56.55.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/38d017b42991036a7b5b343d3be87e9a/assets/Screenshot%202025-07-21%20at%2011.56.55.png)

In the health check settings, change the port to 5080 instead of the default 80, as shown in the image below.

![Screenshot 2025-07-21 at 11.58.52.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/1ed8f4dead3fdbb91b3fd98650d6421f/assets/Screenshot%202025-07-21%20at%2011.58.52.png)

3.Click Save&Close


### ✅ Step 5: Create DTC Policy
1.	Go to DTC → DTC Polices
2.	Click Create DTC Policy

![Screenshot 2025-07-21 at 12.01.46.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/1a5f4a5cff9bdfbe29bf0af1da6a5e32/assets/Screenshot%202025-07-21%20at%2012.01.46.png)

3. Give it the name Lab and keep the settings as you see below




![Screenshot 2025-07-21 at 12.02.35.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/b7b96ba71eaefc138f9bb34600e2bcd4/assets/Screenshot%202025-07-21%20at%2012.02.35.png)

⸻

🧠 Understanding Ratio-Based Load Balancing

Ratio-based load balancing allows precise control over how DNS traffic is distributed across DTC pools or members, using numeric weights. Unlike round-robin, this method doesn’t evenly rotate — it routes traffic proportionally based on the weight assigned.

✅ In Our Lab Scenario
- We’ve created one DTC Pool with a total weight of 100
- This pool contains two backend members:
- 🏠 On-prem instance
- ☁️ AWS-deployed app

Inside this pool, we’re leveraging the DTC Pool’s internal ratio mechanism to gradually shift traffic between the two servers during migration

⸻

4. Click Next
5. Select the appropriate pool that we created in the one of previous steps and keep the settins as you see below

![Screenshot 2025-07-21 at 12.03.54.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/0fcfd48e709529e40de6985f0d841222/assets/Screenshot%202025-07-21%20at%2012.03.54.png)

6. Click Next
7. Click Save&Close


### ✅ Step 6: Create DTC LBDN

1.	Go to DTC → LBDNs

![Screenshot 2025-07-21 at 12.04.54.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/76233fe185c1ac634d20a2b9ae26d9ee/assets/Screenshot%202025-07-21%20at%2012.04.54.png)

2. Click Create LBDN and keep the settings as you see on the picture below

![Screenshot 2025-07-21 at 12.37.13.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/332a10ae25666731ffb52b479757d580/assets/Screenshot%202025-07-21%20at%2012.37.13.png)

3. Click Save&Close



## 🧪 Step: Validate Legacy Application & Prepare for Traffic Shift

With the DTC configuration complete, we’ve initially set 100% of traffic to go to the on-prem legacy application (old IIS server).

✅ What to Do
1.	Open the App in the Client Browser
- In the left-hand Instruqt window, which runs your Windows Client, open Internet Explorer (or Edge if available).
- Go to:

```
http://app.infolab.com:5080
```

![Screenshot 2025-07-21 at 12.14.48.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/bb9b4ef228198068c041d93a52cb63de/assets/Screenshot%202025-07-21%20at%2012.14.48.png)



> [!NOTE]
> You should see the old version of the app.


2.	Run DNS Resolution Ratio Script:

- On the same Windows client, open PowerShell.

![Screenshot 2025-07-21 at 13.10.52.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/35382c4ea2e9ddc9489c6520d9a8f4c1/assets/Screenshot%202025-07-21%20at%2013.10.52.png)

> [!IMPORTANT]
> NOTE:Go to Dowloads folder.

- Run the following script to simulate 50 DNS queries:

```
./test.ps1
```

This will give you a summary of how many DNS responses point to each backend IP.
Example output:

![Screenshot 2025-07-21 at 12.52.16.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/02f4abe15c82c3d5b21ac843fd293225/assets/Screenshot%202025-07-21%20at%2012.52.16.png)


> [!IMPORTANT]
> ✅ This confirms all traffic is going to the legacy server (as expected).


🧠 What’s Next?

We’re ready to shift traffic gradually to the new app hosted in AWS. But to save time in this lab, we’ll go straight to a 50/50 DTC weight ratio and observe how the resolution splits between both endpoints.


## 🔁 Step: Shift Traffic to 50/50


Now that we validated all traffic is hitting the on-prem app, it’s time to update the DTC server weights and start splitting the traffic.

⚙️ Actions:
1.	Go to the DTC Pool.
2.	Edit the server weights:
- Srv1 (on-prem): 50
- Srv2 (AWS): 50
3.	Save the changes.


✅ Validate 50/50 Split

1. Rerun the script( be in the same folder **Dowloads** in the powershell cmd and always flush dns caching values ):

```
ipconfig /flushdns
```

```
./test.ps1
```

2.	You should now see a roughly even distribution between both IPs:
Example output:

![Screenshot 2025-07-21 at 12.53.18.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/ac8c381e68f1c32af0a0e2d4b3e62127/assets/Screenshot%202025-07-21%20at%2012.53.18.png)


## ✅ Step: Complete Migration to New App (100% Cloud)


Once you’re happy with the test, push the full cutover:

Actions:
1.	Go back to the DTC Pool.
2.	Set weights:
- Srv1 (on-prem): 1
- Srv2 (AWS): 100
3.	Rerun the script after flushing the dns cash:

```
ipconfig /flushdns
```

```
./test.ps1
```

You should now see:

![Screenshot 2025-07-21 at 12.54.21.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/257f33bad8b9508e84d50037915cc637/assets/Screenshot%202025-07-21%20at%2012.54.21.png)

4.	After closing the browser and deleting the browser cash - reopen the browser at http://app.infolab.com:5080 — this time, you should only see the new application UI.


![Screenshot 2025-07-21 at 12.18.44.png](https://play.instruqt.com/assets/tracks/mvbwpzb5c9dc/27497bd80a91399f6ddc36db9c60885b/assets/Screenshot%202025-07-21%20at%2012.18.44.png)




💡 Mission accomplished — DNS-based traffic migration complete using Infoblox DTC.
With NIOS-X-as-a-Service, you achieved seamless traffic shift without managing a single server. The entire experience was powered by Infoblox’s SaaS platform — delivering authoritative DNS, traffic control, and policy enforcement with zero infrastructure overhead and built-in scalability.







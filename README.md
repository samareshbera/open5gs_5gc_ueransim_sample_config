# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration
UERANSIM (5G UE & RAN (gNodeB) implementation) supports IPv4 of PDU Session Type from 2020.11.17 version, and the Data Plane facility has been enabled.
Therefore, in order to use U-Plane's DN (Data Network) as a trial, I built a simulation environment for the 5GC mobile network.
This briefly describes the overall and configuration files.

---

<h2 id="conf_list">List of Sample Configurations</h2>

1. [One SGW-C/PGW-C, Multiple SGW-Us/PGW-Us and APNs](https://github.com/s5uishida/open5gs_epc_oai_sample_config)
2. One SMF, Multiple UPFs and DNNs (this article)
3. [Select nearby UPF according to the connected gNodeB](https://github.com/s5uishida/open5gs_5gc_ueransim_nearby_upf_sample_config)
4. [Select UPF based on S-NSSAI](https://github.com/s5uishida/open5gs_5gc_ueransim_snssai_upf_sample_config)
5. [SCP Indirect communication Model C](https://github.com/s5uishida/open5gs_5gc_ueransim_scp_model_c_sample_config)
6. [VoLTE and SMS Configuration for docker_open5gs](https://github.com/s5uishida/docker_open5gs_volte_sms_config)
7. [Monitoring Metrics with Prometheus](https://github.com/s5uishida/open5gs_5gc_ueransim_metrics_sample_config)

---

<h2 id="toc">Table of Contents</h2>

- [Overview of Open5GS 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN](#changes)
  - [Changes in configuration files of Open5GS 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of Open5GS 5GC U-Plane1](#changes_up1)
  - [Changes in configuration files of Open5GS 5GC U-Plane2](#changes_up2)
  - [Changes in configuration files of UERANSIM UE / RAN](#changes_ueransim)
    - [Changes in configuration files of RAN](#changes_ran)
    - [Changes in configuration files of UE0 (IMSI-001010000000000)](#changes_ue0)
    - [Changes in configuration files of UE1 (IMSI-001010000000001)](#changes_ue1)
    - [Changes in configuration files of UE2 (IMSI-001010000000002)](#changes_ue2)
    - [Changes in configuration files of UE3 (IMSI-001010000000003)](#changes_ue3)
    - [Changes in configuration files of UE4 (IMSI-001010000000004)](#changes_ue4)
- [Network settings of Open5GS 5GC and UERANSIM UE / RAN](#network_settings)
  - [Network settings of Open5GS 5GC U-Plane1](#network_settings_up1)
  - [Network settings of Open5GS 5GC U-Plane2](#network_settings_up2)
- [Build Open5GS and UERANSIM](#build)
- [Run Open5GS 5GC and UERANSIM UE / RAN](#run)
  - [Run Open5GS 5GC C-Plane](#run_cp)
  - [Run Open5GS 5GC U-Plane1 & U-Plane2](#run_up)
  - [Run UERANSIM](#run_ueran)
    - [Start gNB](#start_gnb)
    - [Start UE (UE0)](#start_ue)
- [Ping google.com](#ping)
  - [Case for going through DN 10.45.0.0/16](#ping_1)
- [Changelog (summary)](#changelog)

---
<h2 id="overview">Overview of Open5GS 5GC Simulation Mobile Network</h2>

I created a 5GC mobile network (Internet reachable) for simulation with the aim of creating an environment in which packets can be sent end-to-end with different DNs for each DNN.

The following minimum configuration was set as a condition.
- C-Plane have multiple U-Planes.
- U-Plane have multiple DNs.
- Multiple UEs connect to same DN.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - Open5GS v2.4.7 - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM2 | Open5GS 5GC U-Plane1  | 192.168.0.112/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM3 | Open5GS 5GC U-Plane2  | 192.168.0.113/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM4 | UERANSIM RAN (gNodeB) | 192.168.0.131/24 | Ubuntu 20.04 | 1GB | 10GB |
| VM5 | UERANSIM UE | 192.168.0.132/24 | Ubuntu 20.04 | 1GB | 10GB |

Subscriber Information (other information is the same) is as follows.  
**Note. Please select OP or OPc according to the setting of UERANSIM UE configuration files.**
| UE # | IMSI | DNN | OP/OPc |
| --- | --- | --- | --- |
| UE0 | 001010000000000 | internet | OPc |
| UE1 | 001010000000001 | internet2 | OPc |
| UE2 | 001010000000002 | internet2 | OPc |
| UE3 | 001010000000003 | ims | OPc |
| UE4 | 001010000000004 | ims | OPc |

I registered these information with the Open5GS WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

Each DNs are as follows.
| DN | TUNnel interface of DN | DNN | TUNnel interface of UE | U-Plane # |
| --- | --- | --- | --- | --- |
| 10.45.0.0/16 | ogstun | internet | uesimtun0 | U-Plane1 |
| 10.46.0.0/16 | ogstun2 | internet2 | uesimtun1, uesimtun2 | U-Plane1 |
| 10.47.0.0/16 | ogstun3 | ims | uesimtun3, uesimtun4 | U-Plane2 |

Additional information.

Open5GS 5GC U-Plane worked fine on Raspberry Pi 4 Model B. I used [Ubuntu 20.04 (64bit) for Raspberry Pi 4](https://ubuntu.com/download/raspberry-pi) as the OS. I think it would be convenient to place a compact U-Plane in the edge environment and use it as an end-point for DN.

In addition, I have not confirmed the communication performance.

<h2 id="changes">Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN</h2>

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.4.7 - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM/wiki/Installation

<h3 id="changes_cp">Changes in configuration files of Open5GS 5GC C-Plane</h3>

The following parameters including DNN can be used in the logic that selects UPF as the connection destination by PFCP.

- DNN
- TAC (Tracking Area Code)
- nr_CellID

For the sake of simplicity, I used only DNN this time. Please refer to [here](https://github.com/open5gs/open5gs/pull/560#issue-483001043) for the logic to select UPF.

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2022-06-04 19:47:05.006151198 +0900
+++ amf.yaml    2022-06-05 21:34:01.935500250 +0900
@@ -229,23 +229,23 @@
       - addr: 127.0.0.5
         port: 7777
     ngap:
-      - addr: 127.0.0.5
+      - addr: 192.168.0.111
     guami:
       - plmn_id:
-          mcc: 901
-          mnc: 70
+          mcc: 001
+          mnc: 01
         amf_id:
           region: 2
           set: 1
     tai:
       - plmn_id:
-          mcc: 901
-          mnc: 70
+          mcc: 001
+          mnc: 01
         tac: 1
     plmn_support:
       - plmn_id:
-          mcc: 901
-          mnc: 70
+          mcc: 001
+          mnc: 01
         s_nssai:
           - sst: 1
     security:
```
- `open5gs/install/etc/open5gs/smf.yaml`
```diff
--- smf.yaml.orig       2022-06-04 19:47:04.977151107 +0900
+++ smf.yaml    2022-06-05 21:36:29.499567538 +0900
@@ -381,17 +381,18 @@
       - addr: 127.0.0.4
         port: 7777
     pfcp:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.111
     gtpc:
       - addr: 127.0.0.4
-      - addr: ::1
     gtpu:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.111
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
+      - addr: 10.46.0.1/16
+        dnn: internet2
+      - addr: 10.47.0.1/16
+        dnn: ims
     dns:
       - 8.8.8.8
       - 8.8.4.4
@@ -501,7 +502,10 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
+        dnn: [internet, internet2]
+      - addr: 192.168.0.113
+        dnn: ims
 
 #
 # parameter:
```

<h3 id="changes_up1">Changes in configuration files of Open5GS 5GC U-Plane1</h3>

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2022-04-05 20:55:38.304953273 +0900
+++ upf.yaml    2022-06-05 21:39:07.009058283 +0900
@@ -166,12 +166,16 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
+        dev: ogstun
+      - addr: 10.46.0.1/16
+        dnn: internet2
+        dev: ogstun2
 
 #
 # smf:
```

<h3 id="changes_up2">Changes in configuration files of Open5GS 5GC U-Plane2</h3>

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2022-04-05 20:56:57.984902990 +0900
+++ upf.yaml    2022-06-05 22:23:35.800331893 +0900
@@ -166,12 +166,13 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.113
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.113
     subnet:
-      - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+      - addr: 10.47.0.1/16
+        dnn: ims
+        dev: ogstun3
 
 #
 # smf:
```

<h3 id="changes_ueransim">Changes in configuration files of UERANSIM UE / RAN</h3>

<h4 id="changes_ran">Changes in configuration files of RAN</h4>

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2021-04-20 11:07:30.000000000 +0000
+++ open5gs-gnb.yaml    2021-08-29 11:53:34.170068022 +0000
@@ -1,17 +1,17 @@
-mcc: '901'          # Mobile Country Code value
-mnc: '70'           # Mobile Network Code value (2 or 3 digits)
+mcc: '001'          # Mobile Country Code value
+mnc: '01'           # Mobile Network Code value (2 or 3 digits)
 
 nci: '0x000000010'  # NR Cell Identity (36-bit)
 idLength: 32        # NR gNB ID length in bits [22...32]
 tac: 1              # Tracking Area Code
 
-linkIp: 127.0.0.1   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
-ngapIp: 127.0.0.1   # gNB's local IP address for N2 Interface (Usually same with local IP)
-gtpIp: 127.0.0.1    # gNB's local IP address for N3 Interface (Usually same with local IP)
+linkIp: 192.168.0.131   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
+ngapIp: 192.168.0.131   # gNB's local IP address for N2 Interface (Usually same with local IP)
+gtpIp: 192.168.0.131    # gNB's local IP address for N3 Interface (Usually same with local IP)
 
 # List of AMF address information
 amfConfigs:
-  - address: 127.0.0.5
+  - address: 192.168.0.111
     port: 38412
 
 # List of supported S-NSSAIs by this gNB
```

<h4 id="changes_ue0">Changes in configuration files of UE0 (IMSI-001010000000000)</h4>

First, copy `open5gs-ue0.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue0.yaml
```
Next, edit `open5gs-ue0.yaml`.
- `UERANSIM/config/open5gs-ue0.yaml`
```diff
--- open5gs-ue.yaml.orig        2021-09-18 21:11:52.000000000 +0900
+++ open5gs-ue0.yaml    2022-06-05 22:07:37.742307968 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-901700000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '901'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,7 +20,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
```

<h4 id="changes_ue1">Changes in configuration files of UE1 (IMSI-001010000000001)</h4>

First, copy `open5gs-ue1.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue1.yaml
```
Next, edit `open5gs-ue1.yaml`.
- `UERANSIM/config/open5gs-ue1.yaml`
```diff
--- open5gs-ue.yaml.orig        2021-09-18 21:11:52.000000000 +0900
+++ open5gs-ue1.yaml    2022-06-05 22:08:51.685605817 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-901700000000001'
+supi: 'imsi-001010000000001'
 # Mobile Country Code value of HPLMN
-mcc: '901'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,7 +20,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -39,7 +39,7 @@
 # Initial PDU sessions to be established
 sessions:
   - type: 'IPv4'
-    apn: 'internet'
+    apn: 'internet2'
     slice:
       sst: 1
 
```

<h4 id="changes_ue2">Changes in configuration files of UE2 (IMSI-001010000000002)</h4>

First, copy `open5gs-ue2.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue2.yaml
```
Next, edit `open5gs-ue2.yaml`.
- `UERANSIM/config/open5gs-ue2.yaml`
```diff
--- open5gs-ue.yaml.orig        2021-09-18 21:11:52.000000000 +0900
+++ open5gs-ue2.yaml    2022-06-05 22:09:58.470120243 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-901700000000001'
+supi: 'imsi-001010000000002'
 # Mobile Country Code value of HPLMN
-mcc: '901'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,7 +20,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -39,7 +39,7 @@
 # Initial PDU sessions to be established
 sessions:
   - type: 'IPv4'
-    apn: 'internet'
+    apn: 'internet2'
     slice:
       sst: 1
 
```

<h4 id="changes_ue3">Changes in configuration files of UE3 (IMSI-001010000000003)</h4>

First, copy `open5gs-ue3.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue3.yaml
```
Next, edit `open5gs-ue3.yaml`.
- `UERANSIM/config/open5gs-ue3.yaml`
```diff
--- open5gs-ue.yaml.orig        2021-09-18 21:11:52.000000000 +0900
+++ open5gs-ue3.yaml    2022-06-05 22:10:51.613822635 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-901700000000001'
+supi: 'imsi-001010000000003'
 # Mobile Country Code value of HPLMN
-mcc: '901'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,7 +20,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -39,7 +39,7 @@
 # Initial PDU sessions to be established
 sessions:
   - type: 'IPv4'
-    apn: 'internet'
+    apn: 'ims'
     slice:
       sst: 1
 
```

<h4 id="changes_ue4">Changes in configuration files of UE4 (IMSI-001010000000004)</h4>

First, copy `open5gs-ue4.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue4.yaml
```
Next, edit `open5gs-ue4.yaml`.
- `UERANSIM/config/open5gs-ue4.yaml`
```diff
--- open5gs-ue.yaml.orig        2021-09-18 21:11:52.000000000 +0900
+++ open5gs-ue4.yaml    2022-06-05 22:11:41.422607625 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-901700000000001'
+supi: 'imsi-001010000000004'
 # Mobile Country Code value of HPLMN
-mcc: '901'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 
 # Permanent subscription key
 key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
@@ -20,7 +20,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
@@ -39,7 +39,7 @@
 # Initial PDU sessions to be established
 sessions:
   - type: 'IPv4'
-    apn: 'internet'
+    apn: 'ims'
     slice:
       sst: 1
 
```

<h2 id="network_settings">Network settings of Open5GS 5GC and UERANSIM UE / RAN</h2>

<h3 id="network_settings_up1">Network settings of Open5GS 5GC U-Plane1</h3>

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure the TUNnel interface and NAPT.
```
ip tuntap add name ogstun mode tun
ip addr add 10.45.0.1/16 dev ogstun
ip link set ogstun up

iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE

ip tuntap add name ogstun2 mode tun
ip addr add 10.46.0.1/16 dev ogstun2
ip link set ogstun2 up

iptables -t nat -A POSTROUTING -s 10.46.0.0/16 ! -o ogstun2 -j MASQUERADE
```

<h3 id="network_settings_up2">Network settings of Open5GS 5GC U-Plane2</h3>

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure the TUNnel interface and NAPT.
```
ip tuntap add name ogstun3 mode tun
ip addr add 10.47.0.1/16 dev ogstun3
ip link set ogstun3 up

iptables -t nat -A POSTROUTING -s 10.47.0.0/16 ! -o ogstun3 -j MASQUERADE
```

<h2 id="build">Build Open5GS and UERANSIM</h2>

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.4.7 - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM/wiki/Installation

Note. Install MongoDB with package manager on Open5GS 5GC C-Plane machine.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.
```
# apt update
# apt install mongodb
# systemctl start mongodb
# systemctl enable mongodb
```
It is not necessary to install MongoDB on Open5GS 5GC U-Plane machines.

<h2 id="run">Run Open5GS 5GC and UERANSIM UE / RAN</h2>

First run the 5GC, then UERANSIM (UE & RAN implementation).

<h3 id="run_cp">Run Open5GS 5GC C-Plane</h3>

First, run Open5GS 5GC C-Plane.

- Open5GS 5GC C-Plane
```
./install/bin/open5gs-nrfd &
sleep 5
./install/bin/open5gs-smfd &
./install/bin/open5gs-amfd &
./install/bin/open5gs-ausfd &
./install/bin/open5gs-udmd &
./install/bin/open5gs-udrd &
./install/bin/open5gs-pcfd &
./install/bin/open5gs-nssfd &
./install/bin/open5gs-bsfd &
```
Additional information.

PCF was added in Open5GS v2.1.0 released on 2020.12.11. Since PCF connects only to SBI, `pcf.yaml` is used as it is in this configuration example.
And NSSF was added in Open5GS v2.2.0 released on 2021.03.08. Since NSSF also connects only to SBI, `nssf.yaml` is used as it is in this configuration example.
And BSF was added in Open5GS v2.3.0 released on 2021.06.08. Since BSF also connects only to SBI, `bsf.yaml` is used as it is in this configuration example.

<h3 id="run_up">Run Open5GS 5GC U-Plane1 & U-Plane2</h3>

Next, run Open5GS 5GC U-Plane.

- Open5GS 5GC U-Plane1
```
./install/bin/open5gs-upfd &
```
- Open5GS 5GC U-Plane2
```
./install/bin/open5gs-upfd &
```

<h3 id="run_ueran">Run UERANSIM</h3>

Here, the case of UE0 (IMSI-001010000000000) & RAN is described.
First, do an NG Setup between gNodeB and 5GC, then register the UE with 5GC and establish a PDU session.

Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

<h4 id="start_gnb">Start gNB</h4>

Start gNB as follows.
```
# ./nr-gnb -c ../config/open5gs-gnb.yaml
UERANSIM v3.2.6
[2022-06-05 23:09:54.960] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2022-06-05 23:09:54.963] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2022-06-05 23:09:54.963] [sctp] [debug] SCTP association setup ascId[8]
[2022-06-05 23:09:54.964] [ngap] [debug] Sending NG Setup Request
[2022-06-05 23:09:54.964] [ngap] [debug] NG Setup Response received
[2022-06-05 23:09:54.964] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
06/05 23:09:54.969: [amf] INFO: gNB-N2 accepted[192.168.0.131]:55490 in ng-path module (../src/amf/ngap-sctp.c:113)
06/05 23:09:54.969: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:619)
06/05 23:09:54.970: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:876)
06/05 23:09:54.970: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:658)
```

<h4 id="start_ue">Start UE (UE0)</h4>

Start UE (UE0) as follows. This will register the UE with 5GC and establish a PDU session.
```
# ./nr-ue -c ../config/open5gs-ue0.yaml 
UERANSIM v3.2.6
[2022-06-05 23:10:39.362] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2022-06-05 23:10:39.362] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2022-06-05 23:10:39.363] [nas] [info] Selected plmn[001/01]
[2022-06-05 23:10:39.363] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2022-06-05 23:10:39.363] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2022-06-05 23:10:39.364] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2022-06-05 23:10:39.364] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2022-06-05 23:10:39.365] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-06-05 23:10:39.365] [nas] [debug] Sending Initial Registration
[2022-06-05 23:10:39.365] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2022-06-05 23:10:39.366] [rrc] [debug] Sending RRC Setup Request
[2022-06-05 23:10:39.367] [rrc] [info] RRC connection established
[2022-06-05 23:10:39.367] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2022-06-05 23:10:39.368] [nas] [info] UE switches to state [CM-CONNECTED]
[2022-06-05 23:10:39.375] [nas] [debug] Authentication Request received
[2022-06-05 23:10:39.378] [nas] [debug] Security Mode Command received
[2022-06-05 23:10:39.378] [nas] [debug] Selected integrity[2] ciphering[0]
[2022-06-05 23:10:39.388] [nas] [debug] Registration accept received
[2022-06-05 23:10:39.388] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2022-06-05 23:10:39.389] [nas] [debug] Sending Registration Complete
[2022-06-05 23:10:39.389] [nas] [info] Initial Registration is successful
[2022-06-05 23:10:39.390] [nas] [debug] Sending PDU Session Establishment Request
[2022-06-05 23:10:39.390] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-06-05 23:10:39.597] [nas] [debug] Configuration Update Command received
[2022-06-05 23:10:39.618] [nas] [debug] PDU Session Establishment Accept received
[2022-06-05 23:10:39.623] [nas] [info] PDU Session establishment is successful PSI[1]
[2022-06-05 23:10:39.642] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
06/05 23:10:39.372: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:361)
06/05 23:10:39.372: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2072)
06/05 23:10:39.372: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:497)
06/05 23:10:39.372: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1398)
06/05 23:10:39.372: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1197)
06/05 23:10:39.372: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:134)
06/05 23:10:39.372: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:157)
06/05 23:10:39.372: [app] WARNING: Try to discover [AUSF] (../lib/sbi/path.c:110)
06/05 23:10:39.373: [amf] INFO: [256d2ac8-e4d9-41ec-8efb-dd70ded2f7cd] (NF-discover) NF registered (../src/amf/nnrf-handler.c:344)
06/05 23:10:39.373: [amf] INFO: [256d2ac8-e4d9-41ec-8efb-dd70ded2f7cd] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:406)
06/05 23:10:39.374: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:110)
06/05 23:10:39.375: [ausf] INFO: [256c7f38-e4d9-41ec-b70a-cb392d41c775] (NF-discover) NF registered (../src/ausf/nnrf-handler.c:285)
06/05 23:10:39.375: [ausf] INFO: [256c7f38-e4d9-41ec-b70a-cb392d41c775] (NF-discover) NF Profile updated (../src/ausf/nnrf-handler.c:333)
06/05 23:10:39.383: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:110)
06/05 23:10:39.384: [amf] INFO: [256c7f38-e4d9-41ec-b70a-cb392d41c775] (NF-discover) NF registered (../src/amf/nnrf-handler.c:344)
06/05 23:10:39.384: [amf] INFO: [256c7f38-e4d9-41ec-b70a-cb392d41c775] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:406)
06/05 23:10:39.387: [app] WARNING: Try to discover [PCF] (../lib/sbi/path.c:110)
06/05 23:10:39.388: [amf] INFO: [2571975c-e4d9-41ec-baf4-cd4608083bee] (NF-discover) NF registered (../src/amf/nnrf-handler.c:344)
06/05 23:10:39.389: [amf] INFO: [2571975c-e4d9-41ec-baf4-cd4608083bee] (NF-discover) NF Profile updated (../src/amf/nnrf-handler.c:406)
06/05 23:10:39.596: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:1063)
06/05 23:10:39.598: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:389)
06/05 23:10:39.599: [gmm] INFO:     UTC [2022-06-05T14:10:39] Timezone[0]/DST[0] (../src/amf/gmm-build.c:502)
06/05 23:10:39.600: [gmm] INFO:     LOCAL [2022-06-05T23:10:39] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:507)
06/05 23:10:39.601: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2084)
06/05 23:10:39.601: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] (../src/amf/gmm-handler.c:1068)
06/05 23:10:39.603: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:846)
06/05 23:10:39.603: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:2868)
06/05 23:10:39.604: [app] WARNING: Try to discover [UDM] (../lib/sbi/path.c:110)
06/05 23:10:39.605: [smf] INFO: [256c7f38-e4d9-41ec-b70a-cb392d41c775] (NF-discover) NF registered (../src/smf/nnrf-handler.c:286)
06/05 23:10:39.606: [smf] INFO: [256c7f38-e4d9-41ec-b70a-cb392d41c775] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:334)
06/05 23:10:39.609: [app] WARNING: Try to discover [PCF] (../lib/sbi/path.c:110)
06/05 23:10:39.611: [smf] INFO: [2571975c-e4d9-41ec-baf4-cd4608083bee] (NF-discover) NF registered (../src/smf/nnrf-handler.c:286)
06/05 23:10:39.611: [smf] INFO: [2571975c-e4d9-41ec-baf4-cd4608083bee] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:334)
06/05 23:10:39.613: [app] WARNING: Try to discover [BSF] (../lib/sbi/path.c:110)
06/05 23:10:39.614: [pcf] INFO: [256c4626-e4d9-41ec-bce7-fff28bf83bd2] (NF-discover) NF registered (../src/pcf/nnrf-handler.c:288)
06/05 23:10:39.614: [pcf] INFO: [256c4626-e4d9-41ec-bce7-fff28bf83bd2] (NF-discover) NF Profile updated (../src/pcf/nnrf-handler.c:350)
06/05 23:10:39.616: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:492)
06/05 23:10:39.617: [gtp] INFO: gtp_connect() [192.168.0.112]:2152 (../lib/gtp/path.c:60)
06/05 23:10:39.617: [app] WARNING: Try to discover [AMF] (../lib/sbi/path.c:110)
06/05 23:10:39.618: [smf] INFO: [2570bb20-e4d9-41ec-9f29-971744169c26] (NF-discover) NF registered (../src/smf/nnrf-handler.c:286)
06/05 23:10:39.618: [smf] INFO: [2570bb20-e4d9-41ec-9f29-971744169c26] (NF-discover) NF Profile updated (../src/smf/nnrf-handler.c:334)
```
The Open5GS U-Plane1 log when executed is as follows.
```
06/05 23:10:39.598: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:178)
06/05 23:10:39.598: [gtp] INFO: gtp_connect() [192.168.0.111]:2152 (../lib/gtp/path.c:60)
06/05 23:10:39.598: [upf] INFO: UE F-SEID[CP:0x1 UP:0x1] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:397)
06/05 23:10:39.604: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:60)
```
Looking at the console log of the `nr-ue` command, UE0 has been assigned the IP address `10.45.0.2` from Open5GS 5GC.
```
[2022-06-05 23:10:39.642] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
Just in case, make sure it matches the IP address of the UE0's TUNnel interface.
```
# ip addr show
...
13: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::5d3e:bfe8:f4a4:c766/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h2 id="ping">Ping google.com</h2>

Specify the UE0's TUNnel interface and try ping.

Please refer to the following for usage of TUNnel interface.

https://github.com/aligungr/UERANSIM/wiki/Usage

<h3 id="ping_1">Case for going through DN 10.45.0.0/16</h3>

Execute `tcpdump` on VM2 (U-Plane1) and check that the packet goes through `if=ogstun`.
- `ping google.com` on VM5 (UE0)
```
# ping google.com -I uesimtun0 -n
PING google.com (142.251.42.142) from 10.45.0.2 uesimtun0: 56(84) bytes of data.
64 bytes from 142.251.42.142: icmp_seq=1 ttl=61 time=19.0 ms
64 bytes from 142.251.42.142: icmp_seq=2 ttl=61 time=18.5 ms
64 bytes from 142.251.42.142: icmp_seq=3 ttl=61 time=18.3 ms
```
- Run `tcpdump` on VM2 (U-Plane1)
```
# tcpdump -i ogstun -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ogstun, link-type RAW (Raw IP), capture size 262144 bytes
23:13:43.775588 IP 10.45.0.2 > 142.251.42.142: ICMP echo request, id 8, seq 1, length 64
23:13:43.791887 IP 142.251.42.142 > 10.45.0.2: ICMP echo reply, id 8, seq 1, length 64
23:13:44.776843 IP 10.45.0.2 > 142.251.42.142: ICMP echo request, id 8, seq 2, length 64
23:13:44.792868 IP 142.251.42.142 > 10.45.0.2: ICMP echo reply, id 8, seq 2, length 64
23:13:45.778149 IP 10.45.0.2 > 142.251.42.142: ICMP echo request, id 8, seq 3, length 64
23:13:45.794364 IP 142.251.42.142 > 10.45.0.2: ICMP echo reply, id 8, seq 3, length 64
```

You could specify the IP address assigned to the TUNnel interface to run almost any applications as in the following example using `nr-binder` tool.

- Run `curl google.com` on VM5 (UE0)
```
# sh nr-binder 10.45.0.2 curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
- Run `tcpdump` on VM2 (U-Plane1)
```
23:15:20.040857 IP 10.45.0.2.55853 > 172.217.31.142.80: Flags [S], seq 2033550471, win 65280, options [mss 1360,sackOK,TS val 872036884 ecr 0,nop,wscale 7], length 0
23:15:20.058280 IP 172.217.31.142.80 > 10.45.0.2.55853: Flags [S.], seq 2304001, ack 2033550472, win 65535, options [mss 1460], length 0
23:15:20.059963 IP 10.45.0.2.55853 > 172.217.31.142.80: Flags [.], ack 1, win 65280, length 0
23:15:20.060191 IP 10.45.0.2.55853 > 172.217.31.142.80: Flags [P.], seq 1:75, ack 1, win 65280, length 74: HTTP: GET / HTTP/1.1
23:15:20.060336 IP 172.217.31.142.80 > 10.45.0.2.55853: Flags [.], ack 75, win 65535, length 0
23:15:20.118319 IP 172.217.31.142.80 > 10.45.0.2.55853: Flags [P.], seq 1:529, ack 75, win 65535, length 528: HTTP: HTTP/1.1 301 Moved Permanently
23:15:20.119999 IP 10.45.0.2.55853 > 172.217.31.142.80: Flags [.], ack 529, win 64752, length 0
23:15:20.123544 IP 10.45.0.2.55853 > 172.217.31.142.80: Flags [F.], seq 75, ack 529, win 64752, length 0
23:15:20.123886 IP 172.217.31.142.80 > 10.45.0.2.55853: Flags [.], ack 76, win 65535, length 0
23:15:20.139491 IP 172.217.31.142.80 > 10.45.0.2.55853: Flags [F.], seq 529, ack 76, win 65535, length 0
23:15:20.140819 IP 10.45.0.2.55853 > 172.217.31.142.80: Flags [.], ack 530, win 64752, length 0
```
Please note that the `ping` tool does not work with `nr-binder`. Please refer to [here](https://github.com/aligungr/UERANSIM/issues/186#issuecomment-729534464) for the reason.

For `UE1`-`UE4` as well, execute `tcpdump` on each U-Plane and check the packets flowing through `ogstunX`.

You could now create the end-to-end TUN interfaces on the DN and send any packets on the network.

---
In investigating 5G SA, I have built a simulation environment and can now use a very useful system for investigating 5GC and MEC of 5G SA mobile network. I would like to thank the excellent developers and all the contributors of Open5GS and UERANSIM.

<h2 id="changelog">Changelog (summary)</h2>

- [2022.06.05] Updated to Open5GS v2.4.7 and UERANSIM v3.2.6.
- [2021.08.29] Updated to Open5GS v2.3.3 and added a little about BSF. And updated to UERANSIM v3.2.3.
- [2021.03.09] Updated to Open5GS v2.2.0 and added a little about NSSF. And updated to UERANSIM v3.1.3.
- [2021.01.28] Updated to UERANSIM v3.0.1 and updated the operation procedure.
- [2020.12.23] Updated to UERANSIM v2.2.1 and updated the operation procedure.
- [2020.12.20] Updated to UERANSIM v2.1.1.
- [2020.12.14] Updated to UERANSIM v2.0.1 and updated the operation procedure.
- [2020.12.13] Updated to Open5GS v2.1.0 and added a little about PCF.
- [2020.11.29] Initial release.

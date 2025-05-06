# free5GC 5GC & srsRAN 5G with ZeroMQ UE / RAN Sample Configuration
srsRAN_Project and srsRAN_4G software suites include a virtual radio which uses the ZeroMQ networking library to transfer radio samples between applications.
Therefore, in order to use U-Plane's DN (Data Network) as a trial, I built a simulation environment for the 5GC mobile network.
This briefly describes the overall and configuration files in the Proxmox VE environment.

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

- [Overview of free5GC 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of free5GC 5GC and srsRAN 5G ZMQ UE / RAN](#changes)
  - [Changes in configuration files of free5GC 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of free5GC 5GC U-Plane](#changes_up)
  - [Changes in configuration files of srsRAN 5G ZMQ UE / RAN](#changes_srs)
    - [Changes in configuration files of RAN](#changes_ran)
    - [Changes in configuration files of UE](#changes_ue)
- [Network settings of free5GC 5GC](#network_settings)
  - [Network settings of free5GC 5GC U-Plane](#network_settings_up)
- [Build free5GC and srsRAN 5G ZMQ UE / RAN](#build)
- [Run free5GC 5GC and srsRAN 5G ZMQ UE / RAN](#run)
  - [Run free5GC 5GC C-Plane](#run_cp)
  - [Run free5GC 5GC U-Plane](#run_up)
  - [Run srsRAN 5G ZMQ RAN](#run_ran)
  - [Run srsRAN 5G ZMQ UE](#run_ue)
- [Ping google.com](#ping)
  - [Case for going through DN 10.45.0.0/16](#ping_1)
- [Changelog (summary)](#changelog)
---
<a id="overview"></a>

## Overview of free5GC 5GC Simulation Mobile Network

I created a 5GC mobile network (Internet reachable) for simulation with the aim of creating an environment in which packets can be sent end-to-end with one DN for one DNN.

The following minimum configuration was set as a condition.
- Only one each for C-Plane, U-Plane and UE.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - free5GC v4.0.1 (2025.04.27) - https://github.com/free5gc/free5gc
- UPF - go-upf v1.2.6 (2025.03.03) - https://github.com/free5gc/go-upf
- (UPF) - gtp5g v0.9.14 (2025.04.21) - https://github.com/free5gc/gtp5g
- RAN - srsRAN Project (2025.04.23) - https://github.com/srsran/srsRAN_Project
- UE (NR-UE) - srsRAN 4G (2024.02.01) - https://github.com/srsran/srsRAN_4G

Each VMs are as follows.  
| VM | SW & Role | IP address | OS | CPU <br> (Min) | Mem <br> (Min) | HDD <br> (Min) |
| --- | --- | --- | --- | --- | --- | --- |
| VM1 | free5GC 5GC C-Plane | 192.168.0.141/24 | Ubuntu 24.04 | 1 | 2GB | 20GB |
| VM2 | free5GC 5GC U-Plane  | 192.168.0.142/24 | Ubuntu 24.04 | 1 | 1GB | 20GB |
| VM3 | srsRAN Project ZMQ RAN<br>(gNodeB) | 192.168.0.121/24 | Ubuntu 24.04 | 4 | 4GB | 10GB |
| VM4 | srsRAN 4G ZMQ UE<br>(NR-UE) | 192.168.0.122/24 | Ubuntu 22.04 | 1 | 2GB | 10GB |

AMF & SMF addresses are as follows.  
| NF | IP address | IP address on SBI | Supported S-NSSAI |
| --- | --- | --- | --- |
| AMF | 192.168.0.141 | 127.0.0.18 | SST:1, SD:0x000001 |
| SMF | 192.168.0.141 | 127.0.0.2 | SST:1, SD:0x000001 |

Subscriber Information (other information is the same) is as follows.  
| UE | IMSI | DNN | OP/OPc | S-NSSAI |
| --- | --- | --- | --- | --- |
| UE (NR-UE) | 001010000000000 | internet | OPc | SST:1, SD:0x000001 |

I registered these information with the free5GC WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

DN is as follows.
| DN | S-NSSAI | TUNnel interface of DN | DNN | TUNnel interface of UE |
| --- | --- | --- | --- | --- |
| 10.60.0.0/16 | SST:1 <br> SD:0x000001 | upfgtp | internet | tun_srsue |

The main information of gNodeB is as follows.
| MCC | MNC | TAC | gNodeB ID |
| --- | --- | --- | --- |
| 001 | 01 | 1 | 0x19b |

Additional information.

free5GC 5GC U-Plane worked fine on Raspberry Pi 4 Model B. I used [Ubuntu 20.04 (64bit) for Raspberry Pi 4](https://ubuntu.com/download/raspberry-pi) as the OS. I think it would be convenient to place a compact U-Plane in the edge environment and use it as an end-point for DN.

In addition, I have not confirmed the communication performance.

<a id="changes"></a>

## Changes in configuration files of free5GC 5GC and srsRAN 5G ZMQ UE / RAN

Please refer to the following for building free5GC and srsRAN 5G ZMQ UE / RAN respectively.
- free5GC v4.0.1 (2025.04.27) - https://free5gc.org/guide/
- go-upf v1.2.6 (2025.03.03) - https://free5gc.org/guide/
- srsRAN Project (RAN) (2025.04.23) - https://github.com/s5uishida/build_srsran_5g_zmq
- srsRAN 4G (UE) (2024.02.01) - https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins

<a id="changes_cp"></a>

### Changes in configuration files of free5GC 5GC C-Plane

The combination of DNN and S-NSSAI parameters can be used in the logic that selects UPF as the connection destination by PFCP.

- DNN
- S-NSSAI

For the sake of simplicity, This time, only DNN will be changed. S-NSSAI is fixed as `SST=1` and `SD=000001`.

- `free5gc/config/amfcfg.yaml`
```diff
--- amfcfg.yaml.orig    2024-10-13 05:09:24.000000000 +0900
+++ amfcfg.yaml 2025-05-06 09:35:08.253015221 +0900
@@ -5,7 +5,7 @@
 configuration:
   amfName: AMF # the name of this AMF
   ngapIpList:  # the IP list of N2 interfaces on this AMF
-    - 127.0.0.18
+    - 192.168.0.141
   ngapPort: 38412 # the SCTP port listened by NGAP
 
   # Service-based Interface (SBI) Configuration
@@ -30,25 +30,25 @@
   servedGuamiList:
     # <GUAMI> = <MCC><MNC><AMF ID>
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       amfId: cafe00 # AMF identifier (3 bytes hex string, range: 000000~FFFFFF)
 
   # the TAI (Tracking Area Identifier) list supported by this AMF
   supportTaiList:
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       tac: 000001 # Tracking Area Code (3 bytes hex string, range: 000000~FFFFFF)
 
   # the PLMNs (Public land mobile network) list supported by this AMF
   plmnSupportList:
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       snssaiList: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this AMF
         - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-          sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+          sd: 000001 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
         - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
           sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
 
```
- `free5gc/config/ausfcfg.yaml`
```diff
--- ausfcfg.yaml.orig   2024-09-01 09:47:28.000000000 +0900
+++ ausfcfg.yaml        2024-09-01 09:55:54.000000000 +0900
@@ -16,10 +16,8 @@
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
   nrfCertPem: cert/nrf.pem # NRF Certificate
   plmnSupportList: # the PLMNs (Public Land Mobile Network) list supported by this AUSF
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
-    - mcc: 123 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 45  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   groupId: ausfGroup001 # ID for the group of the AUSF
   eapAkaSupiImsiPrefix: false # including "imsi-" prefix or not when using the SUPI to do EAP-AKA' authentication
 
```
- `free5gc/config/nrfcfg.yaml`
```diff
--- nrfcfg.yaml.orig    2024-09-01 09:47:28.000000000 +0900
+++ nrfcfg.yaml 2024-09-01 09:56:10.000000000 +0900
@@ -18,8 +18,8 @@
       key: cert/root.key
     oauth: true
   DefaultPlmnId:
-    mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-    mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+    mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   serviceNameList: # the SBI services provided by this NRF, refer to TS 29.510
     - nnrf-nfm # Nnrf_NFManagement service
     - nnrf-disc # Nnrf_NFDiscovery service
```
- `free5gc/config/nssfcfg.yaml`
```diff
--- nssfcfg.yaml.orig   2024-09-01 09:47:28.000000000 +0900
+++ nssfcfg.yaml        2025-05-06 09:36:14.123669816 +0900
@@ -18,15 +18,15 @@
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
   nrfCertPem: cert/nrf.pem # NRF Certificate
   supportedPlmnList: # the PLMNs (Public land mobile network) list supported by this NSSF
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   supportedNssaiInPlmnList: # Supported S-NSSAI List for each PLMN
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       supportedSnssaiList: # Supported S-NSSAIs of the PLMN
         - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-          sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+          sd: 000001 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
         - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
           sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
         - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
```
- `free5gc/config/smfcfg.yaml`
```diff
--- smfcfg.yaml.orig    2024-10-13 05:09:24.000000000 +0900
+++ smfcfg.yaml 2025-05-06 09:45:07.677138771 +0900
@@ -25,7 +25,7 @@
   snssaiInfos:
     - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
         sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-        sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+        sd: 000001 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
       dnnInfos: # DNN information list
         - dnn: internet # Data Network Name
           dns: # the IP address of DNS
@@ -42,16 +42,16 @@
 
   # Optional: PLMN IDs configuration.
   plmnList:
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
 
   # PFCP (Packet Forwarding Control Protocol) configuration for N4 interface.
   pfcp:
     # addr config is deprecated in smf config v1.0.3, please use the following config
-    nodeID: 127.0.0.1 # the Node ID of this SMF
-    listenAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
-    externalAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    nodeID: 192.168.0.141 # the Node ID of this SMF
+    listenAddr: 192.168.0.141 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    externalAddr: 192.168.0.141 # the IP/FQDN of N4 interface on this SMF (PFCP)
     assocFailAlertInterval: 10s
     assocFailRetryInterval: 30s
     heartbeatInterval: 10s
@@ -63,12 +63,12 @@
         type: AN # the type of the node (AN or UPF)
       UPF: # the name of the node
         type: UPF # the type of the node (AN or UPF)
-        nodeID: 127.0.0.8 # the Node ID of this UPF
-        addr: 127.0.0.8 # the IP/FQDN of N4 interface on this UPF (PFCP)
+        nodeID: 192.168.0.142 # the Node ID of this UPF
+        addr: 192.168.0.142 # the IP/FQDN of N4 interface on this UPF (PFCP)
         sNssaiUpfInfos: # S-NSSAI information list for this UPF
           - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
               sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+              sd: 000001 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
             dnnUpfInfoList: # DNN information list for this S-NSSAI
               - dnn: internet
                 pools:
@@ -91,7 +91,7 @@
         interfaces: # Interface list for this UPF
           - interfaceType: N3 # the type of the interface (N3 or N9)
             endpoints: # the IP address of this N3/N9 interface on this UPF
-              - 127.0.0.8
+              - 192.168.0.142
             networkInstances: # Data Network Name (DNN)
               - internet
 
@@ -99,7 +99,7 @@
     links:
       - A: gNB1
         B: UPF
-
+  ulcl: false
   # retransmission timer for PDU session modification command
   t3591:
     enable: true # true or false
```

<a id="changes_up"></a>

### Changes in configuration files of free5GC 5GC U-Plane

- `go-upf/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2025-05-06 09:30:29.221750800 +0900
+++ upfcfg.yaml 2025-05-06 09:32:16.907209867 +0900
@@ -4,8 +4,8 @@
 # PFCP Configuration
 # The listen IP and nodeID of the N4 interface on this UPF (Can't set to 0.0.0.0)
 pfcp:
-  addr: 127.0.0.8   # IP addr for listening
-  nodeID: 127.0.0.8 # External IP or FQDN can be reached
+  addr: 192.168.0.142   # IP addr for listening
+  nodeID: 192.168.0.142 # External IP or FQDN can be reached
   retransTimeout: 1s # retransmission timeout
   maxRetrans: 3 # the max number of retransmission
 
@@ -18,7 +18,7 @@
   # If you bind to a specific IP, ensure SMF uses the same IP in its N3 configuration.
   # If you bind to all (0.0.0.0), SMF can use any of the available UPF IPs, but do not use 0.0.0.0 in SMF.
   ifList:
-    - addr: 127.0.0.8
+    - addr: 192.168.0.142
       type: N3
       # name: upf.5gc.nctu.me
       # ifname: gtpif
@@ -29,8 +29,6 @@
 dnnList:
   - dnn: internet # Data Network Name
     cidr: 10.60.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
-  - dnn: internet # Data Network Name
-    cidr: 10.61.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
     # natifname: eth0
 
 # Logging Configuration
```

<a id="changes_srs"></a>

### Changes in configuration files of srsRAN 5G ZMQ UE / RAN

<a id="changes_ran"></a>

#### Changes in configuration files of RAN

See [here](https://github.com/s5uishida/build_srsran_5g_zmq#create-the-configuration-file-of-gnodeb) for the original files.

- `srsRAN_Project/build/apps/gnb/gnb_zmq.yaml`
```diff
--- gnb_zmq.yaml.orig   2025-01-15 18:27:10.000000000 +0900
+++ gnb_zmq.yaml        2025-05-06 09:40:48.541434199 +0900
@@ -3,22 +3,30 @@
 # To run the srsRAN Project gNB with this config, use the following command: 
 #   sudo ./gnb -c gnb_zmq.yaml
 
+gnb_id: 0x19B
+
 cu_cp:
   amf:
-    addr: 10.53.1.2                 # The address or hostname of the AMF.
+    addr: 192.168.0.141                 # The address or hostname of the AMF.
     port: 38412
-    bind_addr: 10.53.1.1            # A local IP that the gNB binds to for traffic from the AMF.
+    bind_addr: 192.168.0.121            # A local IP that the gNB binds to for traffic from the AMF.
     supported_tracking_areas:
-      - tac: 7
+      - tac: 1
         plmn_list:
           - plmn: "00101"
             tai_slice_support_list:
               - sst: 1
+                sd: 1
   inactivity_timer: 7200            # Sets the UE/PDU Session/DRB inactivity timer to 7200 seconds. Supported: [1 - 7200].
 
+cu_up:
+  ngu:
+    socket:                               # Define socket(s) for NG-U interface.
+      - bind_addr: 192.168.0.121              # Optional TEXT (auto). Sets local IP address to bind for N3 interface. Format: IPV4 or IPV6 IP address.
+
 ru_sdr:
   device_driver: zmq                # The RF driver name.
-  device_args: tx_port=tcp://127.0.0.1:2000,rx_port=tcp://127.0.0.1:2001,base_srate=23.04e6 # Optionally pass arguments to the selected RF driver.
+  device_args: tx_port=tcp://192.168.0.121:2000,rx_port=tcp://192.168.0.122:2001,base_srate=23.04e6 # Optionally pass arguments to the selected RF driver.
   srate: 23.04                      # RF sample rate might need to be adjusted according to selected bandwidth.
   tx_gain: 75                       # Transmit gain of the RF might need to adjusted to the given situation.
   rx_gain: 75                       # Receive gain of the RF might need to adjusted to the given situation.
@@ -29,7 +37,7 @@
   channel_bandwidth_MHz: 20         # Bandwith in MHz. Number of PRBs will be automatically derived.
   common_scs: 15                    # Subcarrier spacing in kHz used for data.
   plmn: "00101"                     # PLMN broadcasted by the gNB.
-  tac: 7                            # Tracking area code (needs to match the core configuration).
+  tac: 1                            # Tracking area code (needs to match the core configuration).
   pdcch:
     common:
       ss0_index: 0                  # Set search space zero index to match srsUE capabilities
```

<a id="changes_ue"></a>

#### Changes in configuration files of UE

See [here](https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins#create-the-configuration-file-of-nr-ue) for the original files.

- `srsRAN_4G/build/srsue/ue_zmq.conf`
```diff
--- ue_zmq.conf.orig    2023-12-07 03:04:57.000000000 +0900
+++ ue_zmq.conf 2024-03-31 20:55:24.907470983 +0900
@@ -6,7 +6,7 @@
 nof_antennas = 1
 
 device_name = zmq
-device_args = tx_port=tcp://127.0.0.1:2001,rx_port=tcp://127.0.0.1:2000,base_srate=23.04e6
+device_args = tx_port=tcp://192.168.0.122:2001,rx_port=tcp://192.168.0.121:2000,base_srate=23.04e6
 
 [rat.eutra]
 dl_earfcn = 2850
@@ -34,9 +34,9 @@
 [usim]
 mode = soft
 algo = milenage
-opc  = 63BFA50EE6523365FF14C1F45F88737D
-k    = 00112233445566778899aabbccddeeff
-imsi = 001010123456780
+opc  = 8e27b6af0e692e750f32667a3b14605d
+k    = 8baf473f2f8fd09487cccbd7097c6862
+imsi = 001010000000000
 imei = 353490069873319
 
 [rrc]
@@ -44,11 +44,16 @@
 ue_category = 4
 
 [nas]
-apn = srsapn
+apn = internet
 apn_protocol = ipv4
 
+[slicing]
+enable = true
+nssai-sst = 1
+nssai-sd = 1
+
 [gw]
-netns = ue1
+#netns = ue1
 ip_devname = tun_srsue
 ip_netmask = 255.255.255.0
 
```

<a id="network_settings"></a>

## Network settings of free5GC 5GC

<a id="network_settings_up"></a>

### Network settings of free5GC 5GC U-Plane

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure the TUNnel interface and NAPT.
```
# iptables -t nat -A POSTROUTING -s 10.60.0.0/16 ! -o upfgtp -j MASQUERADE
```

<a id="build"></a>

## Build free5GC and srsRAN 5G ZMQ UE / RAN

Please refer to the following for building free5GC and srsRAN 5G ZMQ UE / RAN respectively.
- free5GC v4.0.1 (2025.04.27) - https://free5gc.org/guide/
- go-upf v1.2.6 (2025.03.03) - https://github.com/s5uishida/install_goupf
- srsRAN Project (RAN) (2025.04.23) - https://github.com/s5uishida/build_srsran_5g_zmq
- srsRAN 4G (UE) (2024.02.01) - https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins

Install MongoDB on free5GC 5GC C-Plane machine.
It is not necessary to install MongoDB on free5GC 5GC U-Plane machines.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.

**Note. The installation guide also includes instructions on building the latest committed version.**

<a id="run"></a>

## Run free5GC 5GC and srsRAN 5G ZMQ UE / RAN

First run the 5GC, then the RAN, and the UE.

<a id="run_cp"></a>

### Run free5GC 5GC C-Plane

First, run free5GC 5GC C-Plane.

- free5GC 5GC C-Plane

Create the following shell script and run it.
```bash
#!/usr/bin/env bash

PID_LIST=()

NF_LIST="nrf amf smf udr pcf udm nssf ausf chf nef"

export GIN_MODE=release

for NF in ${NF_LIST}; do
    ./bin/${NF} &
    PID_LIST+=($!)
    sleep 1
done

function terminate()
{
    sudo kill -SIGTERM ${PID_LIST[${#PID_LIST[@]}-2]} ${PID_LIST[${#PID_LIST[@]}-1]}
    sleep 2
}

trap terminate SIGINT
wait ${PID_LIST}
```

<a id="run_up"></a>

### Run free5GC 5GC U-Plane

Next, run free5GC 5GC U-Plane.

- free5GC 5GC U-Plane
```
# cd go-upf
# ./upf -c upfcfg.yaml
```

<a id="run_ran"></a>

### Run srsRAN 5G ZMQ RAN

Run srsRAN 5G ZMQ RAN and connect to free5GC 5GC.
```
# cd srsRAN_Project/build/apps/gnb
# ./gnb -c gnb_zmq.yaml

--== srsRAN gNB (commit 644263b5a7) ==--

Lower PHY in executor blocking mode.
Available radio types: zmq.
Cell pci=1, bw=20 MHz, 1T1R, dl_arfcn=368500 (n3), dl_freq=1842.5 MHz, dl_ssb_arfcn=368410, ul_freq=1747.5 MHz

N2: Connection to AMF on 192.168.0.141:38412 completed
==== gNB started ===
Type <h> to view help
```
The free5GC C-Plane log when executed is as follows.
```
2025-05-06T10:07:56.763895763+09:00 [INFO][AMF][Ngap] [AMF] SCTP Accept from: 192.168.0.121:42098
2025-05-06T10:07:56.764198124+09:00 [INFO][AMF][Ngap] Create a new NG connection for: 192.168.0.121:42098
2025-05-06T10:07:56.764458687+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:42098] Handle NGSetupRequest
2025-05-06T10:07:56.764506789+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:42098] Send NG-Setup response
```

<a id="run_ue"></a>

### Run srsRAN 5G ZMQ UE

Run srsRAN 5G ZMQ UE and connect to free5GC 5GC.
```
# cd srsRAN_4G/build/srsue
# ./src/srsue ue_zmq.conf
Reading configuration file ue_zmq.conf...

Built in Release mode using commit ec29b0c1f on branch master.

Opening 1 channels in RF device=zmq with args=tx_port=tcp://192.168.0.122:2001,rx_port=tcp://192.168.0.121:2000,base_srate=23.04e6
Supported RF device list: zmq file
CHx base_srate=23.04e6
Current sample rate is 1.92 MHz with a base rate of 23.04 MHz (x12 decimation)
CH0 rx_port=tcp://192.168.0.121:2000
CH0 tx_port=tcp://192.168.0.122:2001
Current sample rate is 23.04 MHz with a base rate of 23.04 MHz (x1 decimation)
Current sample rate is 23.04 MHz with a base rate of 23.04 MHz (x1 decimation)
Waiting PHY to initialize ... done!
Attaching UE...
Random Access Transmission: prach_occasion=0, preamble_index=0, ra-rnti=0x39, tti=334
Random Access Complete.     c-rnti=0x4601, ta=0
RRC Connected
PDU Session Establishment successful. IP: 10.60.0.1
RRC NR reconfiguration successful.
```
The free5GC C-Plane log when executed is as follows.
```
2025-05-06T10:08:31.073318424+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:42098] Handle InitialUEMessage
2025-05-06T10:08:31.073369699+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] New RanUe [RanUeNgapID:0][AmfUeNgapID:1]
2025-05-06T10:08:31.073398904+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:42098] 5GSMobileIdentity ["SUCI":"suci-0-001-01-0000-0-0-0000000000", err: <nil>]
2025-05-06T10:08:31.073444819+09:00 [INFO][AMF][CTX] New AmfUe [supi:][guti:00101cafe0000000001]
2025-05-06T10:08:31.073552126+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2025-05-06T10:08:31.073561306+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Handle Registration Request
2025-05-06T10:08:31.073567871+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] RegistrationType: Initial Registration
2025-05-06T10:08:31.073574852+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] MobileIdentity5GS: SUCI[suci-0-001-01-0000-0-0-0000000000]
2025-05-06T10:08:31.073585797+09:00 [INFO][AMF][Gmm] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2025-05-06T10:08:31.073591765+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Authentication procedure
2025-05-06T10:08:31.074481408+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.076761536+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.079046405+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.084190719+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.087199574+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2025-05-06T10:08:31.088325206+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.090285916+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.092930584+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.095040504+09:00 [INFO][AUSF][UeAuth] HandleUeAuthPostRequest
2025-05-06T10:08:31.095200841+09:00 [INFO][AUSF][UeAuth] Serving network authorized
2025-05-06T10:08:31.096112894+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.097278883+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.099261529+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.100141991+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.101371290+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2025-05-06T10:08:31.102653330+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.103853693+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.107106941+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.109267934+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2025-05-06T10:08:31.110404322+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.111829843+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.114661083+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.115329643+09:00 [INFO][UDM][Suci] scheme 0
2025-05-06T10:08:31.115500064+09:00 [INFO][UDM][Suci] SUPI type is IMSI
2025-05-06T10:08:31.115952812+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.117232375+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.119400870+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.120437991+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.121863285+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=UDM&target-nf-type=UDR |
2025-05-06T10:08:31.130015107+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v2/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2025-05-06T10:08:31.130721660+09:00 [INFO][UDM][Proc] ModifyAuthenticationSubscriptionRequest:  [{replace /sequenceNumber  { 000000000025 map[] 0 }}]
2025-05-06T10:08:31.133210037+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v2/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2025-05-06T10:08:31.133812653+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2025-05-06T10:08:31.134345206+09:00 [INFO][AUSF][UeAuth] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2025-05-06T10:08:31.134533445+09:00 [INFO][AUSF][UeAuth] Use 5G AKA auth method
2025-05-06T10:08:31.134581320+09:00 [INFO][AUSF][5gAka] XresStar = 6564333961636261393162336663643366616434633130613762613764396561
2025-05-06T10:08:31.134827872+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2025-05-06T10:08:31.135748381+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Send Authentication Request
2025-05-06T10:08:31.135838222+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] Send Downlink Nas Transport
2025-05-06T10:08:31.136064898+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Start T3560 timer
2025-05-06T10:08:31.161273490+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:42098] Handle UplinkNASTransport
2025-05-06T10:08:31.161368795+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2025-05-06T10:08:31.161569271+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2025-05-06T10:08:31.161637886+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Handle Authentication Response
2025-05-06T10:08:31.161727484+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Stop T3560 timer
2025-05-06T10:08:31.162721454+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.164367089+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.167215970+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.168880928+09:00 [INFO][AUSF][5gAka] Auth5gAkaComfirmRequest
2025-05-06T10:08:31.168951345+09:00 [INFO][AUSF][5gAka] res*: 6564333961636261393162336663643366616434633130613762613764396561
Xres*: 6564333961636261393162336663643366616434633130613762613764396561
2025-05-06T10:08:31.169024496+09:00 [INFO][AUSF][5gAka] 5G AKA confirmation succeeded
2025-05-06T10:08:31.170064965+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.171222373+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.174383339+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.176341764+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2025-05-06T10:08:31.177479912+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.178966907+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.181589684+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.184539064+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v2/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2025-05-06T10:08:31.185012538+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2025-05-06T10:08:31.185510821+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2025-05-06T10:08:31.186160967+09:00 [INFO][AMF][Gmm] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2025-05-06T10:08:31.186245492+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Security Mode Command
2025-05-06T10:08:31.186382632+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] Send Downlink Nas Transport
2025-05-06T10:08:31.186552998+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3560 timer
2025-05-06T10:08:31.225947940+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:42098] Handle UplinkNASTransport
2025-05-06T10:08:31.226010480+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2025-05-06T10:08:31.226204024+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2025-05-06T10:08:31.226359424+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Security Mode Complete
2025-05-06T10:08:31.226463568+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3560 timer
2025-05-06T10:08:31.226568225+09:00 [INFO][AMF][Gmm] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2025-05-06T10:08:31.226654858+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle InitialRegistration
2025-05-06T10:08:31.227539303+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.229192750+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.231393711+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.232666903+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.234051032+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2025-05-06T10:08:31.234901092+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.236660557+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.239841488+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.241741720+09:00 [INFO][UDM][Consumer] TwoLayerPathHandlerFunc,  imsi-001010000000000 nssai
2025-05-06T10:08:31.241891651+09:00 [INFO][UDM][SDM] Handle GetNssai
2025-05-06T10:08:31.242793984+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.245018026+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.249216694+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.252098281+09:00 [INFO][UDR][DataRepo] QueryAmDataProcedure: ueId: imsi-001010000000000, servingPlmnId: 00101
2025-05-06T10:08:31.253169563+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v2/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features= |
2025-05-06T10:08:31.254614370+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v2/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2025-05-06T10:08:31.255268159+09:00 [INFO][AMF][Gmm] RequestedNssai: &{Iei:47 Len:5 Buffer:[4 1 0 0 1]}
2025-05-06T10:08:31.255457028+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:000001}, HomeSnssai: <nil>
2025-05-06T10:08:31.256442983+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.258161734+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.260400096+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.261518248+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.262858743+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2025-05-06T10:08:31.263623333+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.265255983+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.268536905+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.270684288+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2025-05-06T10:08:31.270849032+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2025-05-06T10:08:31.271633824+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.273270749+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.276001753+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.279184331+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v2/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2025-05-06T10:08:31.279546951+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2025-05-06T10:08:31.280978715+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.282883643+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.285974331+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.287901231+09:00 [INFO][UDM][Consumer] TwoLayerPathHandlerFunc,  imsi-001010000000000 am-data
2025-05-06T10:08:31.288075923+09:00 [INFO][UDM][SDM] Handle GetAmData
2025-05-06T10:08:31.288947408+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.290455186+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.293297524+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.295089136+09:00 [INFO][UDR][DataRepo] QueryAmDataProcedure: ueId: imsi-001010000000000, servingPlmnId: 00101
2025-05-06T10:08:31.297320189+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v2/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2025-05-06T10:08:31.299084237+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v2/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2025-05-06T10:08:31.302712523+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.304917174+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.308883041+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.312086346+09:00 [INFO][UDM][Consumer] TwoLayerPathHandlerFunc,  imsi-001010000000000 smf-select-data
2025-05-06T10:08:31.312462918+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2025-05-06T10:08:31.313408042+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.314796440+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.317659630+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.319954737+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v2/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data?supported-features= |
2025-05-06T10:08:31.320633560+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v2/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2025-05-06T10:08:31.322023572+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.323724041+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.327228710+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.328991241+09:00 [INFO][UDM][Consumer] TwoLayerPathHandlerFunc,  imsi-001010000000000 ue-context-in-smf-data
2025-05-06T10:08:31.329224322+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2025-05-06T10:08:31.330139753+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.331755966+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.334581487+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.336812346+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v2/subscription-data/imsi-001010000000000/context-data/smf-registrations?supported-features= |
2025-05-06T10:08:31.337395092+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v2/imsi-001010000000000/ue-context-in-smf-data |
2025-05-06T10:08:31.338876312+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.340432655+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.343742284+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.349052827+09:00 [INFO][UDM][Consumer] TwoLayerPathHandlerFunc,  imsi-001010000000000 sdm-subscriptions
2025-05-06T10:08:31.350154978+09:00 [INFO][UDM][SDM] Handle Subscribe
2025-05-06T10:08:31.351183477+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.352718925+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.355654149+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.359105440+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v2/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2025-05-06T10:08:31.359679390+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v2/imsi-001010000000000/sdm-subscriptions |
2025-05-06T10:08:31.361188966+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.362670852+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.364998428+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.366024370+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.367951083+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2025-05-06T10:08:31.370603984+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.372826575+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.376176020+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.378913380+09:00 [INFO][PCF][AmPol] Handle AM Policy Create Request
2025-05-06T10:08:31.380076616+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.381446222+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.383539018+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.384542425+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.385575021+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2025-05-06T10:08:31.386781897+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.388462673+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.391365003+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.393755685+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v2/policy-data/ues/imsi-001010000000000/am-data |
2025-05-06T10:08:31.395132449+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.396830413+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.399080737+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.400159068+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.401726742+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe00%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2025-05-06T10:08:31.402648032+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.404031978+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.407457985+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.409413302+09:00 [INFO][AMF][Comm] Handle AMF Status Change Subscribe Request
2025-05-06T10:08:31.409653384+09:00 [INFO][AMF][Comm] new AMF Status Subscription[1]
2025-05-06T10:08:31.409801104+09:00 [INFO][AMF][GIN] | 201 |       127.0.0.1 | POST    | /namf-comm/v1/subscriptions |
2025-05-06T10:08:31.410440624+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2025-05-06T10:08:31.411047289+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Registration Accept
2025-05-06T10:08:31.411206072+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] Send Initial Context Setup Request
2025-05-06T10:08:31.411373782+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3550 timer
2025-05-06T10:08:31.484978810+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:42098] Handle InitialContextSetupResponse
2025-05-06T10:08:31.485019356+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] Handle InitialContextSetupResponse (RAN UE NGAP ID: 0)
2025-05-06T10:08:31.685212992+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:42098] Handle UERadioCapabilityInfoIndication
2025-05-06T10:08:31.685248447+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] Handle UERadioCapabilityInfoIndication (RAN UE NGAP ID: 0)
2025-05-06T10:08:31.685301471+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:42098] Handle UplinkNASTransport
2025-05-06T10:08:31.685309593+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2025-05-06T10:08:31.685378812+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2025-05-06T10:08:31.685389872+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Registration Complete
2025-05-06T10:08:31.685396358+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3550 timer
2025-05-06T10:08:31.685415104+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Configuration Update Command
2025-05-06T10:08:31.685430498+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] Send Downlink Nas Transport
2025-05-06T10:08:31.685506100+09:00 [INFO][AMF][Gmm] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2025-05-06T10:08:31.685556592+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:42098] Handle UplinkNASTransport
2025-05-06T10:08:31.685564318+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2025-05-06T10:08:31.685596028+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2025-05-06T10:08:31.685601949+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle UL NAS Transport
2025-05-06T10:08:31.685607511+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2025-05-06T10:08:31.685617114+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:000001}, dnn: internet]
2025-05-06T10:08:31.686601297+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.688123035+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.690472678+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.691616395+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.692695373+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2025-05-06T10:08:31.693433369+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.694894299+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.697822680+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.699783369+09:00 [INFO][NSSF][NsSel] Handle NSSelectionGet
2025-05-06T10:08:31.700125325+09:00 [WARN][NSSF][Util] No TA {"plmnId":{"mcc":"001","mnc":"01"},"tac":"000001"} in NSSF configuration
2025-05-06T10:08:31.700363164+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v2/network-slice-information?nf-id=60be98fb-f23f-41dd-b122-1f4b6e38ac33&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D&tai=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22tac%22%3A%22000001%22%7D |
2025-05-06T10:08:31.700996899+09:00 [WARN][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] nsiInformation is still nil, use default NRF[http://127.0.0.10:8000]
2025-05-06T10:08:31.702033413+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.703838685+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.706325182+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.708739634+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.711359042+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=area1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%5B%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D%5D&target-nf-type=SMF&target-plmn-list=%5B%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%5D |
2025-05-06T10:08:31.712327386+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.713950423+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.717149817+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.719328438+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2025-05-06T10:08:31.721377690+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2025-05-06T10:08:31.722740631+09:00 [INFO][SMF][CTX] UrrPeriod: 30s
2025-05-06T10:08:31.722886493+09:00 [INFO][SMF][CTX] UrrThreshold: 500000
2025-05-06T10:08:31.724832058+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.726675180+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.728951544+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.729955608+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.731245916+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2025-05-06T10:08:31.731966811+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Send NF Discovery Serving UDM Successfully
2025-05-06T10:08:31.732481448+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.734018508+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.737121103+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.738446749+09:00 [INFO][UDM][Consumer] TwoLayerPathHandlerFunc,  imsi-001010000000000 sm-data
2025-05-06T10:08:31.738908943+09:00 [INFO][UDM][SDM] Handle GetSmData
2025-05-06T10:08:31.739819228+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.741378097+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.744161002+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.744632791+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"000001"}]
2025-05-06T10:08:31.746642670+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v2/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2025-05-06T10:08:31.747305386+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v2/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2025-05-06T10:08:31.748544265+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2025-05-06T10:08:31.749566713+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.750938402+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.753126812+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.754105205+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.755574633+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=60be98fb-f23f-41dd-b122-1f4b6e38ac33&target-nf-type=AMF |
2025-05-06T10:08:31.756185366+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2025-05-06T10:08:31.756390228+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.60.0.1
2025-05-06T10:08:31.756545188+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2025-05-06T10:08:31.756622264+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Allocated PDUAdress[10.60.0.1]
2025-05-06T10:08:31.757100175+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.758439341+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.760596479+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.762379746+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.763750800+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=SMF&target-nf-type=PCF |
2025-05-06T10:08:31.764542678+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.765813388+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.769006353+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.770960565+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2025-05-06T10:08:31.771951506+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.773952871+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.778459628+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.782200843+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v2/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2025-05-06T10:08:31.786607413+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.788178547+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.790972233+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.793207741+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v2/application-data/influenceData?dnns=internet&snssais=%5B%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D%5D&supis=imsi-001010000000000 |
2025-05-06T10:08:31.793764727+09:00 [INFO][PCF][SMpolicy] Matched [0] trafficInfluDatas from UDR
2025-05-06T10:08:31.794653463+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.796287737+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.799197886+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.801057111+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v2/application-data/influenceData/subs-to-notify |
2025-05-06T10:08:31.802221579+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.804018963+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.806160357+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.807169944+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.807891792+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=BSF |
2025-05-06T10:08:31.809065556+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2025-05-06T10:08:31.810570976+09:00 [INFO][SMF][PduSess] CHF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2025-05-06T10:08:31.811704613+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.813194870+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.815423413+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.816539069+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2025-05-06T10:08:31.817545810+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=CHF |
2025-05-06T10:08:31.818141460+09:00 [INFO][SMF][Charging] Handle SendConvergedChargingRequest
2025-05-06T10:08:31.818587204+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.819762111+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.822616010+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.828535719+09:00 [INFO][CHF][ChargingPost] HandleChargingdataInitial
2025-05-06T10:08:31.828808231+09:00 [INFO][CHF][ChargingPost] SMF charging event
2025-05-06T10:08:31.829158162+09:00 [ERRO][CHF][ChargingPost] Charging gateway fail to send CDR to billing domain dial tcp 127.0.0.1:2121: connect: connection refused
2025-05-06T10:08:31.829207609+09:00 [INFO][CHF][ChargingPost] Open CDR for UE imsi-001010000000000
2025-05-06T10:08:31.829269634+09:00 [INFO][CHF][ChargingPost] NewChfUe imsi-001010000000000
2025-05-06T10:08:31.829622864+09:00 [INFO][CHF][GIN] | 201 |       127.0.0.1 | POST    | /nchf-convergedcharging/v3/chargingdata |
2025-05-06T10:08:31.830411173+09:00 [INFO][SMF][Charging] Send Charging Data Request[Init] successfully
2025-05-06T10:08:31.830714999+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Install PCCRule[PccRuleId-1]
2025-05-06T10:08:31.830765335+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] No srcTcData and tgtTcData. Nothing to do
2025-05-06T10:08:31.830857664+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Has default path
2025-05-06T10:08:31.832991941+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2025-05-06T10:08:31.835786127+09:00 [INFO][UDM][Consumer] TwoLayerPathHandlerFunc,  imsi-001010000000000 sdm-subscriptions
2025-05-06T10:08:31.836107140+09:00 [INFO][UDM][SDM] Handle Subscribe
2025-05-06T10:08:31.837542563+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.838451860+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2025-05-06T10:08:31.840060765+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.844731269+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.845245790+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.846950606+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.848530810+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v2/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2025-05-06T10:08:31.850820282+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v2/imsi-001010000000000/sdm-subscriptions |
2025-05-06T10:08:31.851267105+09:00 [INFO][SMF][PduSess] SDM Subscription Successful UE: imsi-001010000000000 SubscriptionId: 2
2025-05-06T10:08:31.851409839+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2025-05-06T10:08:31.852064171+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2025-05-06T10:08:31.852778010+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:42098] Handle UplinkNASTransport
2025-05-06T10:08:31.853172261+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2025-05-06T10:08:31.853453235+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2025-05-06T10:08:31.853605484+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Configuration Update Complete
2025-05-06T10:08:31.856093344+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.859721796+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2025-05-06T10:08:31.859764934+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] Send PDU Session Resource Setup Request
2025-05-06T10:08:31.860023823+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2025-05-06T10:08:31.910893636+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:42098] Handle PDUSessionResourceSetupResponse
2025-05-06T10:08:31.911137162+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:42098] Handle PDUSessionResourceSetupResponse (RAN UE NGAP ID: 0)
2025-05-06T10:08:31.912101437+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2025-05-06T10:08:31.913814555+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2025-05-06T10:08:31.917048375+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2025-05-06T10:08:31.919238196+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2025-05-06T10:08:31.921217149+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2025-05-06T10:08:31.921353683+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:def76a9e-4755-44d1-aff2-71cef5b2f79c/modify |
```
The free5GC U-Plane log when executed is as follows.
```
2025-05-06T10:08:31.855461739+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionEstablishmentRequest
2025-05-06T10:08:31.855499686+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805][CPNodeID:192.168.0.141][CPSEID:0x1][UPSEID:0x1] New session
2025-05-06T10:08:31.856737437+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:1 period:30000000000}]
2025-05-06T10:08:31.857111122+09:00 [INFO][UPF][Perio] new ticker [30s]
2025-05-06T10:08:31.857153829+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:2 period:30000000000}]
2025-05-06T10:08:31.942158063+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionModificationRequest
```
The result of `ip addr show` on VM4 (UE) is as follows.
```
# ip addr show
...
9: tun_srsue: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.60.0.1/24 scope global tun_srsue
       valid_lft forever preferred_lft forever
...
```

<a id="ping"></a>

## Ping google.com

Specify the TUN interface on VM4 (UE) and try `ping`.

<a id="ping_1"></a>

### Case for going through DN 10.60.0.0/16

Execute `tcpdump` on VM2 (U-Plane) and check that the packet goes through `if=upfgtp`.
- `ping google.com` on VM4 (UE)
```
# ping google.com -I tun_srsue -n
PING google.com (142.250.196.142) from 10.60.0.1 tun_srsue: 56(84) bytes of data.
64 bytes from 142.250.196.142: icmp_seq=1 ttl=110 time=59.8 ms
64 bytes from 142.250.196.142: icmp_seq=2 ttl=110 time=46.2 ms
64 bytes from 142.250.196.142: icmp_seq=3 ttl=110 time=57.2 ms
```
- Run `tcpdump` on VM2 (U-Plane)
```
# tcpdump -i upfgtp -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on upfgtp, link-type RAW (Raw IP), snapshot length 262144 bytes
10:12:25.547908 IP 10.60.0.1 > 142.250.196.142: ICMP echo request, id 6, seq 1, length 64
10:12:25.566977 IP 142.250.196.142 > 10.60.0.1: ICMP echo reply, id 6, seq 1, length 64
10:12:26.535836 IP 10.60.0.1 > 142.250.196.142: ICMP echo request, id 6, seq 2, length 64
10:12:26.552341 IP 142.250.196.142 > 10.60.0.1: ICMP echo reply, id 6, seq 2, length 64
10:12:27.548918 IP 10.60.0.1 > 142.250.196.142: ICMP echo request, id 6, seq 3, length 64
10:12:27.565576 IP 142.250.196.142 > 10.60.0.1: ICMP echo reply, id 6, seq 3, length 64
```
In addition to `ping`, you may try to access the web by specifying the TUNnel interface with `curl` as follows.
- `curl google.com` on VM4 (UE)
```
# curl --interface tun_srsue google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
- Run `tcpdump` on VM2 (U-Plane)
```
10:13:04.569502 IP 10.60.0.1.45280 > 142.250.196.142.80: Flags [S], seq 2623065466, win 64240, options [mss 1460,sackOK,TS val 2487357686 ecr 0,nop,wscale 7], length 0
10:13:04.585078 IP 142.250.196.142.80 > 10.60.0.1.45280: Flags [S.], seq 1740632482, ack 2623065467, win 65535, options [mss 1412,sackOK,TS val 3806109728 ecr 2487357686,nop,wscale 8], length 0
10:13:04.612652 IP 10.60.0.1.45280 > 142.250.196.142.80: Flags [.], ack 1, win 502, options [nop,nop,TS val 2487357731 ecr 3806109728], length 0
10:13:04.612733 IP 10.60.0.1.45280 > 142.250.196.142.80: Flags [P.], seq 1:75, ack 1, win 502, options [nop,nop,TS val 2487357731 ecr 3806109728], length 74: HTTP: GET / HTTP/1.1
10:13:04.631382 IP 142.250.196.142.80 > 10.60.0.1.45280: Flags [.], ack 75, win 1050, options [nop,nop,TS val 3806109774 ecr 2487357731], length 0
10:13:04.673209 IP 142.250.196.142.80 > 10.60.0.1.45280: Flags [P.], seq 1:774, ack 75, win 1050, options [nop,nop,TS val 3806109816 ecr 2487357731], length 773: HTTP: HTTP/1.1 301 Moved Permanently
10:13:04.699788 IP 10.60.0.1.45280 > 142.250.196.142.80: Flags [.], ack 774, win 501, options [nop,nop,TS val 2487357819 ecr 3806109816], length 0
10:13:04.699788 IP 10.60.0.1.45280 > 142.250.196.142.80: Flags [F.], seq 75, ack 774, win 501, options [nop,nop,TS val 2487357820 ecr 3806109816], length 0
10:13:04.715948 IP 142.250.196.142.80 > 10.60.0.1.45280: Flags [F.], seq 774, ack 76, win 1050, options [nop,nop,TS val 3806109859 ecr 2487357820], length 0
10:13:04.742945 IP 10.60.0.1.45280 > 142.250.196.142.80: Flags [.], ack 775, win 501, options [nop,nop,TS val 2487357860 ecr 3806109859], length 0
```
You could now create the end-to-end TUN interface on the DN and send any packets on the network.

---
In investigating 5G SA, I have built a simulation environment and can now use a very useful system for investigating 5GC and MEC of 5G SA mobile network. I would like to thank the excellent developers and all the contributors of free5GC, srsRAN Project and srsRAN 4G.

<a id="changelog"></a>

## Changelog (summary)

- [2025.05.06] Updated to free5GC `v4.0.1 (2025.04.27)`, go-upf `v1.2.6 (2025.03.03)`, gtp5g `v0.9.14 (2025.04.21)`, srsRAN_Project `24.10 (2025.04.23)` and `gnb_zmq.yaml` according to srsRAN_Project `24.10 (2025.04.23)`.
- [2024.11.08] Updated to free5GC `v3.4.3 (2024.11.08)` and `gnb_zmq.yaml` according to srsRAN_Project `24.10 (2024.10.14)`.
- [2024.03.31] Updated to free5GC `v3.4.1 (2024.03.28)`.
- [2023.11.02] Updated `gnb_zmq.yaml`.
- [2023.10.21] Updated `gnb_zmq.yaml` according to srsRAN_Project `23.10 (2023.10.20)`.
- [2023.10.12] Updated free5GC `v3.3.0 (2023.10.11)`.
- [2023.10.11] Updated srsRAN_Project `(2023.09.20)`.
- [2023.08.26] Initial release.

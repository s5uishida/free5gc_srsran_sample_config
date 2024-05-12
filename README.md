# free5GC 5GC & srsRAN 5G with ZeroMQ UE / RAN Sample Configuration
srsRAN_Project and srsRAN_4G software suites include a virtual radio which uses the ZeroMQ networking library to transfer radio samples between applications.
Therefore, in order to use U-Plane's DN (Data Network) as a trial, I built a simulation environment for the 5GC mobile network.
This briefly describes the overall and configuration files in the Virtualbox VM environment.

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
- 5GC - free5GC v3.4.1 (2024.03.28) - https://github.com/free5gc/free5gc
- RAN - srsRAN Project (2024.03.25) - https://github.com/srsran/srsRAN_Project
- UE (NR-UE) - srsRAN 4G (2024.02.01) - https://github.com/srsran/srsRAN_4G

Each VMs are as follows.  
| VM | SW & Role | IP address | OS | CPU <br> (Min) | Memory <br> (Min) | HDD <br> (Min) |
| --- | --- | --- | --- | --- | --- | --- |
| VM1 | free5GC 5GC C-Plane | 192.168.0.141/24 | Ubuntu 22.04 | 1 | 2GB | 20GB |
| VM2 | free5GC 5GC U-Plane  | 192.168.0.142/24 | Ubuntu 22.04 | 1 | 1GB | 20GB |
| VM3 | srsRAN Project ZMQ RAN<br>(gNodeB) | 192.168.0.121/24 | Ubuntu 22.04 | 2 | 4GB | 10GB |
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
- free5GC v3.4.1 (2024.03.28) - https://free5gc.org/guide/
- srsRAN Project (RAN) (2024.03.25) - https://github.com/s5uishida/build_srsran_5g_zmq
- srsRAN 4G (UE) (2024.02.01) - https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins

<a id="changes_cp"></a>

### Changes in configuration files of free5GC 5GC C-Plane

The combination of DNN and S-NSSAI parameters can be used in the logic that selects UPF as the connection destination by PFCP.

- DNN
- S-NSSAI

For the sake of simplicity, This time, only DNN will be changed. S-NSSAI is fixed as `SST=1` and `SD=000001`.

- `free5gc/config/amfcfg.yaml`
```diff
--- amfcfg.yaml.orig    2024-03-30 10:35:10.534612278 +0900
+++ amfcfg.yaml 2024-03-31 20:30:58.118203896 +0900
@@ -5,7 +5,7 @@
 configuration:
   amfName: AMF # the name of this AMF
   ngapIpList:  # the IP list of N2 interfaces on this AMF
-    - 127.0.0.18
+    - 192.168.0.141
   ngapPort: 38412 # the SCTP port listened by NGAP
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
@@ -24,21 +24,21 @@
   servedGuamiList: # Guami (Globally Unique AMF ID) list supported by this AMF
     # <GUAMI> = <MCC><MNC><AMF ID>
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       amfId: cafe00 # AMF identifier (3 bytes hex string, range: 000000~FFFFFF)
   supportTaiList:  # the TAI (Tracking Area Identifier) list supported by this AMF
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       tac: 000001 # Tracking Area Code (3 bytes hex string, range: 000000~FFFFFF)
   plmnSupportList: # the PLMNs (Public land mobile network) list supported by this AMF
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
   supportDnnList:  # the DNN (Data Network Name) list supported by this AMF
```
- `free5gc/config/ausfcfg.yaml`
```diff
--- ausfcfg.yaml.orig   2024-03-30 10:35:10.534612278 +0900
+++ ausfcfg.yaml        2024-03-30 10:48:53.470630936 +0900
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
--- nrfcfg.yaml.orig    2024-03-30 10:35:10.534612278 +0900
+++ nrfcfg.yaml 2024-03-31 12:34:36.380715909 +0900
@@ -15,8 +15,8 @@
       key: cert/nrf.key # NRF TLS Private key
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
--- nssfcfg.yaml.orig   2024-03-30 10:35:10.535609025 +0900
+++ nssfcfg.yaml        2024-03-31 20:33:17.782753733 +0900
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
--- smfcfg.yaml.orig    2024-03-30 10:35:10.535609025 +0900
+++ smfcfg.yaml 2024-03-31 20:43:06.245192282 +0900
@@ -19,7 +19,7 @@
   snssaiInfos: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this AMF
     - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
         sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-        sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
+        sd: 000001 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
       dnnInfos: # DNN information list
         - dnn: internet # Data Network Name
           dns: # the IP address of DNS
@@ -34,26 +34,26 @@
             ipv4: 8.8.8.8
             ipv6: 2001:4860:4860::8888
   plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
   pfcp: # the IP address of N4 interface on this SMF (PFCP)
     # addr config is deprecated in smf config v1.0.3, please use the following config
-    nodeID: 127.0.0.1 # the Node ID of this SMF
-    listenAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
-    externalAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    nodeID: 192.168.0.141 # the Node ID of this SMF
+    listenAddr: 192.168.0.141 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    externalAddr: 192.168.0.141 # the IP/FQDN of N4 interface on this SMF (PFCP)
   userplaneInformation: # list of userplane information
     upNodes: # information of userplane node (AN or UPF)
       gNB1: # the name of the node
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
@@ -72,7 +72,7 @@
         interfaces: # Interface list for this UPF
           - interfaceType: N3 # the type of the interface (N3 or N9)
             endpoints: # the IP address of this N3/N9 interface on this UPF
-              - 127.0.0.8
+              - 192.168.0.142
             networkInstances: # Data Network Name (DNN)
               - internet
     links: # the topology graph of userplane, A and B represent the two nodes of each link
@@ -93,6 +93,7 @@
   urrPeriod: 10 # default usage report period in seconds
   urrThreshold: 1000 # default usage report threshold in bytes
   requestedUnit: 1000
+  ulcl: false
 logger: # log output setting
   enable: true # true or false
   level: info # how detailed to output, value: trace, debug, info, warn, error, fatal, panic
```

<a id="changes_up"></a>

### Changes in configuration files of free5GC 5GC U-Plane

- `free5gc/config/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2024-03-30 11:20:35.082415729 +0900
+++ upfcfg.yaml 2024-03-31 20:46:35.544422571 +0900
@@ -3,8 +3,8 @@
 
 # The listen IP and nodeID of the N4 interface on this UPF (Can't set to 0.0.0.0)
 pfcp:
-  addr: 127.0.0.8   # IP addr for listening
-  nodeID: 127.0.0.8 # External IP or FQDN can be reached
+  addr: 192.168.0.142   # IP addr for listening
+  nodeID: 192.168.0.142 # External IP or FQDN can be reached
   retransTimeout: 1s # retransmission timeout
   maxRetrans: 3 # the max number of retransmission
 
@@ -13,7 +13,7 @@
   # The IP list of the N3/N9 interfaces on this UPF
   # If there are multiple connection, set addr to 0.0.0.0 or list all the addresses
   ifList:
-    - addr: 127.0.0.8
+    - addr: 192.168.0.142
       type: N3
       # name: upf.5gc.nctu.me
       # ifname: gtpif
```

<a id="changes_srs"></a>

### Changes in configuration files of srsRAN 5G ZMQ UE / RAN

<a id="changes_ran"></a>

#### Changes in configuration files of RAN

See [here](https://github.com/s5uishida/build_srsran_5g_zmq#create-the-configuration-file-of-gnodeb) for the original files.

- `srsRAN_Project/build/apps/gnb/gnb_zmq.yaml`
```diff
--- gnb_zmq.yaml.orig   2024-04-23 20:02:09.000000000 +0900
+++ gnb_zmq.yaml        2024-05-12 16:13:13.243911043 +0900
@@ -3,13 +3,21 @@
 # To run the srsRAN Project gNB with this config, use the following command: 
 #   sudo ./gnb -c gnb_zmq.yaml
 
+gnb_id: 0x19B
+
+slicing:
+  - sst: 1
+    sd: 1
+
 amf:
-  addr: 10.53.1.2                  # The address or hostname of the AMF.
-  bind_addr: 10.53.1.1             # A local IP that the gNB binds to for traffic from the AMF.
+  addr: 192.168.0.141                  # The address or hostname of the AMF.
+#  bind_addr: 10.53.1.1             # A local IP that the gNB binds to for traffic from the AMF.
+  n2_bind_addr: 192.168.0.121          # Optional TEXT. Sets local IP address to bind for N2 interface. Format: IPV4 or IPV6 IP address.
+  n3_bind_addr: 192.168.0.121          # Optional TEXT. Sets local IP address to bind for N3 interface. Format: IPV4 or IPV6 IP address.
 
 ru_sdr:
   device_driver: zmq                # The RF driver name.
-  device_args: tx_port=tcp://127.0.0.1:2000,rx_port=tcp://127.0.0.1:2001,base_srate=23.04e6 # Optionally pass arguments to the selected RF driver.
+  device_args: tx_port=tcp://192.168.0.121:2000,rx_port=tcp://192.168.0.122:2001,base_srate=23.04e6 # Optionally pass arguments to the selected RF driver.
   srate: 23.04                      # RF sample rate might need to be adjusted according to selected bandwidth.
   tx_gain: 75                       # Transmit gain of the RF might need to adjusted to the given situation.
   rx_gain: 75                       # Receive gain of the RF might need to adjusted to the given situation.
@@ -20,7 +28,7 @@
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
- free5GC v3.4.1 (2024.03.28) - https://free5gc.org/guide/
- srsRAN Project (RAN) (2024.03.25) - https://github.com/s5uishida/build_srsran_5g_zmq
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

NF_LIST="nrf amf smf udr pcf udm nssf ausf chf"

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
# cd free5gc
# bin/upf
```

<a id="run_ran"></a>

### Run srsRAN 5G ZMQ RAN

Run srsRAN 5G ZMQ RAN and connect to free5GC 5GC.
```
# cd srsRAN_Project/build/apps/gnb
# ./gnb -c gnb_zmq.yaml

The PRACH detector will not meet the performance requirements with the configuration {Format 0, ZCZ 0, SCS 1.25kHz, Rx ports 1}.
Lower PHY in executor blocking mode.

--== srsRAN gNB (commit 2f90c8b60) ==--

Connecting to AMF on 192.168.0.141:38412
Available radio types: zmq.
Cell pci=1, bw=20 MHz, dl_arfcn=368500 (n3), dl_freq=1842.5 MHz, dl_ssb_arfcn=368410, ul_freq=1747.5 MHz

==== gNodeB started ===
Type <t> to view trace
```
The free5GC C-Plane log when executed is as follows.
```
2024-03-31T21:19:49.290135927+09:00 [INFO][AMF][Ngap] [AMF] SCTP Accept from: 192.168.0.121:55190
2024-03-31T21:19:49.290595275+09:00 [INFO][AMF][Ngap] Create a new NG connection for: 192.168.0.121:55190
2024-03-31T21:19:49.291298109+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:55190] Handle NGSetupRequest
2024-03-31T21:19:49.291326074+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:55190] Send NG-Setup response
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
Random Access Transmission: prach_occasion=0, preamble_index=0, ra-rnti=0x39, tti=174
Random Access Complete.     c-rnti=0x4601, ta=0
RRC Connected
PDU Session Establishment successful. IP: 10.60.0.1
RRC NR reconfiguration successful.
```
The free5GC C-Plane log when executed is as follows.
```
2024-03-31T21:21:40.603006399+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:55190] Handle InitialUEMessage
2024-03-31T21:21:40.603251293+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:55190] New RanUe [RanUeNgapID:0][AmfUeNgapID:1]
2024-03-31T21:21:40.603447323+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:55190] 5GSMobileIdentity ["SUCI":"suci-0-001-01-0000-0-0-0000000000", err: <nil>]
2024-03-31T21:21:40.604190732+09:00 [INFO][AMF][CTX] New AmfUe [supi:][guti:00101cafe0000000001]
2024-03-31T21:21:40.604385041+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2024-03-31T21:21:40.604548318+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Handle Registration Request
2024-03-31T21:21:40.604735859+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] RegistrationType: Initial Registration
2024-03-31T21:21:40.604892934+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] MobileIdentity5GS: SUCI[suci-0-001-01-0000-0-0-0000000000]
2024-03-31T21:21:40.605083580+09:00 [INFO][AMF][Gmm] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2024-03-31T21:21:40.605274399+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Authentication procedure
2024-03-31T21:21:40.606068388+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.608725402+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.611616497+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.615833109+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:40.617020639+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2024-03-31T21:21:40.617668059+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.619187547+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.623241191+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.625127658+09:00 [INFO][AUSF][UeAuth] HandleUeAuthPostRequest
2024-03-31T21:21:40.625415155+09:00 [INFO][AUSF][UeAuth] Serving network authorized
2024-03-31T21:21:40.626334027+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.627693117+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.630433332+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.631695354+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:40.633284655+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2024-03-31T21:21:40.634042408+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.635166898+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.638653966+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.640429084+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2024-03-31T21:21:40.641627088+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.643190099+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.646326944+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.647139802+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2024-03-31T21:21:40.647324563+09:00 [INFO][UDM][Suci] scheme 0
2024-03-31T21:21:40.647810051+09:00 [INFO][UDM][Suci] SUPI type is IMSI
http://127.0.0.10:8000
2024-03-31T21:21:40.648343342+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.650218096+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.652696989+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.654412031+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:40.655539494+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=UDM&target-nf-type=UDR |
2024-03-31T21:21:40.657377940+09:00 [INFO][UDR][DataRepo] Handle QueryAuthSubsData
2024-03-31T21:21:40.659291785+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T21:21:40.660419201+09:00 [INFO][UDM][UEAU] Nil Op
2024-03-31T21:21:40.662208603+09:00 [INFO][UDR][DataRepo] Handle ModifyAuthentication
2024-03-31T21:21:40.664865868+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T21:21:40.665452447+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2024-03-31T21:21:40.665988966+09:00 [INFO][AUSF][UeAuth] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2024-03-31T21:21:40.666223724+09:00 [INFO][AUSF][UeAuth] Use 5G AKA auth method
2024-03-31T21:21:40.666261296+09:00 [INFO][AUSF][5gAka] XresStar = 6362633836333766346565353130656535376537633135623664633863356265
2024-03-31T21:21:40.666379410+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2024-03-31T21:21:40.666868925+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Send Authentication Request
2024-03-31T21:21:40.666994566+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:55190] Send Downlink Nas Transport
2024-03-31T21:21:40.667412889+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Start T3560 timer
2024-03-31T21:21:40.719962166+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:55190] Handle UplinkNASTransport
2024-03-31T21:21:40.720158315+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:55190] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2024-03-31T21:21:40.720458446+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2024-03-31T21:21:40.720783499+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Handle Authentication Response
2024-03-31T21:21:40.720933683+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Stop T3560 timer
2024-03-31T21:21:40.721699287+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.723461139+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.728089622+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.731157059+09:00 [INFO][AUSF][5gAka] Auth5gAkaComfirmRequest
2024-03-31T21:21:40.731308096+09:00 [INFO][AUSF][5gAka] res*: 6362633836333766346565353130656535376537633135623664633863356265
Xres*: 6362633836333766346565353130656535376537633135623664633863356265
2024-03-31T21:21:40.732256697+09:00 [INFO][AUSF][5gAka] 5G AKA confirmation succeeded
2024-03-31T21:21:40.733090521+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.734339132+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.737812101+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.739635202+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2024-03-31T21:21:40.740806812+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.742428279+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.745671813+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.747730217+09:00 [INFO][UDR][DataRepo] Handle CreateAuthenticationStatus
2024-03-31T21:21:40.748977190+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2024-03-31T21:21:40.749349345+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2024-03-31T21:21:40.749922575+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2024-03-31T21:21:40.750389657+09:00 [INFO][AMF][Gmm] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2024-03-31T21:21:40.750485705+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Security Mode Command
2024-03-31T21:21:40.750748791+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:55190] Send Downlink Nas Transport
2024-03-31T21:21:40.751213436+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3560 timer
2024-03-31T21:21:40.803226298+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:55190] Handle UplinkNASTransport
2024-03-31T21:21:40.803610683+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:55190] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2024-03-31T21:21:40.803868889+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2024-03-31T21:21:40.804023420+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Security Mode Complete
2024-03-31T21:21:40.804194889+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3560 timer
2024-03-31T21:21:40.804426178+09:00 [INFO][AMF][Gmm] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2024-03-31T21:21:40.804654769+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle InitialRegistration
2024-03-31T21:21:40.805449786+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.807338049+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.810054030+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.811295137+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:40.812865399+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-03-31T21:21:40.813639952+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.815102017+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.818798635+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.820536781+09:00 [INFO][UDM][SDM] Handle GetNssai
2024-03-31T21:21:40.821453241+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.822948586+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.826367330+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.827885449+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2024-03-31T21:21:40.828890711+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data |
2024-03-31T21:21:40.829299676+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T21:21:40.829848268+09:00 [INFO][AMF][Gmm] RequestedNssai: &{Iei:47 Len:5 Buffer:[4 1 0 0 1]}
2024-03-31T21:21:40.830049417+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:000001}, HomeSnssai: <nil>
2024-03-31T21:21:40.831267883+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.833031910+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.835677975+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.837020961+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:40.838429100+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-03-31T21:21:40.839093779+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.840910742+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.845082215+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.848301810+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2024-03-31T21:21:40.848828367+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2024-03-31T21:21:40.849601831+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.850990251+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.854579089+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.856526214+09:00 [INFO][UDR][DataRepo] Handle CreateAmfContext3gpp
2024-03-31T21:21:40.857924085+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2024-03-31T21:21:40.858303870+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2024-03-31T21:21:40.859307511+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.860941924+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.864699657+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.866471128+09:00 [INFO][UDM][SDM] Handle GetAmData
2024-03-31T21:21:40.867397371+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.868751386+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.872060978+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.873921770+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2024-03-31T21:21:40.874713621+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T21:21:40.875309432+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T21:21:40.876655617+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.878307531+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.881941573+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.883913816+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2024-03-31T21:21:40.884740578+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.886075951+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.889425265+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.891195323+09:00 [INFO][UDR][DataRepo] Handle QuerySmfSelectData
2024-03-31T21:21:40.892074140+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data |
2024-03-31T21:21:40.892694684+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T21:21:40.894281523+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.895904703+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.899511922+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.901160261+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2024-03-31T21:21:40.901909675+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.903292627+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.906797845+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.908316650+09:00 [INFO][UDR][DataRepo] Handle QuerySmfRegList
2024-03-31T21:21:40.909194507+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations |
2024-03-31T21:21:40.909603785+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2024-03-31T21:21:40.913258487+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.915151000+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.918713639+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.920384437+09:00 [INFO][UDM][SDM] Handle Subscribe
2024-03-31T21:21:40.921301435+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.923030409+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.926424530+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.928362293+09:00 [INFO][UDR][DataRepo] Handle CreateSdmSubscriptions
2024-03-31T21:21:40.928632419+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2024-03-31T21:21:40.928981706+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2024-03-31T21:21:40.930338362+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.931932105+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.934830573+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.936154041+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:40.938607042+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2024-03-31T21:21:40.940041170+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.942559557+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.946461898+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.949070526+09:00 [INFO][PCF][AmPol] Handle AM Policy Create Request
2024-03-31T21:21:40.949874058+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.951653255+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.954371100+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.955699687+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:40.956815157+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2024-03-31T21:21:40.957464788+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.958828611+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.962022113+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.963628610+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdAmDataGet
2024-03-31T21:21:40.964631324+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2024-03-31T21:21:40.965713354+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.967427541+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.969948031+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.971293139+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:40.972789090+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe00%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2024-03-31T21:21:40.973589017+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:40.975162881+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:40.978889633+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:40.980497004+09:00 [INFO][AMF][Comm] Handle AMF Status Change Subscribe Request
2024-03-31T21:21:40.980885947+09:00 [INFO][AMF][Comm] new AMF Status Subscription[1]
2024-03-31T21:21:40.981108070+09:00 [INFO][AMF][GIN] | 201 |       127.0.0.1 | POST    | /namf-comm/v1/subscriptions |
2024-03-31T21:21:40.981652432+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2024-03-31T21:21:40.982101565+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Registration Accept
2024-03-31T21:21:40.982340196+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:55190] Send Initial Context Setup Request
2024-03-31T21:21:40.983756727+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3550 timer
2024-03-31T21:21:41.057855672+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:55190] Handle InitialContextSetupResponse
2024-03-31T21:21:41.058102780+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:55190] Handle InitialContextSetupResponse (RAN UE NGAP ID: 0)
2024-03-31T21:21:41.265271124+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:55190] Handle UplinkNASTransport
2024-03-31T21:21:41.265596045+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:55190] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2024-03-31T21:21:41.265853145+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2024-03-31T21:21:41.266001507+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Registration Complete
2024-03-31T21:21:41.266190765+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3550 timer
2024-03-31T21:21:41.266358023+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Configuration Update Command
2024-03-31T21:21:41.266578125+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:55190] Send Downlink Nas Transport
2024-03-31T21:21:41.267317812+09:00 [INFO][AMF][Gmm] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2024-03-31T21:21:41.268045525+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:55190] Handle UplinkNASTransport
2024-03-31T21:21:41.268203266+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:55190] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2024-03-31T21:21:41.268376608+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2024-03-31T21:21:41.268589433+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle UL NAS Transport
2024-03-31T21:21:41.268751813+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2024-03-31T21:21:41.268924152+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:000001}, dnn: internet]
2024-03-31T21:21:41.270263883+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.271985131+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.274825907+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.275984226+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:41.277108825+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2024-03-31T21:21:41.277680096+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.279118012+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.282317952+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.284067277+09:00 [INFO][NSSF][NsSel] Handle NSSelectionGet
2024-03-31T21:21:41.284757979+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=b5af6648-734d-4387-a061-112cef1b0f2c&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2024-03-31T21:21:41.286095744+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.287663006+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.290826157+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.292111043+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:41.293559950+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=area1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T21:21:41.294145389+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.295767867+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.299132402+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.301223901+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2024-03-31T21:21:41.302035761+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2024-03-31T21:21:41.302245090+09:00 [INFO][SMF][CTX] UrrPeriod: 10s
2024-03-31T21:21:41.302402453+09:00 [INFO][SMF][CTX] UrrThreshold: 1000
2024-03-31T21:21:41.303158221+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.304550195+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.307381851+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.308949371+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:41.312573016+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2024-03-31T21:21:41.313145319+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Send NF Discovery Serving UDM Successfully
2024-03-31T21:21:41.313696877+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.314985763+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.318457383+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.320176207+09:00 [INFO][UDM][SDM] Handle GetSmData
2024-03-31T21:21:41.321274984+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.322699636+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.325982626+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.326440396+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"000001"}]
2024-03-31T21:21:41.327838661+09:00 [INFO][UDR][DataRepo] Handle QuerySmData
2024-03-31T21:21:41.329051169+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2024-03-31T21:21:41.329648021+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2024-03-31T21:21:41.330258235+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2024-03-31T21:21:41.331473685+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.332878037+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.335497628+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.336548851+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:41.337913307+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=b5af6648-734d-4387-a061-112cef1b0f2c&target-nf-type=AMF |
2024-03-31T21:21:41.338476405+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2024-03-31T21:21:41.338783286+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.60.0.1
2024-03-31T21:21:41.339082761+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2024-03-31T21:21:41.339195054+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Allocated PDUAdress[10.60.0.1]
2024-03-31T21:21:41.339700257+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.341236429+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.343869331+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.345735601+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:41.347175239+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=SMF&target-nf-type=PCF |
2024-03-31T21:21:41.348099479+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.349446535+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.353047949+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.354613306+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2024-03-31T21:21:41.355343634+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.356862351+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.360110654+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.362040365+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdSmDataGet
2024-03-31T21:21:41.363500590+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2024-03-31T21:21:41.366882033+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.368471445+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.371756937+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.373243922+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataGet
2024-03-31T21:21:41.374288568+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/application-data/influenceData?dnns=internet&internal-Group-Ids=&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D&supis=imsi-001010000000000 |
2024-03-31T21:21:41.374707025+09:00 [INFO][PCF][SMpolicy] Matched [0] trafficInfluDatas from UDR
2024-03-31T21:21:41.375680802+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.377488047+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.380780861+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.382639000+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataSubsToNotifyPost
2024-03-31T21:21:41.382972752+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/application-data/influenceData/subs-to-notify |
2024-03-31T21:21:41.384147928+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.385930471+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.388558955+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.389761645+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:41.390628710+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=BSF |
2024-03-31T21:21:41.391526861+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2024-03-31T21:21:41.392552418+09:00 [INFO][SMF][PduSess] CHF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2024-03-31T21:21:41.393413909+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.395102245+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.397668801+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.399110251+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T21:21:41.400137817+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=CHF |
2024-03-31T21:21:41.400650029+09:00 [INFO][SMF][Charging] Handle SendConvergedChargingRequest
2024-03-31T21:21:41.401094897+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.403515354+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.408562762+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.412923614+09:00 [INFO][CHF][ChargingPost] HandleChargingdataInitial
2024-03-31T21:21:41.413179000+09:00 [INFO][CHF][ChargingPost] SMF charging event
2024-03-31T21:21:41.413553790+09:00 [ERRO][CHF][ChargingPost] Charging gateway fail to send CDR to billing domain dial tcp 127.0.0.1:2122: connect: connection refused
2024-03-31T21:21:41.413651791+09:00 [INFO][CHF][ChargingPost] Open CDR for UE imsi-001010000000000
2024-03-31T21:21:41.413795940+09:00 [INFO][CHF][ChargingPost] NewChfUe imsi-001010000000000
2024-03-31T21:21:41.414249218+09:00 [INFO][CHF][GIN] | 201 |       127.0.0.1 | POST    | /nchf-convergedcharging/v3/chargingdata |
2024-03-31T21:21:41.415127944+09:00 [INFO][SMF][Charging] Send Charging Data Request[Init] successfully
2024-03-31T21:21:41.415306877+09:00 [ERRO][SMF][CTX] No default data path
2024-03-31T21:21:41.415517176+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Has no pre-config route. Has no default path
2024-03-31T21:21:41.415840513+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2024-03-31T21:21:41.416236674+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2024-03-31T21:21:41.417078161+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2024-03-31T21:21:41.419305377+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:55190] Handle UplinkNASTransport
2024-03-31T21:21:41.419525920+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:55190] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2024-03-31T21:21:41.419774439+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2024-03-31T21:21:41.420104714+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Configuration Update Complete
2024-03-31T21:21:41.421447687+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2024-03-31T21:21:41.423313348+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.425117740+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.428747150+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.432181583+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2024-03-31T21:21:41.432432097+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:55190] Send PDU Session Resource Setup Request
2024-03-31T21:21:41.433397981+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2024-03-31T21:21:41.547935081+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:55190] Handle PDUSessionResourceSetupResponse
2024-03-31T21:21:41.548165299+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:55190] Handle PDUSessionResourceSetupResponse (RAN UE NGAP ID: 0)
2024-03-31T21:21:41.549014337+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T21:21:41.550655817+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T21:21:41.554604249+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T21:21:41.556428942+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2024-03-31T21:21:41.559212484+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2024-03-31T21:21:41.559507679+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:09f45845-bcfa-419c-bc1d-28106180e27f/modify |
```
The free5GC U-Plane log when executed is as follows.
```
2024-03-31T21:21:41.348273647+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionEstablishmentRequest
2024-03-31T21:21:41.348338571+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805][CPNodeID:192.168.0.141][CPSEID:0x1][UPSEID:0x1] New session
2024-03-31T21:21:41.349964525+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:1 period:10000000000}]
2024-03-31T21:21:41.350603303+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:2 period:10000000000}]
2024-03-31T21:21:41.488859140+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionModificationRequest
```
The result of `ip addr show` on VM4 (UE) is as follows.
```
# ip addr show
...
6: tun_srsue: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
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
PING google.com (142.251.42.174) from 10.60.0.1 tun_srsue: 56(84) bytes of data.
64 bytes from 142.251.42.174: icmp_seq=1 ttl=61 time=69.0 ms
64 bytes from 142.251.42.174: icmp_seq=2 ttl=61 time=58.8 ms
64 bytes from 142.251.42.174: icmp_seq=3 ttl=61 time=77.6 ms
```
- Run `tcpdump` on VM2 (U-Plane)
```
# tcpdump -i upfgtp -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on upfgtp, link-type RAW (Raw IP), snapshot length 262144 bytes
21:21:43.979814 IP 10.60.0.1 > 142.251.42.174: ICMP echo request, id 3, seq 1, length 64
21:21:43.999176 IP 142.251.42.174 > 10.60.0.1: ICMP echo reply, id 3, seq 1, length 64
21:21:44.968218 IP 10.60.0.1 > 142.251.42.174: ICMP echo request, id 3, seq 2, length 64
21:21:44.987581 IP 142.251.42.174 > 10.60.0.1: ICMP echo reply, id 3, seq 2, length 64
21:21:45.994297 IP 10.60.0.1 > 142.251.42.174: ICMP echo request, id 3, seq 3, length 64
21:21:46.010894 IP 142.251.42.174 > 10.60.0.1: ICMP echo reply, id 3, seq 3, length 64
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
21:28:56.378267 IP 10.60.0.1.34690 > 142.250.207.46.80: Flags [S], seq 2206941537, win 64240, options [mss 1460,sackOK,TS val 722769483 ecr 0,nop,wscale 7], length 0
21:28:56.429607 IP 142.250.207.46.80 > 10.60.0.1.34690: Flags [S.], seq 9984001, ack 2206941538, win 65535, options [mss 1460], length 0
21:28:56.499559 IP 10.60.0.1.34690 > 142.250.207.46.80: Flags [.], ack 1, win 64240, length 0
21:28:56.499559 IP 10.60.0.1.34690 > 142.250.207.46.80: Flags [P.], seq 1:75, ack 1, win 64240, length 74: HTTP: GET / HTTP/1.1
21:28:56.499746 IP 142.250.207.46.80 > 10.60.0.1.34690: Flags [.], ack 75, win 65535, length 0
21:28:56.578398 IP 142.250.207.46.80 > 10.60.0.1.34690: Flags [P.], seq 1:774, ack 75, win 65535, length 773: HTTP: HTTP/1.1 301 Moved Permanently
21:28:56.650298 IP 10.60.0.1.34690 > 142.250.207.46.80: Flags [.], ack 774, win 63467, length 0
21:28:56.650511 IP 10.60.0.1.34690 > 142.250.207.46.80: Flags [F.], seq 75, ack 774, win 63467, length 0
21:28:56.650632 IP 142.250.207.46.80 > 10.60.0.1.34690: Flags [.], ack 76, win 65535, length 0
21:28:56.678567 IP 142.250.207.46.80 > 10.60.0.1.34690: Flags [F.], seq 774, ack 76, win 65535, length 0
21:28:56.726835 IP 10.60.0.1.34690 > 142.250.207.46.80: Flags [.], ack 775, win 63467, length 0
```
You could now create the end-to-end TUN interface on the DN and send any packets on the network.

---
In investigating 5G SA, I have built a simulation environment and can now use a very useful system for investigating 5GC and MEC of 5G SA mobile network. I would like to thank the excellent developers and all the contributors of free5GC, srsRAN Project and srsRAN 4G.

<a id="changelog"></a>

## Changelog (summary)

- [2024.03.31] Updated to free5GC v3.4.1 (2024.03.28).
- [2023.11.02] Updated `gnb_zmq.yaml`.
- [2023.10.21] Updated `gnb_zmq.yaml` according to srsRAN_Project 23.10 (2023.10.20).
- [2023.10.12] Updated free5GC v3.3.0 (2023.10.11).
- [2023.10.11] Updated srsRAN_Project (2023.09.20).
- [2023.08.26] Initial release.

# free5GC 5GC & srsRAN 5G with ZeroMQ UE / RAN Sample Configuration
srsRAN_Project and srsRAN_4G software suites include a virtual radio which uses the ZeroMQ networking library to transfer radio samples between applications.
Therefore, in order to use U-Plane's DN (Data Network) as a trial, I built a simulation environment for the 5GC mobile network.
This briefly describes the overall and configuration files in the Virtualbox VM environment.

---

<a id="conf_list"></a>

## List of Sample Configurations

1. One SMF, one UPF and one DNN (this article)
2. [One SMF, Multiple UPFs and DNNs](https://github.com/s5uishida/free5gc_ueransim_sample_config)
3. [Select nearby UPF according to the connected gNodeB](https://github.com/s5uishida/free5gc_ueransim_nearby_upf_sample_config)
4. [Select UPF based on S-NSSAI](https://github.com/s5uishida/free5gc_ueransim_snssai_upf_sample_config)
5. [ULCL(Uplink Classifier)](https://github.com/s5uishida/free5gc_ueransim_ulcl_sample_config)
6. [ULCL with one I-UPF and two PSA-UPFs](https://github.com/s5uishida/free5gc_ueransim_ulcl_2_sample_config)
7. [VPP-UPF with DPDK](https://github.com/s5uishida/free5gc_ueransim_vpp_upf_dpdk_sample_config)

---

<a id="misc"></a>

## Miscellaneous Notes

- [Install MongoDB 6.0 and free5GC WebUI](https://github.com/s5uishida/free5gc_install_mongodb6_webui)
- [Install MongoDB 4.4.18 on Ubuntu 20.04 for Raspberry Pi 4B](https://github.com/s5uishida/install_mongodb_on_ubuntu_for_rp4b)
- [Build srsRAN_Project 5G RAN with ZeroMQ](https://github.com/s5uishida/build_srsran_5g_zmq)
- [Build srsRAN 4G UE / RAN with ZeroMQ by disabling RF plugins](https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins)
- [A Note for 5G SUCI Profile A/B Scheme](https://github.com/s5uishida/note_5g_suci_profile_ab)
- [A Note for Enabling NetworkInstance IE Encoding for free5GC v3.3.0](https://github.com/s5uishida/enable_network_instance_encoding_free5gc_v3_3_0)

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
- 5GC - free5GC v3.3.0 (2023.10.11) - https://github.com/free5gc/free5gc
- RAN - srsRAN Project (2023.09.20) - https://github.com/srsran/srsRAN_Project
- UE (NR-UE) - srsRAN 4G (2023.06.19) - https://github.com/srsran/srsRAN_4G

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
- free5GC v3.3.0 (2023.10.11) - https://free5gc.org/guide/
- srsRAN Project (RAN) (2023.09.20) - https://github.com/s5uishida/build_srsran_5g_zmq
- srsRAN 4G (UE) (2023.06.19) - https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins

<a id="changes_cp"></a>

### Changes in configuration files of free5GC 5GC C-Plane

The combination of DNN and S-NSSAI parameters can be used in the logic that selects UPF as the connection destination by PFCP.

- DNN
- S-NSSAI

For the sake of simplicity, This time, only DNN will be changed. S-NSSAI is fixed as `SST=1` and `SD=000001`.

- `free5gc/config/amfcfg.yaml`

Please deal with [this](https://github.com/free5gc/free5gc/pull/494) as well.
```diff
--- amfcfg.yaml.orig    2023-08-26 22:40:43.591429760 +0900
+++ amfcfg.yaml 2023-08-26 22:52:47.646160715 +0900
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
--- ausfcfg.yaml.orig   2023-08-26 22:40:43.591429760 +0900
+++ ausfcfg.yaml        2023-08-26 22:52:55.064282537 +0900
@@ -15,8 +15,8 @@
     - nausf-auth # Nausf_UEAuthentication service
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
   plmnSupportList: # the PLMNs (Public Land Mobile Network) list supported by this AUSF
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
     - mcc: 123 # Mobile Country Code (3 digits string, digit: 0~9)
       mnc: 45  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   groupId: ausfGroup001 # ID for the group of the AUSF
```
- `free5gc/config/nrfcfg.yaml`
```diff
--- nrfcfg.yaml.orig    2023-08-26 22:40:43.591429760 +0900
+++ nrfcfg.yaml 2023-08-26 22:53:05.952461341 +0900
@@ -14,8 +14,8 @@
       pem: cert/nrf.pem # NRF TLS Certificate
       key: cert/nrf.key # NRF TLS Private key
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
--- nssfcfg.yaml.orig   2023-08-26 22:40:43.592429779 +0900
+++ nssfcfg.yaml        2023-08-26 22:53:14.448600862 +0900
@@ -17,15 +17,15 @@
     - nnssf-nssaiavailability # Nnssf_NSSAIAvailability service
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
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
--- smfcfg.yaml.orig    2023-08-26 22:40:43.592429779 +0900
+++ smfcfg.yaml 2023-08-26 22:53:53.489241962 +0900
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
             networkInstances:  # Data Network Name (DNN)
               - internet
     links: # the topology graph of userplane, A and B represent the two nodes of each link
@@ -89,8 +89,10 @@
     expireTime: 16s   # default is 6 seconds
     maxRetryTimes: 3 # the max number of retransmission
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
-  #urrPeriod: 10 # default usage report period in seconds
-  #urrThreshold: 1000 # default usage report threshold in bytes
+  urrPeriod: 10 # default usage report period in seconds
+  urrThreshold: 1000 # default usage report threshold in bytes
+  ulcl: false
+  nwInstFqdnEncoding: true
 
 logger: # log output setting
   enable: true # true or false
```

<a id="changes_up"></a>

### Changes in configuration files of free5GC 5GC U-Plane

- `free5gc/config/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2023-08-26 22:47:59.019947847 +0900
+++ upfcfg.yaml 2023-08-26 23:39:19.192682344 +0900
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
--- gnb_zmq.yaml.orig   2023-07-07 00:32:21.000000000 +0900
+++ gnb_zmq.yaml        2023-08-26 22:54:44.877078203 +0900
@@ -3,13 +3,19 @@
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
+  bind_addr: 192.168.0.121             # A local IP that the gNB binds to for traffic from the AMF.
 
 ru_sdr:
   device_driver: zmq                # The RF driver name.
-  device_args: tx_port=tcp://127.0.0.1:2000,rx_port=tcp://127.0.0.1:2001,base_srate=11.52e6 # Optionally pass arguments to the selected RF driver.
+  device_args: tx_port=tcp://192.168.0.121:2000,rx_port=tcp://192.168.0.122:2001,base_srate=11.52e6 # Optionally pass arguments to the selected RF driver.
   srate: 11.52                      # RF sample rate might need to be adjusted according to selected bandwidth.
   tx_gain: 75                       # Transmit gain of the RF might need to adjusted to the given situation.
   rx_gain: 75                       # Receive gain of the RF might need to adjusted to the given situation.
@@ -20,7 +26,7 @@
   channel_bandwidth_MHz: 10         # Bandwith in MHz. Number of PRBs will be automatically derived.
   common_scs: 15                    # Subcarrier spacing in kHz used for data.
   plmn: "00101"                     # PLMN broadcasted by the gNB.
-  tac: 7                            # Tracking area code (needs to match the core configuration).
+  tac: 1                            # Tracking area code (needs to match the core configuration).
   pdcch:
     dedicated:
       ss2_type: common              # Search Space type, has to be set to common
```

<a id="changes_ue"></a>

#### Changes in configuration files of UE

See [here](https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins#create-the-configuration-file-of-nr-ue) for the original files.

- `srsRAN_4G/build/srsue/ue_zmq.conf`
```diff
--- ue_zmq.conf.orig    2023-04-24 18:36:33.000000000 +0900
+++ ue_zmq.conf 2023-08-26 22:55:38.425314807 +0900
@@ -6,7 +6,7 @@
 nof_antennas = 1
 
 device_name = zmq
-device_args = tx_port=tcp://127.0.0.1:2001,rx_port=tcp://127.0.0.1:2000,base_srate=11.52e6
+device_args = tx_port=tcp://192.168.0.122:2001,rx_port=tcp://192.168.0.121:2000,base_srate=11.52e6
 
 [rat.eutra]
 dl_earfcn = 2850
@@ -32,9 +32,9 @@
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
@@ -42,11 +42,16 @@
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
- free5GC v3.3.0 (2023.10.11) - https://free5gc.org/guide/
- srsRAN Project (RAN) (2023.09.20) - https://github.com/s5uishida/build_srsran_5g_zmq
- srsRAN 4G (UE) (2023.06.19) - https://github.com/s5uishida/build_srsran_4g_zmq_disable_rf_plugins

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
Lower PHY in executor blocking mode.

--== srsRAN gNB (commit 5e6f50a20) ==--

Connecting to AMF on 192.168.0.141:38412
Available radio types: zmq.
Cell pci=1, bw=10 MHz, dl_arfcn=368500 (n3), dl_freq=1842.5 MHz, dl_ssb_arfcn=368410, ul_freq=1747.5 MHz

==== gNodeB started ===
Type <t> to view trace
```
The free5GC C-Plane log when executed is as follows.
```
2023-10-12T21:15:19.460678684+09:00 [INFO][AMF][Ngap] [AMF] SCTP Accept from: 192.168.0.121:41703
2023-10-12T21:15:19.462412098+09:00 [INFO][AMF][Ngap] Create a new NG connection for: 192.168.0.121:41703
2023-10-12T21:15:19.463023071+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:41703] Handle NGSetupRequest
2023-10-12T21:15:19.463061473+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:41703] Send NG-Setup response
```

<a id="run_ue"></a>

### Run srsRAN 5G ZMQ UE

Run srsRAN 5G ZMQ UE and connect to free5GC 5GC.
```
# cd srsRAN_4G/build/srsue
# ./src/srsue ue_zmq.conf
Reading configuration file ue_zmq.conf...

Built in Release mode using commit fa56836b1 on branch master.

Opening 1 channels in RF device=zmq with args=tx_port=tcp://192.168.0.122:2001,rx_port=tcp://192.168.0.121:2000,base_srate=11.52e6
Supported RF device list: zmq file
CHx base_srate=11.52e6
Current sample rate is 1.92 MHz with a base rate of 11.52 MHz (x6 decimation)
CH0 rx_port=tcp://192.168.0.121:2000
CH0 tx_port=tcp://192.168.0.122:2001
Current sample rate is 11.52 MHz with a base rate of 11.52 MHz (x1 decimation)
Current sample rate is 11.52 MHz with a base rate of 11.52 MHz (x1 decimation)
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
2023-10-12T21:16:05.883998522+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:41703] Handle InitialUEMessage
2023-10-12T21:16:05.884301049+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:41703] New RanUe [RanUeNgapID:0][AmfUeNgapID:1]
2023-10-12T21:16:05.884490244+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:41703] 5GSMobileIdentity ["SUCI":"suci-0-001-01-0000-0-0-0000000000", err: <nil>]
2023-10-12T21:16:05.885645264+09:00 [INFO][AMF][CTX] New AmfUe [supi:][guti:00101cafe0000000001]
2023-10-12T21:16:05.885861282+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2023-10-12T21:16:05.886077776+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Handle Registration Request
2023-10-12T21:16:05.886247878+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] RegistrationType: Initial Registration
2023-10-12T21:16:05.886463428+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] MobileIdentity5GS: SUCI[suci-0-001-01-0000-0-0-0000000000]
2023-10-12T21:16:05.886620901+09:00 [INFO][AMF][Gmm] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2023-10-12T21:16:05.886762701+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Authentication procedure
2023-10-12T21:16:05.887612800+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:05.888761582+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2023-10-12T21:16:05.890987976+09:00 [INFO][AUSF][UeAuth] HandleUeAuthPostRequest
2023-10-12T21:16:05.891642696+09:00 [INFO][AUSF][UeAuth] Serving network authorized
2023-10-12T21:16:05.893335723+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:05.894724096+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2023-10-12T21:16:05.896162640+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2023-10-12T21:16:05.896510354+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2023-10-12T21:16:05.896679882+09:00 [INFO][UDM][Suci] scheme 0
2023-10-12T21:16:05.896887830+09:00 [INFO][UDM][Suci] SUPI type is IMSI
http://127.0.0.10:8000
2023-10-12T21:16:05.898001945+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:05.899623564+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=UDM&target-nf-type=UDR |
2023-10-12T21:16:05.900747940+09:00 [INFO][UDR][DataRepo] Handle QueryAuthSubsData
2023-10-12T21:16:05.902615132+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2023-10-12T21:16:05.903599463+09:00 [INFO][UDM][UEAU] Nil Op
2023-10-12T21:16:05.904242198+09:00 [INFO][UDR][DataRepo] Handle ModifyAuthentication
2023-10-12T21:16:05.906259156+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2023-10-12T21:16:05.906805952+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2023-10-12T21:16:05.907165671+09:00 [INFO][AUSF][UeAuth] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2023-10-12T21:16:05.907232119+09:00 [INFO][AUSF][UeAuth] Use 5G AKA auth method
2023-10-12T21:16:05.907266784+09:00 [INFO][AUSF][5gAka] XresStar = 6461636532636433336639306561633666383866366238636231373833636530
2023-10-12T21:16:05.907362973+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2023-10-12T21:16:05.907657763+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Send Authentication Request
2023-10-12T21:16:05.907742473+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:41703] Send Downlink Nas Transport
2023-10-12T21:16:05.908347757+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Start T3560 timer
2023-10-12T21:16:05.953714629+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:41703] Handle UplinkNASTransport
2023-10-12T21:16:05.953964716+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:41703] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2023-10-12T21:16:05.954203350+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2023-10-12T21:16:05.954381846+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Handle Authentication Response
2023-10-12T21:16:05.954589119+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:] Stop T3560 timer
2023-10-12T21:16:05.955516348+09:00 [INFO][AUSF][5gAka] Auth5gAkaComfirmRequest
2023-10-12T21:16:05.955573389+09:00 [INFO][AUSF][5gAka] res*: 6461636532636433336639306561633666383866366238636231373833636530
Xres*: 6461636532636433336639306561633666383866366238636231373833636530
2023-10-12T21:16:05.955714346+09:00 [INFO][AUSF][5gAka] 5G AKA confirmation succeeded
2023-10-12T21:16:05.956445832+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2023-10-12T21:16:05.957137534+09:00 [INFO][UDR][DataRepo] Handle CreateAuthenticationStatus
2023-10-12T21:16:05.958521673+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2023-10-12T21:16:05.959121312+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2023-10-12T21:16:05.959543611+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2023-10-12T21:16:05.960181231+09:00 [INFO][AMF][Gmm] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2023-10-12T21:16:05.960723215+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Security Mode Command
2023-10-12T21:16:05.960959669+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:41703] Send Downlink Nas Transport
2023-10-12T21:16:05.961780897+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3560 timer
2023-10-12T21:16:05.998753224+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:41703] Handle UplinkNASTransport
2023-10-12T21:16:05.998998703+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:41703] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2023-10-12T21:16:05.999299720+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2023-10-12T21:16:05.999472366+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Security Mode Complete
2023-10-12T21:16:05.999619379+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3560 timer
2023-10-12T21:16:05.999786343+09:00 [INFO][AMF][Gmm] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2023-10-12T21:16:05.999969789+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle InitialRegistration
2023-10-12T21:16:06.002416727+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:06.003842531+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2023-10-12T21:16:06.005149108+09:00 [INFO][UDM][SDM] Handle GetNssai
2023-10-12T21:16:06.005937069+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2023-10-12T21:16:06.007075405+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data |
2023-10-12T21:16:06.007530661+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-10-12T21:16:06.008122178+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:000001}, HomeSnssai: <nil>
2023-10-12T21:16:06.009181284+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:06.010747360+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2023-10-12T21:16:06.012207876+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2023-10-12T21:16:06.012513208+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2023-10-12T21:16:06.013470252+09:00 [INFO][UDR][DataRepo] Handle CreateAmfContext3gpp
2023-10-12T21:16:06.015014542+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2023-10-12T21:16:06.015402799+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2023-10-12T21:16:06.016146732+09:00 [INFO][UDM][SDM] Handle GetAmData
2023-10-12T21:16:06.016560472+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2023-10-12T21:16:06.017455060+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-10-12T21:16:06.017934105+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-10-12T21:16:06.018722716+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2023-10-12T21:16:06.019457986+09:00 [INFO][UDR][DataRepo] Handle QuerySmfSelectData
2023-10-12T21:16:06.020295468+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data |
2023-10-12T21:16:06.020735354+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-10-12T21:16:06.021449460+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2023-10-12T21:16:06.021851914+09:00 [INFO][UDR][DataRepo] Handle QuerySmfRegList
2023-10-12T21:16:06.022593101+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations |
2023-10-12T21:16:06.023067996+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2023-10-12T21:16:06.024009791+09:00 [INFO][UDM][SDM] Handle Subscribe
2023-10-12T21:16:06.024757794+09:00 [INFO][UDR][DataRepo] Handle CreateSdmSubscriptions
2023-10-12T21:16:06.025062187+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2023-10-12T21:16:06.025453289+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2023-10-12T21:16:06.026589909+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:06.027954250+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2023-10-12T21:16:06.030241199+09:00 [INFO][PCF][AmPol] Handle AM Policy Create Request
2023-10-12T21:16:06.031122122+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:06.032313788+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2023-10-12T21:16:06.033371254+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdAmDataGet
2023-10-12T21:16:06.034300006+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2023-10-12T21:16:06.035357293+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:06.036397481+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe00%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2023-10-12T21:16:06.036987687+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2023-10-12T21:16:06.037539402+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Registration Accept
2023-10-12T21:16:06.037773345+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:41703] Send Initial Context Setup Request
2023-10-12T21:16:06.039656011+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3550 timer
2023-10-12T21:16:06.069280617+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:41703] Handle InitialContextSetupResponse
2023-10-12T21:16:06.069628097+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:41703] Handle InitialContextSetupResponse (RAN UE NGAP ID: 0)
2023-10-12T21:16:06.197154873+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:41703] Handle UplinkNASTransport
2023-10-12T21:16:06.197520945+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:41703] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2023-10-12T21:16:06.197795189+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2023-10-12T21:16:06.198214438+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Registration Complete
2023-10-12T21:16:06.198428882+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3550 timer
2023-10-12T21:16:06.198635736+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Configuration Update Command
2023-10-12T21:16:06.198811927+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:41703] Send Downlink Nas Transport
2023-10-12T21:16:06.199884459+09:00 [INFO][AMF][Gmm] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2023-10-12T21:16:06.200650262+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:41703] Handle UplinkNASTransport
2023-10-12T21:16:06.200815619+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:41703] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2023-10-12T21:16:06.201015434+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2023-10-12T21:16:06.201231511+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle UL NAS Transport
2023-10-12T21:16:06.201375614+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2023-10-12T21:16:06.201523775+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:000001}, dnn: internet]
2023-10-12T21:16:06.202500441+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:06.203817097+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2023-10-12T21:16:06.205193161+09:00 [INFO][NSSF][NsSel] Handle NSSelectionGet
2023-10-12T21:16:06.205698754+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=b2e5b684-b5e7-4e76-8362-3ec7ee33a073&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2023-10-12T21:16:06.206959052+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:06.208685987+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=area1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2023-10-12T21:16:06.210253583+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2023-10-12T21:16:06.211284294+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2023-10-12T21:16:06.211582710+09:00 [INFO][SMF][CTX] UrrPeriod: 10s
2023-10-12T21:16:06.211787687+09:00 [INFO][SMF][CTX] UrrThreshold: 1000
2023-10-12T21:16:06.212532439+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:06.214036264+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2023-10-12T21:16:06.214583801+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Send NF Discovery Serving UDM Successfully
2023-10-12T21:16:06.215403738+09:00 [INFO][UDM][SDM] Handle GetSmData
2023-10-12T21:16:06.215480898+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"000001"}]
2023-10-12T21:16:06.216067343+09:00 [INFO][UDR][DataRepo] Handle QuerySmData
2023-10-12T21:16:06.217308883+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2023-10-12T21:16:06.218012928+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2023-10-12T21:16:06.218659687+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2023-10-12T21:16:06.219794413+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:06.222345606+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=b2e5b684-b5e7-4e76-8362-3ec7ee33a073&target-nf-type=AMF |
2023-10-12T21:16:06.224721699+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2023-10-12T21:16:06.225064162+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.60.0.1
2023-10-12T21:16:06.225256387+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2023-10-12T21:16:06.225420567+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Allocated PDUAdress[10.60.0.1]
2023-10-12T21:16:06.226328323+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:06.227881009+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=area1&requester-nf-type=SMF&target-nf-type=PCF |
2023-10-12T21:16:06.229425125+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2023-10-12T21:16:06.230476525+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdSmDataGet
2023-10-12T21:16:06.232004300+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D |
2023-10-12T21:16:06.234807283+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataGet
2023-10-12T21:16:06.235489434+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/application-data/influenceData?dnns=internet&internal-Group-Ids=&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22000001%22%7D&supis=imsi-001010000000000 |
2023-10-12T21:16:06.235889508+09:00 [INFO][PCF][SMpolicy] Matched [0] trafficInfluDatas from UDR
2023-10-12T21:16:06.236826386+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataSubsToNotifyPost
2023-10-12T21:16:06.237067083+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/application-data/influenceData/subs-to-notify |
2023-10-12T21:16:06.238172271+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2023-10-12T21:16:06.239241395+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=BSF |
2023-10-12T21:16:06.240057530+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2023-10-12T21:16:06.241019402+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Has no pre-config route
2023-10-12T21:16:06.241352863+09:00 [WARN][SMF][PduSess] Create URR
2023-10-12T21:16:06.241629780+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2023-10-12T21:16:06.241935085+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2023-10-12T21:16:06.242694268+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2023-10-12T21:16:06.243485670+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:41703] Handle UplinkNASTransport
2023-10-12T21:16:06.243878654+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:41703] Handle UplinkNASTransport (RAN UE NGAP ID: 0)
2023-10-12T21:16:06.244157862+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2023-10-12T21:16:06.244356193+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:0,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Configuration Update Complete
2023-10-12T21:16:06.245468064+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2023-10-12T21:16:06.247674465+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2023-10-12T21:16:06.247948022+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:41703] Send PDU Session Resource Setup Request
2023-10-12T21:16:06.248865286+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2023-10-12T21:16:06.318483431+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.121:41703] Handle PDUSessionResourceSetupResponse
2023-10-12T21:16:06.318771601+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:0,AU:1(3GPP)][ran_addr:192.168.0.121:41703] Handle PDUSessionResourceSetupResponse (RAN UE NGAP ID: 0)
2023-10-12T21:16:06.319766148+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2023-10-12T21:16:06.321966084+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2023-10-12T21:16:06.322260936+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:93c17951-59c7-46f8-8ca0-edb8b64d1ec8/modify |
```
The free5GC U-Plane log when executed is as follows.
```
2023-10-12T21:16:06.268065550+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionEstablishmentRequest
2023-10-12T21:16:06.268125636+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805][CPNodeID:192.168.0.141][CPSEID:0x1][UPSEID:0x1] New session
2023-10-12T21:16:06.268985572+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:1 period:10000000000}]
2023-10-12T21:16:06.269619573+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:2 period:10000000000}]
2023-10-12T21:16:06.345906243+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.142:8805] handleSessionModificationRequest
```
The result of `ip addr show` on VM4 (UE) is as follows.
```
# ip addr show
...
5: tun_srsue: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
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
PING google.com (142.251.42.142) from 10.60.0.1 tun_srsue: 56(84) bytes of data.
64 bytes from 142.251.42.142: icmp_seq=1 ttl=61 time=76.2 ms
64 bytes from 142.251.42.142: icmp_seq=2 ttl=61 time=78.0 ms
64 bytes from 142.251.42.142: icmp_seq=3 ttl=61 time=67.1 ms
```
- Run `tcpdump` on VM2 (U-Plane)
```
# tcpdump -i upfgtp -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on upfgtp, link-type RAW (Raw IP), snapshot length 262144 bytes
21:19:40.112265 IP 10.60.0.1 > 142.251.42.142: ICMP echo request, id 15, seq 1, length 64
21:19:40.155098 IP 142.251.42.142 > 10.60.0.1: ICMP echo reply, id 15, seq 1, length 64
21:19:41.111260 IP 10.60.0.1 > 142.251.42.142: ICMP echo request, id 15, seq 2, length 64
21:19:41.157498 IP 142.251.42.142 > 10.60.0.1: ICMP echo reply, id 15, seq 2, length 64
21:19:42.114221 IP 10.60.0.1 > 142.251.42.142: ICMP echo request, id 15, seq 3, length 64
21:19:42.146251 IP 142.251.42.142 > 10.60.0.1: ICMP echo reply, id 15, seq 3, length 64
```
In addition to `ping`, you may try to access the web by specifying the TUNnel interface with `curl` as follows.
- Run `curl google.com` on VM4 (UE)
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
21:20:36.810668 IP 10.60.0.1.60838 > 142.251.42.142.80: Flags [S], seq 1546501837, win 64240, options [mss 1460,sackOK,TS val 2864226356 ecr 0,nop,wscale 7], length 0
21:20:36.827197 IP 142.251.42.142.80 > 10.60.0.1.60838: Flags [S.], seq 23104001, ack 1546501838, win 65535, options [mss 1460], length 0
21:20:36.881833 IP 10.60.0.1.60838 > 142.251.42.142.80: Flags [.], ack 1, win 64240, length 0
21:20:36.882096 IP 10.60.0.1.60838 > 142.251.42.142.80: Flags [P.], seq 1:75, ack 1, win 64240, length 74: HTTP: GET / HTTP/1.1
21:20:36.882214 IP 142.251.42.142.80 > 10.60.0.1.60838: Flags [.], ack 75, win 65535, length 0
21:20:36.947922 IP 142.251.42.142.80 > 10.60.0.1.60838: Flags [P.], seq 1:774, ack 75, win 65535, length 773: HTTP: HTTP/1.1 301 Moved Permanently
21:20:36.985330 IP 10.60.0.1.60838 > 142.251.42.142.80: Flags [.], ack 774, win 63467, length 0
21:20:36.985330 IP 10.60.0.1.60838 > 142.251.42.142.80: Flags [F.], seq 75, ack 774, win 63467, length 0
21:20:36.985472 IP 142.251.42.142.80 > 10.60.0.1.60838: Flags [.], ack 76, win 65535, length 0
21:20:37.025268 IP 142.251.42.142.80 > 10.60.0.1.60838: Flags [F.], seq 774, ack 76, win 65535, length 0
21:20:37.088314 IP 10.60.0.1.60838 > 142.251.42.142.80: Flags [.], ack 775, win 63467, length 0
```
You could now create the end-to-end TUN interface on the DN and send any packets on the network.

---
In investigating 5G SA, I have built a simulation environment and can now use a very useful system for investigating 5GC and MEC of 5G SA mobile network. I would like to thank the excellent developers and all the contributors of free5GC, srsRAN Project and srsRAN 4G.

<a id="changelog"></a>

## Changelog (summary)

- [2023.10.12] Updated free5GC v3.3.0 (2023.10.11).
- [2023.10.11] Updated srsRAN_Project (2023.09.20).
- [2023.08.26] Initial release.

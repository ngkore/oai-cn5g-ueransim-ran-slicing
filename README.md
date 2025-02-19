# OAI-CN5G UERANSIM RAN Slicing

This repository provides deployment guides and configuration files for setting up OAI CN5G and UERANSIM for RAN slicing.

![image](https://github.com/user-attachments/assets/bccdc8b6-a722-49c4-8059-1e52e480f739)

## Deployment Guides

Follow these links for detailed deployment steps:

- **OAI CN5G Deployment Guide:** [OAI 5G Core Deployment](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_CN5G.md?ref_type=heads)
- **UERANSIM Deployment Guide:** [UERANSIM Installation](https://github.com/aligungr/UERANSIM/wiki/Installation)

Or follow the steps below.

## Hardware Requirements

- **OS:** Ubuntu 22.04 LTS
- **CPU:** 8 cores x86_64 @ 3.5 GHz
- **RAM:** 32 GB
- **Interfaces:** 2

## OAI CN5G Deployment

### 1. Install Pre-requisites

```bash
sudo apt install -y git net-tools putty

# Install Docker
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to Docker group
sudo usermod -a -G docker $(whoami)
reboot
```

### 2. Download Configuration Files

```bash
wget -O ~/oai-cn5g.zip https://gitlab.eurecom.fr/oai/openairinterface5g/-/archive/develop/openairinterface5g-develop.zip?path=doc/tutorial_resources/oai-cn5g
unzip ~/oai-cn5g.zip
mv ~/openairinterface5g-develop-doc-tutorial_resources-oai-cn5g/doc/tutorial_resources/oai-cn5g ~/oai-cn5g
rm -r ~/openairinterface5g-develop-doc-tutorial_resources-oai-cn5g ~/oai-cn5g.zip
```

#### Changes to `config.yaml` File

[modified config.yaml](./oai-cn5g-config/modified_config.yaml)

```patch
@@ -116,6 +116,10 @@
 snssais:
   - &embb_slice
     sst: 1
+  - &urllc_slice
+    sst: 2
+  - &mmtc_slice
+    sst: 3
 
 ############## NF-specific configuration
 amf:
@@ -142,6 +146,8 @@
       tac: 0x0001
       nssai:
         - *embb_slice
+        - *urllc_slice
+        - *mmtc_slice
   supported_integrity_algorithms:
     - "NIA1"
     - "NIA2"
@@ -175,8 +181,14 @@
       - sNssai: *embb_slice
         dnnSmfInfoList:
           - dnn: "oai"
-          - dnn: "openairinterface"
-          - dnn: "ims"
+          # - dnn: "openairinterface"
+          # - dnn: "ims"
+      - sNssai: *urllc_slice
+        dnnSmfInfoList:
+          - dnn: "oai-a"
+      - sNssai: *mmtc_slice
+        dnnSmfInfoList:
+          - dnn: "oai-b"
   local_subscription_infos:
     - single_nssai: *embb_slice
       dnn: "oai"
@@ -184,18 +196,30 @@
         5qi: 9
         session_ambr_ul: "10Gbps"
         session_ambr_dl: "10Gbps"
-    - single_nssai: *embb_slice
-      dnn: "openairinterface"
+    - single_nssai: *urllc_slice
+      dnn: "oai-a"
       qos_profile:
         5qi: 9
         session_ambr_ul: "10Gbps"
         session_ambr_dl: "10Gbps"
-    - single_nssai: *embb_slice
-      dnn: "ims"
+    - single_nssai: *mmtc_slice
+      dnn: "oai-b"
       qos_profile:
         5qi: 9
         session_ambr_ul: "10Gbps"
         session_ambr_dl: "10Gbps"
+    # - single_nssai: *embb_slice
+    #   dnn: "openairinterface"
+    #   qos_profile:
+    #     5qi: 9
+    #     session_ambr_ul: "10Gbps"
+    #     session_ambr_dl: "10Gbps"
+    # - single_nssai: *embb_slice
+    #   dnn: "ims"
+    #   qos_profile:
+    #     5qi: 9
+    #     session_ambr_ul: "10Gbps"
+    #     session_ambr_dl: "10Gbps"
 
 upf:
   support_features:
@@ -209,19 +233,31 @@
       - sNssai: *embb_slice
         dnnUpfInfoList:
           - dnn: "oai"
-          - dnn: "openairinterface"
-          - dnn: "ims"
+          # - dnn: "openairinterface"
+          # - dnn: "ims"
+      - sNssai: *urllc_slice
+        dnnUpfInfoList:
+          - dnn: "oai-a"
+      - sNssai: *mmtc_slice
+        dnnUpfInfoList:
+          - dnn: "oai-b"
 
 ## DNN configuration
 dnns:
   - dnn: "oai"
     pdu_session_type: "IPV4"
     ipv4_subnet: "10.0.0.0/24"
-  - dnn: "openairinterface"
-    pdu_session_type: "IPV4V6"
-    ipv4_subnet: "10.0.1.0/24"
-    ipv6_prefix: "2001:1:2::/64"
-  - dnn: "ims"
-    pdu_session_type: "IPV4V6"
-    ipv4_subnet: "10.0.9.0/24"
-    ipv6_prefix: "2001:1:2::/64"
+  # - dnn: "openairinterface"
+  #   pdu_session_type: "IPV4V6"
+  #   ipv4_subnet: "10.0.1.0/24"
+  #   ipv6_prefix: "2001:1:2::/64"
+  # - dnn: "ims"
+  #   pdu_session_type: "IPV4V6"
+  #   ipv4_subnet: "10.0.9.0/24"
+  #   ipv6_prefix: "2001:1:2::/64"
+  - dnn: "oai-a"
+    pdu_session_type: "IPV4"
+    ipv4_subnet: "10.0.0.0/24"
+  - dnn: "oai-b"
+    pdu_session_type: "IPV4"
+    ipv4_subnet: "10.0.0.0/24"
```

#### Changes to `oai_db.sql` File

[modified oai_db.sql](./oai-cn5g-config/modified_oai_db.sql)

```patch
@@ -213,11 +213,11 @@
 INSERT INTO `SessionManagementSubscriptionData` (`ueid`, `servingPlmnid`, `singleNssai`, `dnnConfigurations`) VALUES
 ('001010000000001', '00101', '{\"sst\": 1, \"sd\": \"FFFFFF\"}','{\"oai\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 6,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"},\"staticIpAddress\":[{\"ipv4Addr\": \"10.0.0.2\"}]},\"ims\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4V6\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 2,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"}}}');
 INSERT INTO `SessionManagementSubscriptionData` (`ueid`, `servingPlmnid`, `singleNssai`, `dnnConfigurations`) VALUES
-('001010000000002', '00101', '{\"sst\": 1, \"sd\": \"FFFFFF\"}','{\"oai\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 6,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"},\"staticIpAddress\":[{\"ipv4Addr\": \"10.0.0.3\"}]},\"ims\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4V6\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 2,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"}}}');
+('001010000000002', '00101', '{\"sst\": 2, \"sd\": \"FFFFFF\"}','{\"oai-a\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 6,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"},\"staticIpAddress\":[{\"ipv4Addr\": \"10.0.0.3\"}]},\"ims\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4V6\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 2,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"}}}');
 INSERT INTO `SessionManagementSubscriptionData` (`ueid`, `servingPlmnid`, `singleNssai`, `dnnConfigurations`) VALUES
-('001010000000003', '00101', '{\"sst\": 1, \"sd\": \"FFFFFF\"}','{\"oai\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 6,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"},\"staticIpAddress\":[{\"ipv4Addr\": \"10.0.0.4\"}]},\"ims\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4V6\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 2,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"}}}');
-INSERT INTO `SessionManagementSubscriptionData` (`ueid`, `servingPlmnid`, `singleNssai`, `dnnConfigurations`) VALUES
-('001010000000004', '00101', '{\"sst\": 1, \"sd\": \"FFFFFF\"}','{\"oai\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 6,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"},\"staticIpAddress\":[{\"ipv4Addr\": \"10.0.0.5\"}]},\"ims\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4V6\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 2,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"}}}');
+('001010000000003', '00101', '{\"sst\": 3, \"sd\": \"FFFFFF\"}','{\"oai-b\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 6,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"},\"staticIpAddress\":[{\"ipv4Addr\": \"10.0.0.4\"}]},\"ims\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4V6\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 2,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"}}}');
+-- INSERT INTO `SessionManagementSubscriptionData` (`ueid`, `servingPlmnid`, `singleNssai`, `dnnConfigurations`) VALUES
+-- ('001010000000004', '00101', '{\"sst\": 1, \"sd\": \"FFFFFF\"}','{\"oai\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 6,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"},\"staticIpAddress\":[{\"ipv4Addr\": \"10.0.0.5\"}]},\"ims\":{\"pduSessionTypes\":{ \"defaultSessionType\": \"IPV4V6\"},\"sscModes\": {\"defaultSscMode\": \"SSC_MODE_1\"},\"5gQosProfile\": {\"5qi\": 2,\"arp\":{\"priorityLevel\": 15,\"preemptCap\": \"NOT_PREEMPT\",\"preemptVuln\":\"PREEMPTABLE\"},\"priorityLevel\":1},\"sessionAmbr\":{\"uplink\":\"1000Mbps\", \"downlink\":\"1000Mbps\"}}}');
```

### 3. Pull and Run OAI CN5G

```bash
cd ~/oai-cn5g
docker compose pull
docker compose up -d
```

To stop OAI CN5G:

```bash
cd ~/oai-cn5g
docker compose down
```

## UERANSIM Deployment

### 1. Clone Repository

```bash
cd ~
git clone https://github.com/aligungr/UERANSIM
```

### 2. Install Dependencies

```bash
sudo apt update
sudo apt upgrade
sudo apt install -y make gcc g++ libsctp-dev lksctp-tools iproute2
sudo snap install cmake --classic
```

### 3. Build UERANSIM

```bash
cd ~/UERANSIM
make
```

## Configuration Files for gNB and UE

Add these config files at `~/UERANSIM/config/`

- [gNB Configuration Files](./gnb-config/)
- [UE Configuration Files](./ue-config/)

## Terminals

### Terminal 1: Watch AMF Logs

```bash
docker logs oai-amf -f
```

### Terminal 2: Run gNB for SST-1

Change the IP of `gnb-1` to interface 1.

```bash
cd ~/UERANSIM/config/
sudo ../build/nr-gnb -c oai-gnb-sst-1.yaml
```

### Terminal 3: Run gNB for SST-2

Change the IP of `gnb-2` to interface 2.

```bash
cd ~/UERANSIM/config/
sudo ../build/nr-gnb -c oai-gnb-sst-2.yaml
```

### Terminal 4: Run UE for SST-1

Update the gNB IP list.

```bash
cd ~/UERANSIM/config/
sudo ../build/nr-ue -c oai-ue-sst-1.yaml
```

### Terminal 5: Run UE for SST-2

Update the gNB IP list.

```bash
cd ~/UERANSIM/config/
sudo ../build/nr-ue -c oai-ue-sst-2.yaml
```

### Terminal 6: Run UE for SST-3

Update the gNB IP list.

```bash
cd ~/UERANSIM/config/
sudo ../build/nr-ue -c oai-ue-sst-3.yaml
```

> [!NOTE]
> Remove the gNB IP (e.g., `gnb1`) from the UE config to check the connection with another gNB (e.g., `gnb2`).

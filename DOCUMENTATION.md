# SigPloit Documentation

## 1. Project Overview
SigPloit is a signaling security testing framework dedicated to Telecom Security professionals and researchers. It is designed to pentest and exploit vulnerabilities in signaling protocols used by mobile operators across various generations (2G, 3G, 4G, and IMS).

SigPloit aims to cover:
- **SS7**: 2G/3G Voice and SMS attacks.
- **GTP**: 3G/4G Data roaming attacks on IPX/GRX interconnects.
- **Diameter**: 4G LTE roaming interconnects (Planned for Version 3).
- **SIP**: VoLTE and IMS infrastructure (Planned for Version 4).

## 2. Installation and Requirements
### Requirements
- **Operating System**: Linux (Ubuntu recommended).
- **Python**: Version 2.7.
- **Java**: Version 1.7 or higher.
- **Tools**: `lksctp-tools` (Install via `sudo apt-get install lksctp-tools`).

### Installation
1. Clone the repository.
2. Navigate to the SigPloit directory: `cd SigPloit`
3. Install Python dependencies: `sudo pip2 install -r requirements.txt`

### Running SigPloit
Execute the main script with root privileges if necessary:
```bash
python sigploit.py
```

## 3. Main Menu and Module Selection
Upon launching `sigploit.py`, you are presented with the main menu:
- **0) SS7**: Accesses the SS7 vulnerability suite.
- **1) GTP**: Accesses the GTP vulnerability suite.
- **2) Diameter**: (Coming Soon) 4G Data attacks.
- **3) SIP**: (Coming Soon) 4G IMS attacks.

## 4. SS7 Module
The SS7 module is organized into four attack categories. These attacks typically involve calling Java-based JAR files that handle the TCAP/MAP signaling.

### 4.1 Location Tracking
Used to identify the geographic location of a subscriber.
- **SendRoutingInfo (SRI)**: Used to route calls; can be used to retrieve the subscriber's HLR and MSC address.
- **ProvideSubscriberInfo (PSI)**: Provides reliable location tracking by requesting cell ID and location information.
- **SendRoutingInfoForSM (SRI-SM)**: Retrieves the MSC/VLR address for SMS delivery.
- **AnyTimeInterrogation (ATI)**: Requests subscriber information including location and state from the HLR.
- **SendRoutingInfoForGPRS (SRI-GPRS)**: Retrieves the SGSN address for a subscriber.

### 4.2 Call and SMS Interception
- **UpdateLocation**: A stealthy method for SMS interception by tricking the HLR into updating the subscriber's location to an attacker-controlled MSC.

### 4.3 Fraud & Info Gathering
- **SendIMSI**: Retrieves the IMSI of a subscriber.
- **MTForwardSMS**: Used for SMS phishing and spoofing.
- **InsertSubscriberData (ISD)**: Manipulates the subscriber profile in the VLR.
- **SendAuthenticationInfo (SAI)**: Retrieves authentication vectors for a subscriber.

### 4.4 Denial of Service (DoS)
- **PurgeMS**: Performs a mass DoS attack by telling the HLR that the subscriber is no longer reachable, taking them off the network.

## 5. GTP Module
The GTP module focuses on attacks occurring on the GTPv2 protocol used in 4G data roaming.

### 5.1 Attack Categories
- **Information Gathering**:
    - **GTP Nodes Discovery**: Discovers Network Elements (NE) using EchoRequest, CreateSession, DeleteSession, or DeleteBearer messages.
    - **TEID Allocation Discovery**: Discovers TEID allocation patterns.
- **Fraud**:
    - **Tunnel Hijack**: Attempts to hijack a user's data tunnel using ModifyBearerRequest messages.
- **Denial of Service (Planned/Partially Implemented)**:
    - **Massive DoS**: Sending Delete PDN Connection Set Request.
    - **User DoS**: Sending Delete Session/Bearer requests for specific TEIDs.

### 5.2 Configuration Options
GTP attacks use configuration files (`.cnf`) located in `gtp/config/`. Key parameters include:
- **target**: The IP address or network range of the target node (e.g., 10.10.10.1/32).
- **config**: Path to the `.cnf` file used for the attack.
- **listening**: (True/False) Whether to accept and process replies from the target.
- **verbosity**: Level of output detail.
- **output**: Path to save results (e.g., `results.csv`).

### 5.3 Configuration File (.cnf) Parameters
Sections in `.cnf` files:
- **[GENERIC]**:
    - `interface`: The GTP interface type (e.g., 4: S5/S8 SGW GTP-U, 7: S5/S8 PGW GTP-C).
    - `source_ip`: Attacker's IP address.
    - `teid`: Tunnel Endpoint Identifier.
    - `sqn`: Sequence Number.
- **[IES]**: Information Elements required for the specific message (IMSI, MCC, MNC, APN, etc.).

## 6. Testing Environment
The `Testing/` directory contains server-side simulations for various attacks. These are useful for testing SigPloit in a controlled environment without access to a real telecom network.

- **Location**: `Testing/Server/Attacks/`
- **Contents**: Java-based servers (e.g., `SRILowLevelServer.java`) and their JAR files.
- **Usage**: Each server directory usually contains a `README_Instructions` file detailing the hardcoded values (IPs, Ports, MSISDN) and how to execute the simulator.

## 7. File Structure
- `sigploit.py`: The main entry point of the framework.
- `ss7main.py`: Main menu and logic for SS7 attacks.
- `gtpmain.py`: Main menu and logic for GTP attacks.
- `ss7/`: Python wrappers and attack implementations for SS7.
    - `attacks/`: Contains the JAR files and configuration for SS7 messages.
- `gtp/`: Implementation of the GTPv2 protocol and attacks.
    - `attacks/`: Python scripts for specific GTP attacks.
    - `config/`: Configuration files (`.cnf`) for GTP attacks.
    - `gtp_v2_core/`: Core protocol handling for GTPv2.
- `Testing/`: Simulated server environments for testing.
- `requirements.txt`: Python dependencies.

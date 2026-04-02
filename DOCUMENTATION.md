# SigPloit Documentation

SigPloit is a signaling security testing framework dedicated to Telecom Security professionals and researchers. It allows for pentesting and exploiting vulnerabilities in signaling protocols used by mobile operators (SS7, GTP, Diameter, SIP).

## Table of Contents
1. [Architecture & Project Structure](#architecture--project-structure)
2. [SS7 Module](#ss7-module)
    - [Location Tracking](#location-tracking)
    - [Interception](#interception)
    - [Fraud & Info](#fraud--info)
    - [Denial of Service (DoS)](#denial-of-service-dos)
3. [GTP Module](#gtp-module)
    - [Information Gathering](#information-gathering)
    - [Fraud](#fraud)
    - [Configuration Options](#configuration-options)
    - [Configuration Files](#configuration-files)
4. [Installation & Requirements](#installation--requirements)
5. [Usage Guide](#usage-guide)

---

## Architecture & Project Structure

The project is organized into several main modules, each targeting a different signaling protocol.

- **`sigploit.py`**: The main entry point of the framework. It provides the top-level menu to select between SS7, GTP, and future modules.
- **`ss7main.py`**: The main controller for the SS7 module. It manages the attack categories and menus for SS7.
- **`gtpmain.py`**: The main controller for the GTP module. It manages the attack categories and menus for GTP.
- **`ss7/`**: Contains the logic and individual attack scripts for SS7.
    - `ss7/attacks/`: Contains subdirectories for each attack category, housing the compiled Java `.jar` files used for execution.
- **`gtp/`**: Contains the logic and individual attack scripts for GTP.
    - `gtp/attacks/`: Contains the implementation of GTP attacks in Python.
    - `gtp/config/`: Contains configuration files (`.cnf`) for GTP attacks.
    - `gtp/gtp_v2_core/`: Contains the core GTPv2 protocol implementation and utilities.
- **`Testing/`**: Contains server-side components for testing attacks in a lab environment.

---

## SS7 Module

The SS7 module is designed for testing 2G/3G Voice and SMS attacks. Attacks in this module are generally implemented as Java applications packaged in `.jar` files, which are called by the Python framework.

### Location Tracking
Used to identify the geographic location of a subscriber.

| Message/Attack | Description | JAR Location |
| --- | --- | --- |
| **SendRoutingInfo (SRI)** | Location Tracking; used to route calls. Often blocked by operators. | `ss7/attacks/tracking/sri/SendRoutingInfo.jar` |
| **ProvideSubscriberInfo (PSI)** | Reliable Location Tracking. | `ss7/attacks/tracking/psi/ProvideSubscriberInfo.jar` |
| **SendRoutingInfoForSM (SRI-SM)** | Reliable Location Tracking via SMS. May need to be run twice if home routing is used. | `ss7/attacks/tracking/srism/SendRoutingInfoforSM.jar` |
| **AnyTimeInterrogation (ATI)** | Location Tracking; highly effective but blocked by most operators. | `ss7/attacks/tracking/ati/AnyTimeInterrogation.jar` |
| **SendRoutingInfoForGPRS (SRI-GPRS)** | Location tracking used for data routing; retrieves SGSN GT. | `ss7/attacks/tracking/srigprs/SendRoutingInfoForGPRS.jar` |

### Interception
Focuses on intercepting subscriber communications.

| Message/Attack | Description | JAR Location |
| --- | --- | --- |
| **UpdateLocation (UL)** | Stealthy SMS Interception by updating the subscriber's location to an attacker-controlled GT. | `ss7/attacks/interception/ul/UpdateLocation.jar` |

### Fraud & Info
Used for gathering subscriber information or performing fraudulent activities.

| Message/Attack | Description | JAR Location |
| --- | --- | --- |
| **SendIMSI** | Retrieving the IMSI of a subscriber. | `ss7/attacks/fraud/simsi/SendIMSI.jar` |
| **MTForwardSMS** | SMS Phishing and Spoofing. | `ss7/attacks/fraud/mtsms/MTForwardSMS.jar` |
| **InsertSubscriberData (ISD)** | Subscriber Profile Manipulation. | `ss7/attacks/fraud/isd/InsertSubscriberData.jar` |
| **SendAuthenticationInfo (SAI)** | Subscriber Authentication Vectors retrieval. | `ss7/attacks/fraud/sai/SendAuthenticationInfo.jar` |

### Denial of Service (DoS)
Aims to disrupt service for subscribers or network nodes.

| Message/Attack | Description | JAR Location |
| --- | --- | --- |
| **PurgeMS** | Mass DoS attack on subscribers to take them off the network. | `ss7/attacks/dos/prgms/PurgeMS.jar` |

---

## GTP Module

The GTP module focuses on 3G/4G data attacks occurring on IPX/GRX interconnects. Unlike SS7, these are primarily implemented in Python.

### Information Gathering

| Attack | Description | Python Script |
| --- | --- | --- |
| **GTP Nodes Discovery** | Network Element Discovery using EchoRequest, CreateSession, DeleteSession, or DeleteBearer messages. | `gtp/attacks/info/discover_gtp_nodes.py` |
| **TEID Allocation Discovery** | TEID Discovery using CreateSession, ModifyBearer, or CreateBearer messages. | `gtp/attacks/info/discover_teid_allocation.py` |

### Fraud

| Attack | Description | Python Script |
| --- | --- | --- |
| **Tunnel Hijack** | TEID Hijack using ModifyBearerRequest messages. | `gtp/attacks/fraud/tunnel_hijacking.py` |

### Configuration Options
GTP attacks use a command-line interface within SigPloit to set parameters.

- **`config`**: Path to the configuration file (`.cnf`).
- **`target`**: The target network or IP (e.g., `10.10.10.1/32` or `10.10.10.0/24`).
- **`listening`**: Boolean (True/False) to indicate if the tool should wait for replies from the target.
- **`verbosity`**: Level of output detail (default: 2).
- **`output`**: File to save results (default: `results.csv`).

### Configuration Files
Located in `gtp/config/`, these files define the structure of the GTP messages sent during attacks.

- **`EchoRequest.cnf`**: Used for node discovery.
- **`CreateSession.cnf`**: Used for node and TEID discovery.
- **`DeleteSession.cnf`**: Used for node discovery.
- **`DeleteBearer.cnf`**: Used for node discovery.
- **`ModifyBearer.cnf`**: Used for TEID discovery and tunnel hijacking.
- **`TunnelHijack.cnf`**: Specific for tunnel hijacking attacks.

---

## Installation & Requirements

### Prerequisites
- **Operating System**: Linux (recommended: Ubuntu/Debian)
- **Python**: 2.7
- **Java**: 1.7 or higher
- **Dependencies**: `lksctp-tools` (run `sudo apt-get install lksctp-tools`)

### Setup
1. Clone the repository.
2. Install Python dependencies:
   ```bash
   pip install -r requirements.txt
   ```

---

## Usage Guide

1. **Start SigPloit**:
   ```bash
   python sigploit.py
   ```
2. **Main Menu**: Choose the signaling protocol (0 for SS7, 1 for GTP).
3. **Attack Categories**: Select an attack category (e.g., Location Tracking for SS7).
4. **Execute Attack**:
   - For **SS7**: Select the specific message/attack. The framework will launch the corresponding Java application. Follow the on-screen prompts.
   - For **GTP**:
     - Use `show options` to see current parameters.
     - Use `set <option> <value>` to configure (e.g., `set target 1.2.3.4`).
     - Use `run` to execute the attack.
5. **Navigation**: Use `back` to return to previous menus or `exit`/`quit` to leave the framework.

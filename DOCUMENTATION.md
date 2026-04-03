# SigPloit Detailed Documentation

SigPloit is a professional signaling security testing framework for mobile operator interconnects. It supports SS7, GTP, and provides placeholders for Diameter and SIP.

## Table of Contents
1. [Architecture & Project Structure](#architecture--project-structure)
2. [SS7 Module Deep Dive](#ss7-module)
    - [Attack Categories & Messages](#ss7-attacks)
    - [Parameter Dictionary & Origins](#ss7-parameters)
    - [Attack Surface & Vectors](#ss7-attack-surface)
3. [GTP Module Deep Dive](#gtp-module)
    - [Attack Categories](#gtp-attacks)
    - [Configuration Options](#gtp-options)
    - [IE Parameter Dictionary & Origins](#gtp-parameters)
    - [Attack Surface & Vectors](#gtp-attack-surface)
4. [Diameter & SIP Modules](#other-modules)
5. [Virtual Lab & Testing (Testing/Server)](#virtual-lab)
6. [Installation & Usage](#installation)

---

## Architecture & Project Structure

SigPloit is built with a modular approach, separating protocol logic from the main UI.

- **`sigploit.py`**: The CLI entry point. It manages the global state and main navigation.
- **`ss7main.py` / `gtpmain.py`**: Protocol-specific controllers. They handle menu navigation and parameter preparation for their respective modules.
- **`ss7/`**: Implements SS7/MAP logic.
    - **Wrappers**: `tracking.py`, `interception.py`, `fraud.py`, `dos.py` wrap calls to Java-based attack engines.
    - **Engines**: Located in `ss7/attacks/`, these are compiled Java `.jar` files that handle the low-level M3UA/SCCP/MAP stack.
- **`gtp/`**: Implements GTPv2 logic natively in Python.
    - **`gtp_v2_core/`**: A custom implementation of the GTPv2-C and GTPv2-U protocols.
    - **`config/`**: Contains `.cnf` files which are essentially "Attack Templates" defining message structures.
- **`Testing/Server/`**: A collection of Java-based simulators that act as HLRs, VLRs, or MSCs to respond to SigPloit attacks in a controlled environment.

---

## SS7 Module Deep Dive <a name="ss7-module"></a>

### Attack Categories & Messages <a name="ss7-attacks"></a>

| Category | Message | Purpose | Implementation Path |
| --- | --- | --- | --- |
| **Location** | `SendRoutingInfo` (SRI) | Retrieve the serving MSC/VLR address. | `ss7/attacks/tracking/sri/` |
| **Location** | `ProvideSubscriberInfo` (PSI) | Request detailed location (Cell ID) from the serving VLR. | `ss7/attacks/tracking/psi/` |
| **Location** | `AnyTimeInterrogation` (ATI) | Direct query to HLR for subscriber state and location. | `ss7/attacks/tracking/ati/` |
| **Interception** | `UpdateLocation` (UL) | Register the attacker's node as the new VLR for the target. | `ss7/attacks/interception/ul/` |
| **Fraud** | `SendIMSI` | Map a phone number (MSISDN) to its internal ID (IMSI). | `ss7/attacks/fraud/simsi/` |
| **Fraud** | `MTForwardSMS` | Inject a spoofed SMS directly to the target's MSC. | `ss7/attacks/fraud/mtsms/` |
| **DoS** | `PurgeMS` | Inform the HLR that the subscriber has been "purged" from the VLR. | `ss7/attacks/dos/prgms/` |

### Parameter Dictionary & Origins <a name="ss7-parameters"></a>

| Parameter | Technical Meaning | Source / How to Obtain |
| --- | --- | --- |
| **MSISDN** | Mobile Station International Subscriber Directory Number. | Public knowledge; the target's phone number. |
| **IMSI** | International Mobile Subscriber Identity. | Obtained via `SendIMSI` or `SRI` responses. Private ID. |
| **GT (Global Title)** | The "IP Address" of the SS7 node (e.g., HLR GT, VLR GT). | Obtained from `SRI` or `SRI-SM` responses; indicates the serving node. |
| **PC (Point Code)** | Physical node address in the SS7 network. | Obtained via network discovery or provided by the signaling provider. |
| **SSN (Subsystem)** | Specific service on a node (HLR=6, VLR=7, MSC=8, gsmSCF=147). | Standardized values defined by ITU-T/3GPP. |
| **RI (Routing Indicator)** | Defines if routing is based on GT or PC + SSN. | Configuration choice based on network topology. |

### Attack Surface & Vectors <a name="ss7-attack-surface"></a>

The **SS7 Attack Surface** is the global signaling interconnect network (SIGTRAN/M3UA). Access is typically gained via:
1. **Low-cost Signaling Providers**: Companies that sell access to the SS7 network for legitimate services (A2P SMS) but often have weak vetting.
2. **Compromised Network Nodes**: Gaining access to a small operator's STP (Signaling Transfer Point) or HLR/VLR.

#### Detailed Attack Vector: SMS Interception (The "UpdateLocation" Attack)
This is one of the most critical attack vectors in SigPloit.
1. **Discovery**: The attacker uses `SendIMSI` with the target's **MSISDN** to retrieve the **IMSI** and the **HLR GT**.
2. **Spoofing**: The attacker sends a `MAP_UPDATE_LOCATION` message to the target's **HLR**.
   - The message contains the target's **IMSI**.
   - The **Originating GT** is set to the attacker's own Global Title (pretending to be a new VLR the target just roamed into).
3. **Execution**: The **HLR**, believing the subscriber has moved, sends a `MAP_CANCEL_LOCATION` to the real VLR (disconnecting the user) and sends the `MAP_INSERT_SUBSCRIBER_DATA` (subscriber profile) to the attacker.
4. **Intercept**: The **HLR** now thinks the attacker's node is the serving VLR. Any incoming SMS or call for the target is routed to the attacker.

---

## GTP Module Deep Dive <a name="gtp-module"></a>

### Attack Categories <a name="gtp-attacks"></a>

- **Information Gathering**: Discovering GTP-capable nodes (SGW, PGW, MME) and active Tunnel Endpoint Identifiers (TEIDs).
- **Fraud (Tunnel Hijacking)**: Redirecting user data traffic by updating the session information in the core network.
- **DoS**: Dropping sessions or connections by spoofing management messages.

### Configuration Options <a name="gtp-options"></a>

GTP attacks use a CLI shell with the following commands:
- `set target <IP/CIDR>`: The IP address or range of the target GSN/SGW/PGW.
- `set config <path>`: Points to a `.cnf` file in `gtp/config/` (the "payload").
- `set listening <True/False>`: Whether to wait for and log response packets.
- `set verbosity <1-3>`: Level of detail in the console output.

### IE Parameter Dictionary & Origins <a name="gtp-parameters"></a>

Configured within `.cnf` files in the `[IES]` section:

| Parameter | Description | Source / Origin |
| --- | --- | --- |
| **TEID** | Tunnel Endpoint Identifier. | Discovered via `TEID Allocation Discovery` or `GTP Nodes Discovery`. |
| **MCC / MNC** | Mobile Country Code / Mobile Network Code. | Publicly available codes for every operator (e.g., 222/88 for TIM Italy). |
| **APN** | Access Point Name. | Often standardized (e.g., `wap.tim.it`, `internet`, `ims`). |
| **RAT Type** | Radio Access Technology (1:UTRAN, 2:GERAN, 6:E-UTRAN). | Depends on the target generation (3G vs 4G). |
| **F-TEID** | Fully Qualified TEID (IP + TEID). | The attacker's IP and a chosen TEID for redirection. |
| **EBI** | EPS Bearer ID. | Typically 5 for the default bearer; can be discovered. |
| **LAC / RAC** | Location Area Code / Routing Area Code. | Discovered via signaling or network proximity. |
| **NIT** | Node ID Type (0:IPv4, 2:FQDN). | Technical choice based on target configuration. |
| **Cause** | Error code for `Delete Bearer` (e.g., 3: Reactivation Requested). | Used to trigger specific behaviors in the target node. |

### GTP Attack Surface & Vectors <a name="gtp-attack-surface"></a>

The **GTP Attack Surface** is the GPRS Roaming eXchange (GRX) or IP eXchange (IPX). These are private IP networks that connect mobile operators.

#### Detailed Attack Vector: Tunnel Hijacking
1. **Reconnaissance**: Attacker uses `GTP Nodes Discovery` to find the IP of a PGW and `TEID Allocation Discovery` to find active sessions.
2. **Exploitation**: Attacker sends a `Modify Bearer Request` to the **SGW**.
   - The message targets a specific **TEID**.
   - It specifies a new **F-TEID** (S-GW address for the User Plane) which points to the attacker's IP.
3. **Redirection**: The **PGW** updates the tunnel information. All data packets originating from the internet for that subscriber are now encapsulated and sent to the attacker's IP instead of the real SGW.

---

## Diameter & SIP Modules <a name="other-modules"></a>

As of Version 1.1 (BETA), these modules are placeholders for future releases:
- **Diameter**: Targeting 4G LTE Roaming interconnects. Currently displays a "will be updated in version 3" message.
- **SIP**: Targeting VoLTE and IMS infrastructure. Currently displays a "will be updated in version 4" message.

---

## Virtual Lab & Testing (Testing/Server) <a name="virtual-lab"></a>

SigPloit includes a comprehensive set of "Vulnerable Servers" for safe testing.

### Available Simulators
- **Location Tracking Servers**: `AnyTimeInterrogation_Server`, `ProvideSubscriberInfo_Server`, etc.
- **Fraud Servers**: `SendIMSI_Server`, `MTForwardSMS_Server`.
- **Interception Servers**: `UpdateLocation_Server`.

### Lab Workflow
1. **Launch Server**: Go to `Testing/Server/Attacks/...` and run `java -jar <Server>.jar`.
2. **Identify Hardcoded Parameters**: Read the `README_Instructions` in the server folder.
   - *Example*: `AnyTimeInterrogation_Server` expects the attacker at `192.168.56.101:2905` and responds to MSISDN `96599657765`.
3. **Configure SigPloit**: In `sigploit.py`, use the parameters specified in the server's README to ensure the message is accepted and processed by the simulator.

---

## Installation & Usage <a name="installation"></a>

### Prerequisites
- **Linux** (Tested on Ubuntu/Debian).
- **Python 2.7**.
- **Java 1.7+**.
- **lksctp-tools**: Required for SCTP/SIGTRAN communication. Install via `sudo apt-get install lksctp-tools`.

### Running SigPloit
```bash
# Install Python dependencies
sudo pip install -r requirements.txt

# Start the framework
python sigploit.py
```
1. Select **Protocol** (0 for SS7, 1 for GTP).
2. Select **Attack Category**.
3. Fill in the **Parameters** (Origins described in the dictionaries above).
4. View the output in the console or the designated `results.csv` (for GTP).

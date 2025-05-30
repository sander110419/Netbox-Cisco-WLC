# Cisco WLC to NetBox Synchronization Tool

This script synchronizes Cisco Wireless LAN Controller (WLC) access points to NetBox, providing an automated way to maintain your NetBox DCIM database with accurate information from your WLC environment.

## Overview

The Cisco_WLC_to_Netbox.py script discovers and synchronizes the following WLC resources to NetBox:
- Access Points → NetBox Devices
- AP IP Addresses → NetBox IP Addresses
- AP MAC Addresses → NetBox Interface MAC Addresses
- Site information derived from AP naming convention → NetBox Sites

## Prerequisites

- Python 3.6+
- SSH access to a Cisco WLC
- A running NetBox instance with API access
- Required Python packages (see Installation)

## Installation

1. Clone this repository:
```bash
git clone https://github.com/yourusername/Cisco_WLC_to_Netbox.git
```

2. Install the required dependencies:
```bash
pip install paramiko pynetbox tqdm urllib3
```

## Configuration

The script can be configured using command-line arguments or environment variables:

### Environment Variables
- `WLC_HOST`: Hostname or IP address of your Cisco WLC
- `WLC_USERNAME`: WLC SSH username
- `WLC_PASSWORD`: WLC SSH password
- `NETBOX_URL`: URL of your NetBox instance
- `NETBOX_TOKEN`: NetBox API token with write access

## Usage

### Basic Usage
```bash
python Cisco_WLC_to_Netbox.py --wlc-host 10.65.32.2 --wlc-username admin --wlc-password your-password --netbox-url https://your-netbox-instance/ --netbox-token your-netbox-token
```

### Using Environment Variables
```bash
export WLC_HOST=10.65.32.2
export WLC_USERNAME=admin
export WLC_PASSWORD=your-password
export NETBOX_URL=https://your-netbox-instance/
export NETBOX_TOKEN=your-netbox-token
python Cisco_WLC_to_Netbox.py
```

## Features

- **Automatic Discovery**: Automatically discovers all access points configured on the WLC
- **SSH Automation**: Uses paramiko to handle SSH connections and command execution
- **Resource Mapping**: Maps WLC access points to appropriate NetBox objects
- **Idempotent Operation**: Can be run multiple times safely, updating existing resources
- **Tagging**: Adds "cisco-wlc-sync" tag to all created/updated objects in NetBox
- **Site Detection**: Extracts facility IDs from AP names (e.g., "APN-008-01" → facility ID "8")
- **Name Handling**: Automatically handles truncation for device names
- **Management Interface**: Creates management interfaces for access points with their MAC addresses

## Data Synchronization Details

1. **Access Points**: Retrieved from WLC using the "show ap config general" command
2. **Facility ID Extraction**: Extracts facility IDs from AP names using pattern matching
   - Format: APN-NNN-XX → facility ID NNN (e.g., APN-008-01 → facility ID 8)
   - Special cases: APN-000-XXX, APN-LOODS-01, APN-INFRA-00 → facility ID 00
3. **Device Types**: Created based on detected Cisco AP models
4. **Management Interface**: Created for each AP with its Ethernet MAC address
5. **IP Addresses**: Associated with the management interface of each AP and set as primary IP

## Troubleshooting

- **SSH Connection Issues**: Ensure the WLC is reachable and that credentials are correct.
- **"--More--" Prompt Handling**: The script automatically handles pagination in WLC command output.
- **SSL Certificate Issues**: The script disables SSL verification for NetBox by default.
- **Logging**: The script logs operations at INFO level. Review logs for troubleshooting.
- **Timeout Issues**: Command execution has a 30-second timeout to prevent script hangs.

## Notes

- The script is designed to be run periodically to keep NetBox updated with the current state of WLC access points.
- All objects created or updated by the script receive a "cisco-wlc-sync" tag for identification.
- The script uses facility IDs extracted from AP names to match with sites in NetBox.
- The script handles name truncation to comply with NetBox's 64-character name limit.
- For AP names with dots (e.g., "AP.example.com"), only the portion before the first dot is used.
- The script gracefully handles pagination in WLC command output, automatically sending space when "--More--" prompts are encountered.

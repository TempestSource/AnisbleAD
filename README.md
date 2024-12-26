# AnisbleAD
Guide to setup Anisble for AD controlled machines using GPO and AD CA

Ansible uses winRM by default, and this guide configures it to use kerberos authentication and https 

# Part 1: Enabling WinRM with GPO
## Step 1: Create a GPO and link it to the desired group of machines

## Step 2: Configure GPO to Enable WinRM
Allowing remote management
- Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Windows Remote Management (WinRM) -> WinRM Service -> Allow remote server management through WinRM -> Enabled

Enabling the WinRM Service on Startup
- Computer Configuration -> Policies -> Windows Settings -> Security Settings -> System Services -> Windows Remote Management (WS-Management) -> Automatic

Allowing WinRM through the Firewall
- Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Windows Defender Firewall with Advanced Security -> Inbound Rules
- Create a new rule -> TCP Port: 5986 -> Allow -> Only leave domain checked

# Part 2: Configuring the Certificate

# Part 3: Configuring the Ansible Host

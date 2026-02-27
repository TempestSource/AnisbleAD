# AnsibleAD
Guide to setup Ansible for AD controlled machines using GPO and AD CA

Ansible uses WinRM by default, and this guide configures it to use Kerberos authentication and https 

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
## Step 1: Enable auto-enrollment
In the GPO from steps 1/2:
- Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Public Key Policies -> Certificate Service Client - Auto-Enrollment
- Set configuration model to enabled and check both boxes

## Step 2: Creating the Certificate
- Open the certificate authority panel and navigate to your CA
- Right click certificate templates -> Manage
- Right click Web Server -> Duplicate
- Set your certificate name in the general tab
- In subject name choose build from this AD information
- Change subject name format to common name
- Leave only the DNS name box checked
- In the security tab, add the group/user you want to manage
- Under permissions for this user/group, check read, enroll, and autoenroll
- Save the certificate template and close the CA console, but not the original panel
- Right click certificate template -> new -> certificate template to issue -> find your new certificate template

# Part 3: Configuring the Ansible Host
## Step 1: Install the Kerberos library
This step will depend on the OS/Distro of your Ansible host; follow the Ansible documentation for this

## Step 2: Configure Ansible hosts variables
- Navigate to your Ansible hosts file
- Make sure the Windows machines you are managing are in a group, I have mine named windesktop
- Configure as shown:
```ini
[windesktop:vars]
ansible_user=Administrator@DOMAIN.NAME
ansible_port=5986
ansible_connection=winrm
ansible_winrm_transport=kerberos
ansible_winrm_scheme=https
```

## Step 3: Authenticating with Kerberos
- To obtain a Kerberos ticket for authentication run the following on the Ansible host:
```bash
kinit Administrator@DOMAIN.NAME
```

## Step 4: Copying the certificate to the Ansible host
To connect with https, we will need to add our CA to our trust store
- On the Windows server with CA, open the CA panel
- Right click on your CA server name and open the properties menu
- Select Certificate #0 and click view certificate
- Open the details tab and select copy to file
- Select DER .CER and export the file, here I used caCert.cer
- Copy the file to your Ansible host, I used scp but you can use a shared folder or other method

## Step 5: Converting from DER to PEM
In order for OpenSSL to read the certificate, we need to convert it
```bash
openssl x509 -inform DER -outform PEM -in caCert.cer -out caCert.crt
```

## Step 6: Importing the key
```bash
sudo cp caCert.crt /usr/local/share/ca-certificates
sudo update-ca-certificates
```

# Part 4: Testing
At this point, you should be good to go. To test, from your Ansible host run the following:
```bash
ansible windesktop -m win_ping
```
You should receive a SUCCESS with a pong message, and the output should be highlighted green instead of red

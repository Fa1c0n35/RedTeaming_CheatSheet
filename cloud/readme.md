# Pentesting the cloud cheatsheet
* [General](#General)
  * [Scaning tools](#Scanning-tools)
* [Recon \ OSINT](/OSINT.md#Cloud)
* [Initial access attacks](initial-access-attacks.md)
* [Cloud Services](readme.md)
  * [Azure](azure/readme.md)
  * [Amazon Web Services](aws/readme.md)
  * [Google Cloud](gc/readme.md)

# General
- Google Cloud Platform != Google Workspace
  - Google Cloud Platform provides cloud services. Iaas, Paas, Saas
  - Google Workspace (G-suite) provides business applications. Saas, Idaas
- Azure != Microsoft 365 != Azure AD
  - Azure AD is the Identity Management part of Azure. This is where all the users are.
  - Azure provides cloud services. Iaas, Paas, Saas
  - Microsoft365 provides business applications. Saas, Idaas
- Google Workspace and Microsoft 365 are productivity suites
- Rules of engagement 
  - Azure https://www.microsoft.com/en-us/msrc/pentest-rules-of-engagement
  - AWS: https://aws.amazon.com/security/penetration-testing/
  - GCP  https://support.google.com/cloud/answer/6262505?hl=en
- Enumerate host https://github.com/dafthack/HostRecon

## Scanning tools
### Enumeration
- AWS
  - https://github.com/carnal0wnage/weirdAAL

### Vulnerability scanning
- Scoutsuite
  - https://github.com/nccgroup/ScoutSuite
  - Scans AWS, Azure, GCP, Alibaba cloud, Oracle cloud
- Scoutsploit
  - https://github.com/cloudsploit/scans
  - Scans AWS, Azure, GCP, Oracle
- Kics
  - https://github.com/Checkmarx/kics
  - Terraform, Ansible, Docker etc 

### Privesc scanning
- GCP
  - https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation/tree/master/PrivEscScanner
- AWS
  - https://github.com/RhinoSecurityLabs/pacu
- Azure
  - https://github.com/Azure/Stormspotter
- General
  - https://github.com/cyberark/SkyArk

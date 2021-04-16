# CitrixFAS

The Federated Authentication Service (FAS) is a Citrix component that integrates with your Active Directory certificate authority (CA), allowing users to be seamlessly authenticated within a Citrix environment. This document describes various authentication architectures that may be appropriate for your deployment.

When enabled, the FAS delegates user authentication decisions to trusted StoreFront servers. StoreFront has a comprehensive set of built-in authentication options built around modern web technologies and is easily extensible using the StoreFront SDK or third-party IIS plugins. The basic design goal is that any authentication technology that can authenticate a user to a web site can now be used to log in to a Citrix XenApp or XenDesktop deployment.

This document covers some examples of top-level deployment architectures, in increasing complexity.



How it works
The FAS is authorized to issue smart card class certificates automatically on behalf of Active Directory users who are authenticated by StoreFront. This uses similar APIs to tools that allow administrators to provision physical smart cards. When a user is brokered to a Citrix XenApp or XenDesktop Virtual Delivery Agent (VDA), the certificate is attached to the machine, and the Windows domain sees the logon as a standard smart card authentication. The Citrix Federated Authentication Service is a privileged component designed to integrate with Active Directory Certificate Services. It dynamically issues certificates for users, allowing them to log on to an Active Directory environment as if they had a smart card. This allows StoreFront to use a broader range of authentication options, such as SAML (Security Assertion Markup Language) assertions. SAML is commonly used as an alternative to traditional Windows user accounts on the Internet. The following diagram shows the Federated Authentication Service integrating with a Microsoft Certification Authority and providing support services to StoreFront and XenApp and XenDesktop Virtual Delivery Agents (VDAs).


![alt text](https://user-images.githubusercontent.com/82611568/114958398-87410d80-9e28-11eb-890d-5477923a96b5.png)

Requirements
AD Certificate Services
Delivery Controller (7.12+ Recommended)
StoreFront 3.6+
VDA 7.9 (7.12+ Recommended)
To start off with 

make sure XDC works with standard AD username and password first
add FAS before adding Federation
add Netscaler and make sure it works before LDAP
once everything looks good add your Federation as long as the Netscaler passes a UPN to Storefront and one through 3 works
The Federated Authentication Service is supported on Windows servers (Windows Server 2008 R2 or later).

Citrix recommends installing the FAS on a server that does not contain other Citrix components.
The Windows Server should be secured. It will have access to a registration authority certificate and a private key that allows it to automatically issue certificates for domain users, and it will have access to those user certificates and private keys.
The FAS PowerShell SDK requires Windows PowerShell 64-bit installed on the FAS server.
A Microsoft Enterprise Certification Authority is required to issue user certificates.
In the XenApp or XenDesktop Site:

The Delivery Controllers must be minimum version 7.15.
The VDAs must be minimum version 7.15. Check that the Federated Authentication Service Group Policy configuration has been applied correctly to the VDAs before creating the Machine Catalog in the usual way; see the Configure Group Policy section for details.
The StoreFront server must be minimum version 3.12 (this is the version provided with the XenApp and XenDesktop 7.15 ISO).
When planning your deployment of this service, review the security considerations section.

References:

Active Directory Certificate Services
https://technet.microsoft.com/en-us/library/hh831740.aspx

Configuring Windows for Certificate Logon Article
Implementation Considerations

FAS Config tool is running very slow
Potential issue: Missing Microsoft Enterprise CA
Solution: Remove legacy CA servers or do FAS configuration tasks manually
Scalability
Speed

Cached certificate login is almost as a fast as password
certificate generation takes about one second
certificates are automatically cast for re-use (Default should be 1 week)
Scalability

CA load is modest
can configure CA to not keep the certificates
or delete occasionally and backup see a to trigger space reclaim
FAS load is modest when using cash certificates
Can generate certificates from PowerShell
FAS load is significant when generating certificates
Recommend one to 1-2 FAS servers for storefront
Recommend dedicated CA servers used by FAS




Considerations

Cold Start during a period of 60-90 min

4 FAS Servers and 2 CA Servers (all with 8 vCPU each) can login 200k users under specific lab conditions.
FAS CPU usage 55% on four 8 vCPU FAS servers
CA CPU usage 7% on two 8 vCPU CA's
AD Domain Controller CPU usage due to XDC will increase 30% creating Certs
Warm start over a period of 25-30 min

FAS CPU usage 10% on four 8 vCPU FAS servers
Neg CA CPU usage is seen 
Security Considerations

Secure FAS servers the same way as a Domain Controller
Leverage ACLs in FAS rules to limit the scope of access
Internal deployment


The FAS allows users to securely authenticate to StoreFront using a variety of authentication options (including Kerberos single sign-on) and connect through to a fully authenticated Citrix HDX session.



This allows Windows authentication without prompts to enter user credentials or smart card PINs, and without using “saved password management” features such as the Single Sign-on Service. This can be used to replace the Kerberos Constrained Delegation logon features available in earlier versions of XenApp.

All users have access to public key infrastructure (PKI) certificates within their session, regardless of whether or not they log on to the endpoint devices with a smart card. This allows smooth migration to two-factor authentication models, even from devices such as smartphones and tablets that do not have a smart card reader.

This deployment adds a new server running the FAS, which is authorized to issue smart card class certificates on behalf of users. These certificates are then used to log on to user sessions in a Citrix HDX environment as if a smart card logon was used.

localized image

The XenApp or XenDesktop environment must be configured similarly as smart card logon

In an existing deployment, this usually involves only ensuring that a domain-joined Microsoft certificate authority (CA) is available, and that domain controllers have been assigned domain controller certificates. (See the “Issuing Domain Controller Certificates” section in CTX206156.)

Related information:

Keys can be stored in a Hardware Security Module (HSM) or built-in Trusted Platform Module (TPM).
NetScaler Gateway deployment
The NetScaler deployment is similar to the internal deployment but adds Citrix NetScaler Gateway paired with StoreFront, moving the primary point of authentication to NetScaler itself. Citrix NetScaler includes sophisticated authentication and authorization options that can be used to secure remote access to a company’s web sites.

This deployment can be used to avoid multiple PIN prompts that occur when authenticating first to NetScaler and then logging in to a user session. It also allows the use of advanced NetScaler authentication technologies without additionally requiring AD passwords or smart cards.

localized image

The XenApp or XenDesktop environment must be configured similarly to a smart card logon.

In an existing deployment, this usually involves only ensuring that a domain-joined Microsoft certificate authority (CA) is available, and that domain controllers have been assigned Domain Controller certificates. (See the “Issuing Domain Controller Certificates” section in CTX206156).

When configuring NetScaler as the primary authentication system, ensure that all connections between NetScaler and StoreFront are secured with TLS. In particular, ensure that the Callback Url is correctly configured to point to the NetScaler server, as this can be used to authenticate the NetScaler server in this deployment.

localized image

Related information:

To configure NetScaler Gateway, see How to Configure NetScaler Gateway 10.5 to use with StoreFront 3.6 and XenDesktop 7.6.
ADFS SAML deployment
A key NetScaler authentication technology allows integration with Microsoft ADFS, which can act as a SAML Identity Provider (IdP). A SAML assertion is a cryptographically-signed XML block issued by a trusted IdP that authorizes a user to log on to a computer system. This means that the FAS server now allows the authentication of a user to be delegated to the Microsoft ADFS server (or other SAML-aware IdP).

localized image



Installation
Install the Federated Authentication Service
For security, Citrix recommends that the FAS be installed on a dedicated server that is secured in a similar way to a domain controller or certificate authority. The FAS can be installed from the Federated Authentication Service button on the autorun splash screen when the ISO is inserted.

This will install the following components:

Federated Authentication Service
PowerShell snap-in cmdlets to remotely configure the Federated Authentication Service
Federated Authentication Service administration console
Federated Authentication Service Group Policy templates (CitrixFederatedAuthenticationService.admx/adml)
Certificate template files for simple certificate authority configuration
Performance counters and event logs


Enable the Federated Authentication Service plug-in on a StoreFront store
To enable Federated Authentication Service integration on a StoreFront Store, run the following PowerShell cmdlets as an Administrator account. If you have more than one store, or if the store has a different name, the path text below may differ.

Copy

```
Get-Module "Citrix.StoreFront.*" -ListAvailable | Import-Module

$StoreVirtualPath = "/Citrix/Store"

$store = Get-STFStoreService -VirtualPath $StoreVirtualPath

$auth = Get-STFAuthenticationService -StoreService $store

Set-STFClaimsFactoryNames -AuthenticationService $auth -ClaimsFactoryName "FASClaimsFactory"

Set-STFStoreLaunchOptions -StoreService $store -VdaLogonDataProvider "FASLogonDataProvider"
```
To stop using the FAS, use the following PowerShell script:

Copy

```
Get-Module "Citrix.StoreFront.*" -ListAvailable | Import-Module

$StoreVirtualPath = "/Citrix/Store"

$store = Get-STFStoreService -VirtualPath $StoreVirtualPath

$auth = Get-STFAuthenticationService -StoreService $store

Set-STFClaimsFactoryNames -AuthenticationService $auth -ClaimsFactoryName "standardClaimsFactory"

Set-STFStoreLaunchOptions -StoreService $store -VdaLogonDataProvider ""
```
Configure the Delivery Controller
To use the Federated Authentication Service, configure the XenApp or XenDesktop Delivery Controller to trust the StoreFront servers that can connect to it: run the Set-BrokerSite -TrustRequestsSentToTheXmlServicePort $true PowerShell cmdlet.

Configure Group Policy
After you install the Federated Authentication Service, you must specify the full DNS addresses of the FAS servers in Group Policy using the Group Policy templates provided in the installation.

Important: Ensure that the StoreFront servers requesting tickets and the VDAs redeeming tickets have the identical configuration of DNS addresses, including the automatic server numbering applied by the Group Policy object.

For simplicity, the following examples configure a single policy at the domain level that applies to all machines; however, that is not required. The FAS will function as long as the StoreFront servers, VDAs, and the machine running the FAS administration console see the same list of DNS addresses. Note that the Group Policy object adds an index number to each entry, which must also match if multiple objects are used.

Step 1. On the server where you installed the FAS, locate the C:\Program Files\Citrix\Federated Authentication Service\PolicyDefinitions\CitrixFederatedAuthenticationService.admx file and the en-US folder.

localized image

Step 2. Copy these to your domain controller and place them in the C:\Windows\PolicyDefinitions and en-US subfolder.

Step 3. Run the Microsoft Management Console (mmc.exe from the command line). From the menu bar, select File > Add/Remove Snap-in. Add the Group Policy Management Editor.

When prompted for a Group Policy Object, select Browse and then select Default Domain Policy. Alternatively, you can create and select an appropriate policy object for your environment, using the tools of your choice. The policy must be applied to all machines running affected Citrix software (VDAs, StoreFront servers, administration tools).

localized image

Step 4. Navigate to the Federated Authentication Service policy located in Computer Configuration/Policies/Administrative Templates/Citrix Components/Authentication.

localized image

Step 5. Open the Federated Authentication Service policy and select Enabled. This allows you to select the Show button, where you configure the DNS addresses of your FAS servers.

localized image

Step 6. Enter the DNS addresses of the servers hosting your Federated Authentication Service.

Remember: If you enter multiple addresses, the order of the list must be consistent between StoreFront servers and VDAs. This includes blank or unused list entries.

Step 7. Click OK to exit the Group Policy wizard and apply the group policy changes. You may need to restart your machines (or run gpupdate /force from the command line) for the change to take effect.

Enable in-session certificate support and disconnect on lock
localized image

In-session certificate support
The Group Policy template includes support for configuring the system for in-session certificates. This places certificates in the user’s personal certificate store after logon for application use. For example, if you require TLS authentication to web servers within the VDA session, the certificate can be used by Internet Explorer. By default, VDAs will not allow access to certificates after logon.

Disconnect on lock
If this policy is enabled the user’s session is automatically disconnected when they lock the screen. This functionality provides similar behavior to the “disconnect on smart card removal” policy and is useful for situations where users do not have Active Directory logon credentials.

NOTE:

The disconnect on lock policy applies to all sessions on the VDA.

Using the Federated Authentication Service administration console
The Federated Authentication Service administration console is installed as part of the Federated Authentication Service. An icon (Citrix Federated Authentication Service) is placed in the Start Menu.

The console attempts to automatically locate the FAS servers in your environment using the Group Policy configuration. If this fails, see the Configure Group Policy section.

localized image

If your user account is not a member of the Administrators group on the machine running the Federated Authentication Service, you will be prompted for credentials.

localized image

The first time the administration console is used, it guides you through a three-step process that deploys certificate templates, sets up the certificate authority, and authorizes the Federated Authentication Service to use the certificate authority. Some of the steps can alternatively be completed manually using OS configuration tools.

localized image

Deploy certificate templates
To avoid interoperability issues with other software, the Federated Authentication Service provides three Citrix certificate templates for its own use.

Citrix_RegistrationAuthority_ManualAuthorization
Citrix_RegistrationAuthority
Citrix_SmartcardLogon
These templates must be registered with Active Directory. If the console cannot locate them, the Deploy certificate templates tool can install them. This tool must be run as an account that has permissions to administer your Enterprise forest.

localized image

The configuration of the templates can be found in the XML files with extension .certificatetemplate that are installed with the Federated Authentication Service in:

C:\Program Files\Citrix\Federated Authentication Service\CertificateTemplates

If you do not have permission to install these template files, give them to your Active Directory Administrator.

To manually install the templates, you can use the following PowerShell commands:

Copy

```
$template = [System.IO.File]::ReadAllBytes("$Pwd\Citrix_SmartcardLogon.certificatetemplate")

 $CertEnrol = New-Object -ComObject X509Enrollment.CX509EnrollmentPolicyWebService

 $CertEnrol.InitializeImport($template)

 $comtemplate = $CertEnrol.GetTemplates().ItemByIndex(0)
$writabletemplate = New-Object -ComObject X509Enrollment.CX509CertificateTemplateADWritable

 $writabletemplate.Initialize($comtemplate)

 $writabletemplate.Commit(1, $NULL)
 ```
Set up Active Directory Certificate Services
After installing the Citrix certificate templates, they must be published on one or more Microsoft Certification Authority servers. Refer to the Microsoft documentation on how to deploy Active Directory Certificate Services.

If the templates are not published on at least one server, the Setup certificate authority tool offers to publish them. You must run this tool as a user that has permissions to administer the certificate authority.

(Certificate templates can also be published using the Microsoft Certification Authority console.)

localized image

Authorize the Federated Authentication Service
The final setup step in the console initiates the authorization of the Federated Authentication Service. The administration console uses the Citrix_RegistrationAuthority_ManualAuthorization template to generate a certificate request, and then sends it to one of the certificate authorities that publish that template.

localized image

After the request is sent, it appears in the Pending Requests list of the Microsoft Certification Authority console. The certificate authority administrator must choose to Issue or Deny the request before configuration of the Federated Authentication Service can continue. Note that the authorization request appears as a Pending Request from the FAS machine account.

localized image

Right-click All Tasks and then select Issue or Deny for the certificate request. The Federated Authentication Service administration console automatically detects when this process completes. This can take a couple of minutes.

localized image

Configure user rules
A user rule authorizes the issuance of certificates for VDA logon and in-session use, as directed by StoreFront. Each rule specifies the StoreFront servers that are trusted to request certificates, the set of users for which they can be requested, and the set of VDA machines permitted to use them.

To complete the setup of the Federated Authentication Service, the administrator must define the default rule by switching to the User Rules tab of the FAS administration console, selecting a certificate authority to which the Citrix_SmartcardLogon template is published, and editing the list of StoreFront servers. The list of VDAs defaults to Domain Computers and the list of users defaults to Domain Users; these can be changed if the defaults are inappropriate.

localized image

Fields:

Certificate Authority and Certificate Template: The certificate template and certificate authority that will be used to issue user certificates. This should be the Citrix_SmartcardLogon template, or a modified copy of it, on one of the certificate authorities that the template is published to.

The FAS supports adding multiple certificate authorities for failover and load balancing, using PowerShell commands. Similarly, more advanced certificate generation options can be configured using the command line and configuration files. See the PowerShell and Hardware security modules sections.

In-Session Certificates: The Available after logon check box controls whether a certificate can also be used as an in-session certificate. If this check box is not selected, the certificate will be used only for logon or reconnection, and the user will not have access to the certificate after authenticating.

List of StoreFront servers that can use this rule: The list of trusted StoreFront server machines that are authorized to request certificates for logon or reconnection of users. Note that this setting is security-critical, and must be managed carefully.

localized image

List of VDA desktops and servers that can be logged into by this rule: The list of VDA machines that can log users on using the Federated Authentication Service system.

localized image

List of users that StoreFront can log in using this rule: The list of users who can be issued certificates through the Federated Authentication Service.

localized image

Advanced use
You can create additional rules to reference different certificate templates and authorities, which may be configured to have different properties and permissions. These rules can be configured for use by different StoreFront servers, which will need to be configured to request the new rule by name. By default, StoreFront requests default when contacting the Federated Authentication Service. This can be changed using the Group Policy Configuration options.

To create a new certificate template, duplicate the Citrix_SmartcardLogon template in the Microsoft Certification Authority console, rename it (for example, Citrix_SmartcardLogon2), and modify it as required. Create a new user rule by clicking Add to reference the new certificate template.

Upgrade considerations
All Federated Authentication Service server settings are preserved when you perform an in-place upgrade.
Upgrade the Federated Authentication Service by running the full-product XenApp and XenDesktop installer.
Before upgrading the Federated Authentication Service from 7.15 LTSR to 7.15 LTSR CU2 (or a later supported CU), upgrade the Controller and VDAs (and other core components) to the required version.
Ensure that the Federated Authentication Service console is closed before you upgrade the Federated Authentication Service.
Ensure that at least one Federated Authentication Service server is available at all times. If no server is reachable by a Federation Authentication Service-enabled StoreFront server, users cannot log on or start applications.
Security considerations
The Federated Authentication Service has a registration authority certificate that allows it to issue certificates autonomously on behalf of your domain users. As such, it is important to develop and implement a security policy to protect the FAS servers, and to constrain their permissions.

Delegated Enrollment Agents
FAS issues user certificates by acting as an enrollment agent. The Microsoft Certification Authority allows control of which templates the FAS server can use, as well as limiting which users the FAS server can issue certificates for.

localized image

Citrix strongly recommends configuring these options so that the Federated Authentication Service can only issue certificates for the intended users. For example, it is good practice to prevent the Federated Authentication Service from issuing certificates to users in an Administration or Protected Users group.

Access Control List configuration
As described in the Configure user rules section, you must configure a list of StoreFront servers that are trusted to assert user identities to the Federated Authentication Service when certificates are issued. Similarly, you can restrict which users will be issued certificates, and which VDA machines they can authenticate to. This is in addition to any standard Active Directory or certificate authority security features you configure.

Firewall settings
All communication to FAS servers uses mutually authenticated Windows Communication Foundation (WCF) Kerberos network connections over port 80.

Event log monitoring
The Federated Authentication Service and the VDA write information to the Windows Event Log. This can be used for monitoring and auditing information. The Event logs section lists event log entries that may be generated.

Hardware security modules
All private keys, including those of user certificates issued by the Federated Authentication Service, are stored as non-exportable private keys by the Network Service account. The Federated Authentication Service supports the use of a cryptographic hardware security module if your security policy requires it.

The low-level cryptographic configuration is available in the FederatedAuthenticationService.exe.config file. These settings apply when private keys are first created. Therefore, different settings can be used for registration authority private keys (for example, 4096 bit, TPM protected) and runtime user certificates.

ProviderLegacyCsp	When set to true, FAS will use the Microsoft CryptoAPI (CAPI). Otherwise, FAS will use the Microsoft Cryptography Next Generation API (CNG).
ProviderName	Name of the CAPI or CNG provider to use.
ProviderType	Refers to Microsoft KeyContainerPermissionAccessEntry.ProviderType Property PROV_RSA_AES 24. Should always be 24 unless you are using an HSM with CAPI and the HSM vendor specifies otherwise.
KeyProtection	Controls the “Exportable” flag of private keys. Also allows the use of Trusted Platform Module (TPM) key storage, if supported by the hardware.
KeyLength	Key length for RSA private keys. Supported values are 1024, 2048 and 4096 (default: 2048).


PowerShell SDK
Although the Federated Authentication Service administration console is suitable for simple deployments, the PowerShell interface offers more advanced options. When you are using options that are not available in the console, Citrix recommends using only PowerShell for configuration.

The following command adds the PowerShell cmdlets:

Add-PSSnapin Citrix.Authentication.FederatedAuthenticationService.V1

Use Get-Help <cmdlet name> to display cmdlet help. The following table lists several commands where * represents a standard PowerShell verb (such as New, Get, Set, Remove).

*-FasServer	Lists and reconfigures the FAS servers in the current environment.
*-FasAuthorizationCertificate	Manages the Registration Authority certificate.
*-FasCertificateDefinition	Controls the parameters that the FAS uses to generate certificates.
*-FasRule	Manages User Rules configured on the Federated Authentication Service.
*-FasUserCertificate	Lists and manages certificates cached by the Federated Authentication Service.


PowerShell cmdlets can be used remotely by specifying the address of a FAS server.

You can also download a zip file containing all the FAS PowerShell cmdlet help files; see the PowerShell SDK article.



Federated Authentication Service PowerShell cmdlets
C

You can use the Federated Authentication Service administration console for simple deployments; however, the PowerShell interface offers more advanced options. If you plan to use options that are not available in the console, Citrix recommends using only PowerShell for configuration.

The following command adds the FAS PowerShell cmdlets:

Copy

Add-PSSnapin Citrix.Authentication.FederatedAuthenticationService.V1
In a PowerShell window, you can use Get-Help <cmdlet name> to display cmdlet help.

The zip file linked below contains help files for all FAS PowerShell SDK cmdlets. To use it, click the link, which will download the zip file. Then extract its content to a local folder. The index.html file lists all cmdlets, with links to individual cmdlet help files.

Troubleshooting
Performance counters
The Federated Authentication Service includes a set of performance counters for load tracking purposes.

localized image



The following table lists the available counters. Most counters are rolling averages over five minutes.

Active Sessions	Number of connections tracked by the Federated Authentication Service.
Concurrent CSRs	Number of certificate requests processed at the same time.
Private Key ops	Number of private key operations performed per minute.
Request time	Length of time to generate and sign a certificate.
Certificate Count	Number of certificates cached in the Federated Authentication Service.
CSR per minute	Number of CSRs processed per minute.
Low/Medium/High	Estimates of the load that the Federated Authentication Service can accept in terms of “CSRs per minute”. Exceeding the “High Load” threshold may result in session launches failing.


Event logs
The following tables list the event log entries generated by the Federated Authentication Service.

Administration events
[Event Source: Citrix.Authentication.FederatedAuthenticationService]

These events are logged in response to a configuration change in the Federated Authentication Service server.

[S001] ACCESS DENIED: User [{0}] is not a member of the Administrators group
[S002] ACCESS DENIED: User [{0}] is not an Administrator of Role [{1}]
[S003] Administrator [{0}] setting Maintenance Mode to [{1}]
[S004] Administrator [{0}] enrolling with CA [{1}] templates [{2} and {3}]
[S005] Administrator [{0}] de-authorizing CA [{1}]
[S006] Administrator [{0}] creating new Certificate Definition [{1}]
[S007] Administrator [{0}] updating Certificate Definition [{1}]
[S008] Administrator [{0}] deleting Certificate Definition [{1}]
[S009] Administrator [{0}] creating new Role [{1}]
[S010] Administrator [{0}] updating Role [{1}]
[S011] Administrator [{0}] deleting Role [{1}]
[S012] Administrator [{0}] creating certificate [upn: {0} sid: {1} role: {2}][Certificate Definition: {3}]
[S013] Administrator [{0}] deleting certificates [upn: {0} role: {1} Certificate Definition: {2}]
[S401] Performing configuration upgrade – [From version {0}][to version {1}]
[S402] ERROR: The Citrix Federated Authentication Service must be run as Network Service [currently running as: {0}]


Creating identity assertions [Federated Authentication Service]
These events are logged at runtime on the Federated Authentication Service server when a trusted server asserts a user logon.

[S101] Server [{0}] is not authorized to assert identities in the role [{1}]
[S102] Server [{0}] failed to assert UPN [{1}] (Exception: {2}{3})
[S103] Server [{0}] requested UPN [{1}] SID {2}, but lookup returned SID {3}
[S104] Server [{0}] failed to assert UPN [{1}] (UPN not allowed by role [{2}])
[S105] Server [{0}] issued identity assertion [upn: {0}, role {1}, Security Context: [{2}]

[S120] Issuing certificate to [upn: {0} role: {1} Security Context: [{2}]]
[S121] Issuing certificate to [upn: {0} role: {1}] on behalf of account {2}
[S122] Warning: Server is overloaded [upn: {0} role: {1}][Requests per minute {2}].


Acting as a relying party [Federated Authentication Service]
These events are logged at runtime on the Federated Authentication Service server when a VDA logs on a user.

[S201] Relying party [{0}] does not have access to a password.
[S202] Relying party [{0}] does not have access to a certificate.
[S203] Relying party [{0}] does not have access to the Logon CSP
[S204] Relying party [{0}] accessing the Logon CSP [Operation: {1}]
[S205] Calling account [{0}] is not a relying party in the role [{1}]
[S206] Calling account [{0}] is not a relying party
[S207] Relying party [{0}] asserting identity [upn: {1}] in role: [{2}]
[S208] Private Key operation failed [Operation: {0}][upn: {1} role: {2} certificateDefinition {3}][Error {4} {5}].


In-session certificate server [Federated Authentication Service]
These events are logged on the Federated Authentication Service server when a user uses an in-session certificate.

[S301] Access Denied: User [{0}] does not have access to a Virtual Smart Card
[S302] User [{0}] requested unknown Virtual Smart Card [thumbprint: {1}]
[S303] User [{0}] does not match Virtual Smart Card [upn: {1}]
[S304] User [{1}] running program [{2}] on computer [{3}] using Virtual Smart Card [upn: {4} role: {5}] for private key operation: [{6}]
[S305] Private Key operation failed [Operation: {0}][upn: {1} role: {2} containerName {3}][Error {4} {5}].


Log on [VDA]
[Event Source: Citrix.Authentication.IdentityAssertion]

These events are logged on the VDA during the logon stage.

[S101] Identity Assertion Logon failed. Unrecognised Federated Authentication Service [id: {0}]
[S102] Identity Assertion Logon failed. Could not lookup SID for {0} [Exception: {1}{2}]
[S103] Identity Assertion Logon failed. User {0} has SID {1}, expected SID {2}
[S104] Identity Assertion Logon failed. Failed to connect to Federated Authentication Service: {0} [Error: {1} {2}]
[S105] Identity Assertion Logon. Logging in [Username: {0}][Domain: {1}]
[S106] Identity Assertion Logon. Logging in [Certificate: {0}]
[S107] Identity Assertion Logon failed. [Exception: {1}{2}]
[S108] Identity Assertion Subsystem. ACCESS_DENIED [Caller: {0}]




In-session certificates [VDA]
These events are logged on the VDA when a user attempts to use an in-session certificate.

[S201] Virtual Smart Card Authorized [User: {0}][PID: {1} Name:{2}][Certificate {3}]
[S202] Virtual Smart Card Subsystem. No smart cards available in session {0}
[S203] Virtual Smart Card Subsystem. Access Denied [caller: {0}, session {1}, expected: {2}]
[S204] Virtual Smart Card Subsystem. Smart card support disabled.




Smart Card Configuration for Testing Citrix Environments
Smart card config Citrix Env.pdf

Information
The attached PDF describes how to set up a smartcard card environment from scratch for testing Citrix Products. In particular, this applies to Citrix Ready testing for Thin Client terminals. Note that this deployment is illustrative only, and may not be appropriate for a production deployment This document provides a step-by-step guide to configuring a Windows Domain, Microsoft Certificate Authority, Internet Information Services, Citrix Storefront, Citrix XenDesktop, and Citrix Receivers on Windows, Macintosh OSX, ChromeOS, and Linux for Smart Card support. This deployment is designed for smart cards conforming to the NIST PIV standard, as required by Citrix Ready certification for Thin Clients.

Smart Card Test Device Selection
Smart card driver software for PIV cards is supplied by the Operating System vendors. For the purposes of the documentation, the Yubikey 4 smart card is used and its software is open source, and available for free download from their website. Yubikey 4 is an all-in-one USB CCID PIV device that can easily be purchased from Amazon or other retail vendors and doesn’t compete with Enterprise smartcard vendor partners. Note that some organizations may require more advanced smart card driver software that can be installed according to the smart card driver vendor’s documentation. ![image](https://user-images.githubusercontent.com/82611568/114958377-7b554b80-9e28-11eb-8486-d8886c031063.png)


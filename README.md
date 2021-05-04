# CitrixFAS

The Federated Authentication Service (FAS) is a Citrix component that integrates with your Active Directory certificate authority (CA), allowing users to be seamlessly authenticated within a Citrix environment. This document describes various authentication architectures that may be appropriate for your deployment.

When enabled, the FAS delegates user authentication decisions to trusted StoreFront servers. StoreFront has a comprehensive set of built-in authentication options built around modern web technologies and is easily extensible using the StoreFront SDK or third-party IIS plugins. The basic design goal is that any authentication technology that can authenticate a user to a web site can now be used to log in to a Citrix XenApp or XenDesktop deployment.

This document covers some examples of top-level deployment architectures, in increasing complexity.



How it works
The FAS is authorized to issue smart card class certificates automatically on behalf of Active Directory users who are authenticated by StoreFront. This uses similar APIs to tools that allow administrators to provision physical smart cards. When a user is brokered to a Citrix XenApp or XenDesktop Virtual Delivery Agent (VDA), the certificate is attached to the machine, and the Windows domain sees the logon as a standard smart card authentication. The Citrix Federated Authentication Service is a privileged component designed to integrate with Active Directory Certificate Services. It dynamically issues certificates for users, allowing them to log on to an Active Directory environment as if they had a smart card. This allows StoreFront to use a broader range of authentication options, such as SAML (Security Assertion Markup Language) assertions. SAML is commonly used as an alternative to traditional Windows user accounts on the Internet. The following diagram shows the Federated Authentication Service integrating with a Microsoft Certification Authority and providing support services to StoreFront and XenApp and XenDesktop Virtual Delivery Agents (VDAs).

For more information check <a href="https://github.com/dalhakeem/CitrixFAS/wiki">Wiki Link</a> 2021-05-04

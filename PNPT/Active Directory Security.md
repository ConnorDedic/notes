Server Management to edit settings 

Download [MCST](https://www.microsoft.com/en-us/download/details.aspx?id=55319) scripts to improve baselines and add policies

| Exploit                                                                  | Remediation                                                                                                                                                                                                             |
| ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Link-Local Multicast Name Resolution/LLMNR (steals a user:hash via MiTM) | make user hashcrack proof<br>                                                                                                                                                                                           |
| Server Message Block/SMB Relay (same as above but passing hash instead)  | enable/enforce SMB message signing, disable NTLM auth, Tiered accounts, Restrict local admins                                                                                                                           |
| MiTM6                                                                    | block DHCPv6 traffic and incoming router advertisements via GP                                                                                                                                                          |
| Kerberoasting                                                            | strong passwords for DA and SA accounts, SA accounts do not run as DA (least privilege)                                                                                                                                 |
| Token Impersonation                                                      | limit user/group account creation, tiered accounts, restrict local admins                                                                                                                                               |
| link(LNK) file                                                           | dont click unknown shared files                                                                                                                                                                                         |
| Golden Ticket                                                            | [Change KRBTGT account password 2x](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/forest-recovery-guide/ad-forest-recovery-reset-the-krbtgt-password)                                          |
| ZeroLogon                                                                | [1](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/forest-recovery-guide/ad-forest-recovery-reset-the-krbtgt-password) [2](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2020-1472) |

Kerberos
Kerberos is a network authentication protocol designed by the Massachusetts Institute of
Technology (MIT) in 1988 and is widely used in modern Windows domains and enterprise
environments. Unlike NTLM, which relies on password hashing and challenge-response
mechanisms, Kerberos uses ticket-based authentication to provide mutual verification
between clients and servers, enhancing security and efficiency. Kerberos was ahead of its
time, possessing a security model that accounted for publicly available networks rather
than the much more commonly implemented isolated networks of the 80’s and early 90’s.
Kerberos Authentication Process
Kerberos is named after the 3 headed dog guarding the gates to the underworld as found in
in Greek mythology. As such, it utilizes a trusted third-party model involving three main
components:
1. Key Distribution Center (KDC): The central authority responsible for issuing and
validating authentication tickets. It consists of two subcomponents:
a. Authentication Server (AS): Verifies the user's identity and issues a Ticket
Granting Ticket (TGT).
b. Ticket Granting Service (TGS): Issues service tickets that allow access to
specific resources.
2. Client: The user or system requesting access to resources.
3. Service Server (SS): The server providing access to a resource or service.
Authentication Flow:
4. AS Request: The client sends a request to the KDC’s Authentication Server (AS)
with its credentials (encrypted with the client’s password).
5. TGT Issuance: The AS verifies the client’s credentials and issues a Ticket Granting
Ticket (TGT), which is encrypted with the KDC’s secret key and can only be
decrypted by the KDC.
6. TGS Request: The client uses the TGT to request access to specific services. It
sends the TGT and a request for a service ticket to the TGS.
7. Service Ticket Issuance: The TGS issues a service ticket, encrypted with the
service’s secret key.
8. Access Service: The client presents the service ticket to the target service, which
verifies it and grants access
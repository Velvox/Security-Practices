# This document contains policies, scripts and tweaks to enhance security and privacy in Edge

#### Edge is NOT an privacy focussed browser so we can only change things to make it better not to make it good.

If you want to gain an better understanding of what these policies do? [Check this article](https://learn.microsoft.com/nl-nl/DeployEdge/microsoft-edge-policies)

## Basic security improvements
### This blocks, limits or enables certain functions via Registry editor with "Enterprise" policies.

[This script](Basic.reg) writes the necassary registry entries to enable, block or limit certain functions e.g. disabling "DiagnosticData"

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Edge]
"BlockThirdPartyCookies"=dword:00000001
"EnhanceSecurityMode"=dword:00000002
"AutofillCreditCardEnabled"=dword:00000000
"AutofillMembershipsEnabled"=dword:0000000
"AutomaticHttpsDefault"=dword:0000002
"TrackingPrevention"=dword:0000003
"UserFeedbackAllowed"=dword:0000001
"PersonalizationReportingEnabled"=dword:0000001
"DiagnosticData"=dword:0000000

```

## TLS Cipher Suite Deny List
### This blocks certain TLS ciphers via Registry editor with "Enterprise" policies[].

[This script](TLSCipherSuiteDenyList.reg) writes the necessary registry entries to block certain "weak" ciphers so the browser doesnt use it anymore

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Edge\TLSCipherSuiteDenyList]
"1"="0xc013"  ; TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
"2"="0xc014"  ; TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
"3"="0x009c"  ; TLS_RSA_WITH_AES_128_GCM_SHA256
"4"="0x009d"  ; TLS_RSA_WITH_AES_256_GCM_SHA384
"5"="0x002f"  ; TLS_RSA_WITH_AES_128_CBC_SHA
"6"="0x0035"  ; TLS_RSA_WITH_3DES_EDE_CBC_SHA
"7"="0x002b"  ; TLS_RSA_WITH_AES_256_CBC_SHA
"8"="0x000a"  ; TLS_RSA_WITH_DES_CBC_SHA
"9"="0x0005"  ; TLS_RSA_WITH_RC4_128_SHA
"10"="0x0004" ; TLS_RSA_WITH_RC4_128_MD5
"11"="0x0002" ; TLS_RSA_WITH_NULL_MD5
"12"="0x0001" ; TLS_RSA_WITH_NULL_SHA
"13"="0x0023" ; TLS_RSA_EXPORT_WITH_RC4_40_MD5
"14"="0x0026" ; TLS_RSA_EXPORT_WITH_DES40_CBC_SHA
"15"="0x0038" ; TLS_DH_anon_WITH_AES_128_CBC_SHA
"16"="0x0037" ; TLS_D
"17"="0xc02b" ; TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
"18"="0xc02f" ; TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256

```

Finaly if you want to check your ciphers go to this website [SSL Labs Browser test](https://clienttest.ssllabs.com:8443/ssltest/viewMyClient.html) Scroll down to "Protocol Features > Cipher Suites" and all exept "TLS_GREASE_*" should be green. If there are any with an different color? You may implemented this the wrong way, try again or report an error to us.
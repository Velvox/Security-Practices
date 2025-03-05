# Code signing guide

First generate an self signed certifiate (Or buy one from an reputable seller and go ahead to step: 2).

### 1. Generate the certificate
```ps
New-SelfSignedCertificate `
    -Type CodeSigningCert `
    -Subject "CN=Name_of_the_certificate, O=YourOrganization, L=YourCity, S=YourState, C=Your2LetterCountryCode, Email=email.ThatShowsUpOnTheCertificate@domain.tdl" `
    -KeyAlgorithm RSA `
    -KeyLength 4096 `
    -HashAlgorithm SHA512 `
    -KeyUsage DigitalSignature `
    -CertStoreLocation "Cert:\CurrentUser\My" `
    -NotAfter (Get-Date).AddYears(5)
```
* **BE ADVISED**: For this guide we used an certificate timeframe after 5 years. Choose an exeptable timeframe for your Threat model.

#### Subject explanation
* Purpose: Defines the identity of the certificate holder. It includes multiple parts:
    * **CN** (Common Name): The name of the certificate (e.g., "Name_of_the_certificate").
    * **O** (Organization): The name of the organization or company.
    * **L** (Locality): The city where the organization is located.
    * **S** (State): The state or region.
    * **C** (Country): The country in a 2-letter code (e.g., "US").
    * **Email**: An email address that will appear in the certificate.

Now this certificate will be stored with its private key in the "Current User" Windows Certificate Store (Part of Microsoft Management Consol/MMC).

Now we have the certificate in the Certificate Store now we need to export this certificate without it's private key. And import it to the `Local Computer/Trusted Root Certification Authorities` to ensure it is trusted.

Now we can sign our first script!

### 2. Sign the script

First we need to check for the certificate to be in the right place. And check if there are more certificates.

```ps
Get-ChildItem –Path Cert:\LocalMachine\My -CodeSigningCert
```
This will return something like.
```ps
# DO NOT ATTEMPT TO RUN THIS
PS C:\Users\User\Desktop> Get-ChildItem –Path Cert:\CurrentUser\My -CodeSigningCert


   PSParentPath: Microsoft.PowerShell.Security\Certificate::CurrentUser\My

Thumbprint                                Subject
----------                                -------
D8F3E9D79D80161E32A6922207ACA76B45ACE7EF  CN=DEMO Internal Code Signing, O=DEMO, L=DEMO, S=DEMO, C=US
```

Now we can sign our script

```ps
$cert = (Get-ChildItem –Path Cert:\LocalMachine\My -CodeSigningCert)[0] # [0] Because we want the first certificate that showed up on the previous command.
$timestampUrl = "http://timestamp.digicert.com" # DiGicert's timestamp server, this will add an counter signature with the date and time of the signature. 
Set-AuthenticodeSignature -FilePath C:\Path\To\YourScript.ps1 -Certificate $cert -HashAlgorithm SHA256 -TimestampServer $timestampUrl # Use SHA256 because it is an secure and widely used hashing algorithm.
```

### 3. Check the signature

You can check the signature in multiple way's. 

#### Option 1
You can right-click on the file and open the `"Properties"` and go to `"Digital Signatures"`. This will show the name, digest algorithm and timestamp (if set).

#### Option 2 
You can run the following Powershell command.
```ps
Get-AuthenticodeSignature "C:\Path\To\YourScript.ps1"
``` 

This will give an response that will look like the following.

```ps
# DO NOT ATTEMPT TO RUN THIS
PS C:\Users\User\Desktop> Get-AuthenticodeSignature "C:\Path\To\YourScript.ps1"


    Directory: C:\Path\To


SignerCertificate                         Status                                 Path
-----------------                         ------                                 ----
D8F3E9D79D80161E32A6922207ACA76B45ACE7EF  Valid                                  YourScript.ps1

``` 

## Pro tips

### 1. Sign multiple files at once
Using astrix's (*) you can sign multiple files at once.
```ps
Set-AuthenticodeSignature -FilePath C:\Path\To\*.ps1 -Certificate $cert -HashAlgorithm SHA256
``` 
This command signs all the files that have the .ps1 (Powershell script) extention.
Response:
```ps
PS C:\Users\Koen> Set-AuthenticodeSignature -FilePath C:\Path\To\*.ps1 -Certificate $cert -HashAlgorithm SHA256 -TimestampServer $timestampUrl


    Directory: C:\Users\Koen\Desktop


SignerCertificate                         Status                                                                              Path
-----------------                         ------                                                                              ----
D8F3E9D79D80161E32A6922207ACA76B45ACE7EF  Valid                                                                               YourScript.ps1
D8F3E9D79D80161E32A6922207ACA76B45ACE7EF  Valid                                                                               YourScript2.ps1
```

### 2. Don't sign from the Certificate Store
Signing from the Certificate store makes key management simple but insecure. Every program or user could sign files using your private key. This whould be an security nightmare! Export the Certificate with Privatekey in an password protected file (Choose AES256-SHA256 when getting the option to set an password). 

Also export the Certificate without the Privatekey this is the file you want to use on the PC's that need to trust your signature otherwise your files will be handled like they are unsigned (If you use an certificate from an authority you don't need to import certificates because they will trust the signature by default).

So how are we going to sign scripts from an .pfx key file? We ofcorse use an PowerShell script😂

```ps
$pfxFilePath = Read-Host "Enter the full path to the .pfx certificate file"
$Password = Read-Host "Enter password for private key" -AsSecureString
$certWithKey = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
$certWithKey.Import($pfxFilePath, $Password, [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::Exportable)

$timestampUrl = "http://timestamp.digicert.com"
$scriptPath = Read-Host "Enter the full path of one or more files (using wildcards) you want to sign"
Set-AuthenticodeSignature -FilePath $scriptPath -Certificate $certWithKey -HashAlgorithm SHA256 -TimestampServer $timestampUrl

Write-Host "Script('s) signed successfully!"
Write-Host "To close, press ENTER"
[void][System.Console]::ReadLine()
```
You can put the script in an .ps1 file and call it when needed or paste the full code block in the PowerShell console.

### 2.1 Make the script simpler to use

If you are developing an bigger system and you need to sign multiple files multiple times a day it gets frustrating to copy/paste or type out the full path of everything every time, we got an fix for that!


Modify the following lines

```ps
$pfxFilePath = "C:\Path\To\Key.pfx"

$scriptPath = "D:\Path\To\Dev\Files"
```
**DO NOT HARDCODE YOUR PRIVATE KEY CREDENTIALS IN THE SCRIPT (OR ANY WHERE ELSE SAVE IT SECURELY IN YOUR PASSWORD MANAGER)**

### Sign multiple file extentions or folders apart from each other
For this one you need to change an bit more to the script but this will make your workflow more efficent and allows you to target files more specificly.

```ps
$scriptPath1 = "D:\Path\To\Dev\Files\*.bat" # This only signs .bat files
$scriptPath2 = "D:\Path\To\Dev\Files\*.ps1" # This only signs .ps1 files
$scriptPath3 = "D:\Path\To\Dev\Files\*.js"  # This only signs .js (JavaScript) files
$scriptPath4 = "D:\Path\To\Dev\Files\*.dll" # This only signs .dll files

Set-AuthenticodeSignature -FilePath $scriptPath1 -Certificate $certWithKey -HashAlgorithm SHA256 -TimestampServer $timestampUrl # Signs the files/paths defined in $scriptPath1
Set-AuthenticodeSignature -FilePath $scriptPath2 -Certificate $certWithKey -HashAlgorithm SHA256 -TimestampServer $timestampUrl # Signs the files/paths defined in $scriptPath2
Set-AuthenticodeSignature -FilePath $scriptPath3 -Certificate $certWithKey -HashAlgorithm SHA256 -TimestampServer $timestampUrl # Signs the files/paths defined in $scriptPath3
Set-AuthenticodeSignature -FilePath $scriptPath4 -Certificate $certWithKey -HashAlgorithm SHA256 -TimestampServer $timestampUrl # Signs the files/paths defined in $scriptPath4
```

Creating such simple scripts can speedup the workflow while keeping your scripts & key's secure and stay within Application Control, AppLocker and Execution policies!

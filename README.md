## Leveraging PTT to defeat DSE and run Kernel Drivers with Secure Boot Enabled

... With a Test Signing Certificate and No Extended Validation.

Code Signing is an amazing thing, But it has a glaring flaw depending on your motherboard which allows you to run Test Signed Kernel Drivers in a full trust environment with no indication to the OS that something may be wrong.

### 1. Create the Certificate

_If you already own any valid code signing certificate you can use that instead and should skip to Step 3_

Open Powershell and enter the following to create a Code Signing Certificate that is valid for 2 Years and meets the [EFI_CERT_X509](https://download.lenovo.com/pccbbs/thinkcentre_pdf/certificate_based_bios_management_guide.pdf) spec

```ps
# Define EFI_CERT_X509
$certSplat = @{
	DnsName = 'TheCert'
	KeyUsage = @('KeyEncipherment','DataEncipherment','KeyAgreement','CRLSign', 'CertSign', 'KeyAgreement', 'DigitalSignature')
	Type = 'CodeSigningCert'
	NotAfter = (Get-Date).AddYears(2)
  	KeyAlgorithm = 'RSA' 
  	KeyLength = 2048
}

# Create the self-signed EFI_CERT_X509
$cert = New-SelfSignedCertificate @certSplat
```

![](https://i.imgur.com/WpNQ3rD.png)

### 2. Export the Certificate

Run `certlm.msc`

![](https://i.imgur.com/WnZa3Px.png)

Select `Personal Certificates`

Open the Certificate and Select "Copy to File" on the "Details" tab.

![](https://i.imgur.com/3UJtxyS.png)

Complete the Wizard and when prompted select "No" to exporting the private key.

![](https://i.imgur.com/Jdy9W4d.png) 

Leave everything else as defaults and select a filename

![](https://i.imgur.com/5K309kR.png)

Finish the wizard

![](https://i.imgur.com/grcmM4p.png)

_Note: You can delete this certificate from the local store now if you are loading the driver before the OS_

### 3. USB Drive

Put the Certificate onto a USB stick, we are going to import the Certificate into the `BIOS` so the Kernel will trust our `Signature` and run our driver as if Microsoft had signed it themselves.

### 4. Restart PC and Enter the Bios

_(These steps may vary slightly depending on your BIOS but the concept is the same)_

Select the `Secure Boot Menu` in your Bios

In the `Key Management` section select `Authorized Signatures` (Or wherever the `Microsoft Production PCA Certificate` is located)

Select `Append/Add` from the Menu that pops up

Locate `TheCert` on your `USB` and select it

You'll be prompted to confirm you want to update the Certificate Store, Select `Yes`.

## 5. Success, We are now an Extended Validator

The certificate you just added to the `BIOS` can now be used for Extended Validation of Kernel Drivers without using any exploits, essentially bypassing Driver Signing Enforcement (DSE) because there is no third party involved.

![](https://i.imgur.com/v3qcVeM.jpg)

## 6. Sign your Driver

Restart your computer and sign your kernel driver with `TheCert`, You are done.

# Considerations

You could specially craft a Bios Update to add this certificate and then Flash the BIOS if your motherboard doesn't have this menu option because the platform itself supports changing the certificates in the store.

This certificate (and Microsofts one) don't need to exist in your local store.

Tested on a motherboard with the `Z490 Chipset`

This allows you to create drivers the OS isn't even aware of again.

This is not an exploit.

This is a proof of concept, I don't recommend doing it until you fully understand what you are doing.

Extended Validation only exists in your head.

#### Have Fun and remember, Gödel's theorem suggests certain information can travel faster than the speed of light.

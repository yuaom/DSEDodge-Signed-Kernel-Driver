## Leveraging PTT to defeat DSE and run Kernel Drivers with Secure Boot Enabled

... With a Test Signing Certificate and No Extended Validation.

Code Signing is an amazing thing, But it has a glaring flaw depending on your motherboard which allows you to run Test Signed Kernel Drivers in a full trust environment with no indication to the OS that something may be wrong.

_Before you start you need to install the [Windows Driver Kit](https://docs.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk)_

### 1. [Creating the Certificates](https://github.com/HypsyNZ/DSEDodge-Signed-Kernel-Driver/tree/main/CERT#readme)

To begin, we need to create our own `CA` and `SPC` and a `PFX` we can use as a `Production Certificate` later.

```ps
makecert -r -pe -n "CN=Demo_CA_Root" -ss CA -sr CurrentUser ^
   -a sha256 -cy authority -sky signature ^
   -sv Demo_CA_Root.pvk Demo_CA_Root.cer

makecert -pe -n "CN=Demo_SPC_Code_Signing" -a sha256 -cy end ^
   -sky signature ^
   -ic Demo_CA_Root.cer -iv Demo_CA_Root.pvk ^
   -sv Demo_SPC_Code_Signing.pvk Demo_SPC_Code_Signing.cer ^
   -eku 1.3.6.1.5.5.7.3.3

pvk2pfx -pvk Demo_SPC_Code_Signing.pvk -spc Demo_SPC_Code_Signing.cer ^
   -pfx Demo_SPC_PFX.pfx ^
   -po x
```

### 3. USB Drive

Put the `Demo_SPC_Code_Signing.cer` and `Demo_CA_Root.cer` Certificate onto a USB stick, we are going to import the Certificates into the `BIOS` so the Kernel will trust our `Signature` and install/run our driver as if Microsoft had signed it themselves.

### 4. Restart PC and Enter the Bios

_(These steps may vary slightly depending on your BIOS but the concept is the same)_

Select the `Secure Boot Menu` in your Bios

In the `Key Management` section select `Authorized Signatures` (Or wherever the `Microsoft Production PCA Certificate` is located)

Select `Append/Add` from the Menu that pops up

Locate `Demo_SPC_Code_Signing.cer` and `Demo_CA_Root.cer` on your `USB`, pressing enter twice when selecting them.

The second time your press enter you'll be prompted to confirm you want to update the Certificate Store, Select `Yes`.

### 5. Success, We are now an "Extended Validator" on this PC.

The certificates you just added to the `BIOS` can now be used for "Extended Validation" of Kernel Drivers without using any exploits, essentially bypassing Driver Signing Enforcement (DSE) because there is no third party involved and you just sign it yourself without the extra steps.

![](https://i.imgur.com/ydRADjq.jpg)

### 6. Trust the CA Certificate

Restart the computer and open the `Demo_CA_Root` certificate

Install the certificate to the `Trusted Root`

Now drivers signed by `Demo_SPC_PFX.pfx` will be trusted.

### 7. BUILDING THE DRIVER

Now you can Production Sign your driver instead of Test Signing

`Demo_SPC_Code_Signing` is the `Cross-Signing Certificate`

`Demo_SPC_PFX.pfx` is the `Production Certificate`

![](https://i.imgur.com/CSzLRM7.png)

### 7. CREATE AND SIGN THE SECURITY CATALOG

DSE will stop us from installing the Kernel Driver if the catalog isn't signed correctly, manually run these commands.

```cmd
inf2cat /os:10_x64 /driver:.\x64\Release

SignTool sign /fd sha256 /f .\CERT\Demo_SPC_PFX.pfx /p x /v .\x64\Release\KMDFDriver\kmdfdriver.cat
```

### 8. Install your Driver

Now you can install and run your driver without configuring [BCEDIT](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/the-testsigning-boot-configuration-option)

![](https://i.imgur.com/w52wRtC.png)

------

Tested on a motherboard with the `Z490 Chipset`

#### Have Fun and remember, Gödel's theorem suggests certain information can travel faster than the speed of light.

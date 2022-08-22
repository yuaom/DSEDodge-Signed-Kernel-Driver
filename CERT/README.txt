#--------------------------------------------------------------------------------------------------------
# Create Root Certificate Authority (CA) to add to the UEFI Certificates and Trusted Root of the Local Machine
# The Kernel will trust anything signed by this authority.
#--------------------------------------------------------------------------------------------------------

makecert -r -pe -n "CN=Demo_CA_Root" -ss CA -sr CurrentUser -a sha256 -cy authority -sky signature -sv Demo_CA_Root.pvk Demo_CA_Root.cer

#--------------------------------------------------------------------------------------------------------
# Create Cross Signing Certificate (SPC) that is Issued By our own Root Certificate Authority (CA)
#--------------------------------------------------------------------------------------------------------

makecert -pe -n "CN=Demo_SPC_Code_Signing" -a sha256 -cy end -sky signature -ic Demo_CA_Root.cer -iv Demo_CA_Root.pvk -sv Demo_SPC_Code_Signing.pvk Demo_SPC_Code_Signing.cer -eku 1.3.6.1.5.5.7.3.3

#--------------------------------------------------------------------------------------------------------
# Create the Production Code Signing Certificate that uses the SPC which is issued by our own CA
#--------------------------------------------------------------------------------------------------------

pvk2pfx -pvk Demo_SPC_Code_Signing.pvk -spc Demo_SPC_Code_Signing.cer -pfx Demo_SPC_PFX.pfx -po x

#--------------------------------------------------------------------------------------------------------
# Post Build: Create a Security Catalog for the Kernel Mode Driver
#--------------------------------------------------------------------------------------------------------

inf2cat /os:10_x64 /driver:.\x64\Release

#--------------------------------------------------------------------------------------------------------
# Post Build: Sign the Security Catalog with the Production Certificate (PFX) so DSE doesn't block us from installing it.
#--------------------------------------------------------------------------------------------------------

SignTool sign /fd sha256 /f .\CERT\Demo_SPC_PFX.pfx /p x /v .\x64\Release\KMDFDriver\kmdfdriver.cat

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

inf2cat /os:10_x64 /driver:.\x64\Release

SignTool sign /fd sha256 /f .\CERT\Demo_SPC_PFX.pfx /p x /v .\x64\Release\KMDFDriver\kmdfdriver.cat
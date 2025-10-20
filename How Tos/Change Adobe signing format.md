Set the registry key -

**Path** - `Computer\HKEY_CURRENT_USER\Software\Adobe\Adobe Acrobat\DC\Security\cPubSec`
**Value** - `aSignFormat`

Allowable values include:
- adbe.pkcs7.detached
- adbe.pkcs7.sha1
- adbe.x509.rsa_sha1
- ETSI.CAdES.detached

[https://www.adobe.com/devnet-docs/acrobatetk/tools/PrefRef/Windows/Security.html#idkeyname_1_24424](https://www.adobe.com/devnet-docs/acrobatetk/tools/PrefRef/Windows/Security.html#idkeyname_1_24424)
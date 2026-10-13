# TOTP
* **Phishing resistance** Low. This is a six digit code that a user can be tricked into typing into a malicious prompt and then retrieved by the attacker within a validation window.   
* **Offline capability** Full. There is no phone data or cloud capacity since it uses an authentication app that computes the code locally.
* **Implementation complexity** Low. The command is libpam-google-authenticator plus a few sshd_config lines.
* **Cost per user** Almost $0. It uses free authenticator apps and an open-source PAM module. The only cost is admin time.
* **Works with SSH** Yes, natively. Use the commands pam_google_authenticator via keyboard-interactive, and it combines cleanly with public key or SSH CA certificate auth using AuthenticationMethods publickey,keyboard-interactive:pam.
* **Regulatory compliance** For NIST when it has two factors it cannot reach AAL3 because it is not phishing-resistant and the secret is exportable. For HIPPA it supports the access control standard and the authentication standard. 

# FIDO2 
* **Phishing resistance**
* **Offline capability**
* **Implementation complexity**
* **Cost per user**
* **Works with SSH**
* **Regulatory compliance**

# Push Notification 
* **Phishing resistance**
* **Offline capability**
* **Implementation complexity**
* **Cost per user**
* **Works with SSH**
* **Regulatory compliance**

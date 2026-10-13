# TOTP
* **Phishing resistance** Low. This is a six digit code that a user can be tricked into typing into a malicious prompt and then retrieved by the attacker within a validation window.   
* **Offline capability** Full. There is no phone data or cloud capacity since it uses an authentication app that computes the code locally.
* **Implementation complexity** Low. The command is libpam-google-authenticator plus a few sshd_config lines.
* **Cost per user** Almost $0. It uses free authenticator apps and an open-source PAM module. The only cost is admin time.
* **Works with SSH** Yes, natively. Use the commands pam_google_authenticator via keyboard-interactive, and it combines cleanly with public key or SSH CA certificate auth using AuthenticationMethods publickey,keyboard-interactive:pam.
* **Regulatory compliance** For NIST when it has two factors it cannot reach AAL3 because it is not phishing-resistant and the secret is exportable. For HIPPA it supports the access control standard and the authentication standard. 

# FIDO2 
* **Phishing resistance** High. There is no code for a human to hand over. The private key never leaves the hardware, so a server can't harvest anything.
* **Offline capability** Full. Since the key only needs a USB or a NFC port it has no network dependancy at all.
* **Implementation complexity** Medium. Somewhat trivial work in buying keys, setting PINs, registering a spare per person, and tracking and revoking lost keys.
* **Cost per user** Anywhere from $50 to $90 per key and you would need two keys per user.
* **Works with SSH** Yes, natively. Use commands ssh-keygen -t ed25519-sk -O verify-required
* **Regulatory compliance** For NIST it supports cryptography requirements when the authenticator is hardware-bound with a non-exportable key and user verification. For HIPPA it is usable because certain events our easy to show an auditor.  

# Push Notification 
* **Phishing resistance** Medium. A basic push setup is vulnerable to prompt bombing and social engineering. Number matching or Duo push is an improvement from that but since it is using a separate advise it is not completely immune to impersonation.  
* **Offline capability** Poor. It needs the phone online and the server able to reach the vendor's cloud API. 
* **Implementation complexity** High. You need a integration key on every server, firewall rules for outbound access, and ongoing management of a third-party dependency.
* **Cost per user** Around $4 per month for push apps.
* **Works with SSH** Yes, but via add-ons. Such as pam_duo for Duo. They add a runtime dependency on a third-party service in the login path of every server.
* **Regulatory compliance** For NIST it cannot reach AAL3 since it uses an out-of-band authenticator through an app. For HIPPA there should be few issues but just have to make sure you do your due diligence for the vendor of the app.

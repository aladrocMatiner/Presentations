
# | INTRO

- WKD aka Web Key Directory: A simple HTTPS directory where your public keys live. Mail clients query it automatically.
- WKS aka Web Key Service: A server-side helper that lets you upload/refresh keys automatically instead of copying files by hand.

f you only need to serve your public key and don’t mind updating it manually, WKD alone is enough. If you want to push updates from your local machine (gpg --send-wks), you also need WKS.

# | Install

0. Install gpg
```bash
TODO: sudo apt gpg2
```

2. Creating a working directory for the keys used by the WKD (Web Key Directory),

```bash
mkdir /var/lib/gnupg/wks
chown webkey:webkey /var/lib/gnupg/wks
chmod 2750 /var/lib/gnupg/wks
```

2. Creating subdirectories for each domain you want to manage keys for: 

```bash
mkdir /var/lib/gnupg/wks/example.net
```

3. Running the command to create required subdirectories and permissions:

```bash
gpg-wks-server --list-domains
```
4. Configuring a submission address to receive key material for publication, e.g. creating a file named submission-address inside domain dirs containing an email address to send keys to.

5. Creating a GPG key for this submission address used by the service:

```bash
gpg --batch --passphrase '' --quick-gen-key key-submission@example.net
``` 
6. Installing the submission key into the WKD structure using its fingerprint:

```bash
gpg-wks-server --install-key <fingerprint> key-submission@example.net
```

# | Links
- https://wiki.gnupg.org/WKD
- https://wiki.gnupg.org/WKS
- https://www.redhat.com/en/blog/getting-started-gpg
- https://man.archlinux.org/man/gpg-wks-server.1.en

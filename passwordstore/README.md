# | Into

passwordstore is a bash script that makes extremely easy encrypt secrets on keystore style and also store this stored secrets on GIT
to have version hantering

# | Install

Dependencies:

- pass: Main package

- gnu2: Crypto gpg

- Git: Backups & Storage

## || Install Ubuntu

```bash
sudo apt update
sudo apt install pass gnupg2 git
```
## || Install SUSE

```bash
sudo zypper refresh
sudo zypper in pass gpg2 git
```

# | Generate GPG keys

## || The Interactive way

```bash
gpg --full-generate-key
```
## || The fast way

### | Modern
```bash
gpg --batch --gen-key <<EOF
Key-Type: eddsa
Key-Curve: ed25519
Subkey-Type: ecdh
Subkey-Curve: cv25519
Name-Real: Jose Rafael Romero Miret
Name-Email: jose.romero@aladroc.io
Name-Comment: This is a test (ECC)
Expire-Date: 0
%no-protection
%commit
EOF
```

### ||| Legacy

```bash
gpg --batch --gen-key <<EOF
Key-Type: default
Key-Length: 4096
Subkey-Type: default
Name-Real: Jose Rafael Romero Miret
Name-Email: jose.romero@aladroc.io
Name-Comment: This is a test
Expire-Date: 0
%no-protection
%commit
EOF
```

## || GPG commands to know

```bash
#Listing Certificates
gpg --list-keys
```

```bash
#Export your certificates
gpg --list-secret-keys --keyid-format=long
## example
sec   rsa4096/ABCD1234EFGH5678 2025-09-17 [SCEA]
      uid [ultimate] Jose Rafael Romero Miret (Personal key for password-store) <jose.romero@aladroc.io>

## Public key
gpg --export -a <<KeyID>> > jose-public.asc
### Example
gpg --export -a ABCD1234EFGH5678 > jose-public.asc

## Private key
gpg --export-secret-keys -a <<KEYID>> > jose-private.asc
gpg --export-secret-subkeys -a <<KEYID>> > jose-secret-subkeys.asc #(Optinal If you have)
### Example
gpg --export-secret-subkeys -a ABCD1234EFGH5678 > jose-private.asc
gpg --export-secret-subkeys -a ABCD1234EFGH5678 > jose-secret-subkeys.asc #(Optinal If you have)
```

```bash
# Import keys
gpg --import jose-public.asc
gpg --import jose-private.asc
gpg --import jose-secret-subkeys.asc #(Optinal If you have)
```

```bash
# Trusting the keys (optional but recommended)
gpg --edit-key <<KEYID>>
gpg>trust
gpg>5    #(ultimate)
gpg>quit

## Example
gpg --edit-key ABCD1234EFGH5678
gpg>trust
gpg>5    #(ultimate)
gpg>quit
```

Notes:

## The main (primary) key

- Called primary key or sec (secret) in gpg --list-secret-keys.

- It’s your identity:

      - Holds your name, email, and comment.

      - Signs other keys and subkeys to prove they belong to you.

- By default it can:

      - Sign (S)

      - Certify (C) — create and sign new subkeys

      - Optionally Encrypt or Authenticate (but best practice is not to use it for everyday encryption).

Think of it as the master identity + signing authority.

## Subkeys

- Shown as ssb (secret subkey) in the list.

- Each subkey is bound (certified) by the primary key.

- Common uses:

    - E → encryption

    - S → signing

    - A → authentication (SSH/Git)

You can have multiple subkeys for different tasks and rotate/expire them independently.

Why?
Because the primary key can stay offline (e.g., on a USB in a safe),
while subkeys do the everyday work.

If a subkey is compromised, you can revoke and replace that subkey using the offline primary key,
without changing your email or re-sharing your main key.


```bash
# Example
sec   rsa4096/ABCD1234EFGH5678 2025-09-17 [SC]
      uid   [ultimate] Jose Rafael Romero Miret <jose.romero@aladroc.io>
ssb   rsa4096/1111AAAA2222BBBB 2025-09-17 [E]
ssb   rsa4096/3333CCCC4444DDDD 2025-09-17 [S]
```


## || Sidekick: Web of Trust
TODO

# | Passwordstore

## || Initializing the backend

# Initializing
```bash
pass init <<KEYID>> # one key
pass init "<<KEYID1>> <<KEYID2>> <<KEYID3>>"
## Example
pass init ABCD1234EFGH5678
pass init "ABCD1234EFGH5678 1111AAAA2222BBBB 333CCCC4444DDDD"
```

## || Insert Secret

```bash
# Insert
pass insert <<folder>>/..<<folder>>../<<keyName>>
## Note: you can build a tree structure using nested folders
## Note: pass insert -m allows you to paste multiple lines secret

## Example
pass insert email/github

Enter password for email/github:
Retype password for email/github:
```

## || Generate Secret

```bash
# Generate Password
pass generate <<folder>>/..<<folder>>../<<keyName>> <<Password Length>>
## Note: pass generate -c copies password to clipboard

## Example
pass generate email/github 24
The generated password for email/github is:
d@j#X9fR!2lVmE4kT8pZs3wq
```

## || List Secrets

```bash
pass
pass ls
pass ls --flat

## Example
## For those inserted secrets:
email/github
email/gmail
servers/proxmox/root
servers/proxmox/user
wifi/home
api/tokens/slack

## Will show something like this:
Password Store
├── api
│   └── tokens
│       └── slack
├── email
│   ├── github
│   └── gmail
├── servers
│   └── proxmox
│       ├── root
│       └── user
└── wifi
    └── home

## Example --flat option
api/tokens/slack
email/github
email/gmail
servers/proxmox/root
servers/proxmox/user
wifi/home

## Notes :: File structure
tree ~/.password-store
~/.password-store/
├── api
│   └── tokens
│       └── slack.gpg
├── email
│   ├── github.gpg
│   └── gmail.gpg
├── servers
│   └── proxmox
│       ├── root.gpg
│       └── user.gpg
└── wifi
    └── home.gpg
```
# || Show Secret

```bash
# Show Secret
pass show <<folder>>/..<<folder>>../<<keyName>>
```

## || Check keys encrypted a secret

```bash
# Check keys encrypted a secret
gpg --list-packets ~/.password-store/<<folder>>/..<<folder>>../<<keyName>>

## Example
gpg --list-packets ~/.password-store/email/github.gpg

# off=0 ctb=84 tag=1 hlen=3 plen=268
:pubkey enc packet: version 3, algo 1, keyid 1234ABCD5678EF90
        data: ...
# off=271 ctb=84 tag=1 hlen=3 plen=268
:pubkey enc packet: version 3, algo 1, keyid 9ABCDEF01234ABCD
        data: ...
# off=542 ctb=d4 tag=18 hlen=2 plen=...
:encrypted data packet:
        length: ...
        mdc_method: 2
```

## || Backup on GIT

```bash
# The vanilla way
cd ~/.password-store
git init

git remote add origin git@github.com:YOURUSER/password-store.git
git add .
git commit -m "Initial password-store commit"
git push -u origin master

# The pass command way
pass git init
pass git remote add origin git@github.com:YOURUSER/password-store.git
pass git push -u origin master

pass git add -A
pass git commit -m "Add new secrets"
pass git push

# Automatizing a bit
pass insert -m my/new/secret
pass git add -A && pass git commit -m "Added my/new/secret" && pass git push
## building an alias...
alias pass-commit="pass git add -A && pass git commit -m 'Update' && pass git push"
```

## || Restore on another machine from GIT

```bash
git clone git@github.com:YOURUSER/password-store.git ~/.password-store
gpg --import jose-private.asc
pass show email/github
```

# | Links

- https://www.passwordstore.org/

- https://git.zx2c4.com/password-store/

- https://gnupg.org/ 

- https://git-scm.com/ 

- https://github.com/aladrocMatiner/Presentations/tree/main/passwordstore


# | Other

- https://github.com/FiloSottile/passage



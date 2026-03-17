# 23. SSH and SCP

## SSH (Secure Shell)

SSH provides encrypted remote login and command execution. Uses public-key or password authentication.

Connect:

```bash
ssh username@hostname
ssh -p 561 username@hostname   # Non-default port
ssh -i ~/.ssh/mykey username@hostname
```

Options: **-l** user, **-i** identity (private key), **-F** config file, **-p** port, **-v/-vv/-vvv** verbose.

## SSH keys

Generate key pair:

```bash
ssh-keygen -t rsa -b 2048
# or
ssh-keygen -t ed25519
```

Default: **~/.ssh/id_rsa** (private), **~/.ssh/id_rsa.pub** (public). Copy public key to server: **`ssh-copy-id user@host`** or append **id_rsa.pub** to **~/.ssh/authorized_keys** on the server.

## SCP (secure copy)

Copy files over SSH:

```bash
scp file user@host:path          # To remote
scp user@host:path file         # From remote
scp -r dir user@host:path        # Recursive (directory)
```

## SFTP

Interactive file transfer over SSH: **`sftp user@host`**. Commands: **get**, **put**, **ls**, **cd**, etc.

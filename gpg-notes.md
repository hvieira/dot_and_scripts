# GPG setup

## Accessing the yubikey

Check gpg daemon is running

`gpg-agent status`

Check it can read the yubikey

`gpg --card-status`

If it cannot read the yubikey, then it might require scdaemon config as per https://support.yubico.com/hc/en-us/articles/360013714479-Troubleshooting-Issues-with-GPG 

## Setting up the yubikey and keys

Change pin & admin pin of the yubikey if necessary

`gpg --change-pin`

NOTE: Remember the pins!

Create master key

`gpg --full-generate-key`

NOTE: Remember the private key password of course!

Export key (pub and private) with ASCII armor and keep it safe

```
gpg --export -a <key ID> > master.key.pub
gpg --export-secret-key -a <key ID> > master.key.private
```

Create the sub keys (signature, encryption and authentication) from the master key on the card

`gpg --edit-key <key ID>`
and then
`addcardkey`

and follow the promtps

At this point the subkeys, each with single responsibility (sign, encrypt, authenticate) are in the yubikey

`gpg --card-status`

### Publish the public key somewhere where you can fetch it from later on
Github can be used for this purpose

## SSH auth

based on https://hedberg.io/yubikey-for-ssh/

Add this to `.gnupg/gpg-agent.conf`

```
enable-ssh-support
default-cache-ttl 60
max-cache-ttl 120
pinentry-program /usr/bin/pinentry-gnome3
```

Add this to .zshrc

```
export "SSH_AUTH_SOCK=${HOME}/.gnupg/S.gpg-agent.ssh"
gpgconf --launch gpg-agent
```

If restarting the shell/reload zshrc is not necessary, then

```
ssh-add -L
```

Should list the public key

I had to remove id_rsa files/pointers under .ssh/ to get it working

## Signing Data
Verify that the public key is present. If `gpg --card-status` and `gpg --list-secret-keys --keyid-format=long` are working (as in the keys are listed), then
the public key is present. If not, then it needs to be imported by `gpg --card-edit` -> `fetch` command - it'll pick up the public key from the URL configured in the yubikey 

### Github commit signing
https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key
# Yubikey macOS Setup

*You bought a yubikey - now what?*

The goal is to outline the steps to configure your yubikey in a sane method
and to use it to maximize your security.

This guide is for users who are comfortable with the command line and various
technical jargon.

This is highly opinionated on how you should and should not use your yubikey
but is organized well enough that you should be able to modify if you have a
need.

The instructions have been tested on macOS 10.12 (Sierra) with a Yubikey 4.

To perform these instructions, the Yubikey should be plugged into your computer's USB port.

## Use for Two Factor Authentication / U2F Setup

### GitHub

1. Go to your [GitHub Security Settings](https://github.com/settings/security)
2. Turn on `Two-factor Authentication` if it's not already enabled. You will
   need to set up either an SMS or TOTP (Google Authenticator) if it's not.
3. Under `Security keys, choose `Register new device`
4. Type in a name: `yourname-yubikey-nano4` or something else that will help
   you remember the key
5. Click `Add`
6. Follow the instructions on screen - you'll probably need to tap the yubikey
   for it to register.

Yubico has more [detailed instructions](https://www.yubico.com/support/knowledge-base/categories/articles/use-yubikey-github/).

### Google

Similar instructions. 

1. Go to your [Google Sign-in & Security page](https://myaccount.google.com/security)
2. Click `Two-step verification` and you may be prompted for your password.
3. Click `Add Security Key` and follow the on-screen instructions. You may
   need to tap or touch your yubikey.

Yubico has a [video](https://www.yubico.com/why-yubico/for-individuals/gmail-for-individuals/)

## Setup GPG Key

Before you being, you'll need to install [GPGTools GPG Suite](https://gpgtools.org/). As you do this, here are a few notes about it:
   
- Stash the DMG somewhere if you ever need to uninstall it, as an uninstaller is in the DMG package
- After installation completes, you don't need to do anything via the GPG Keychain GUI
- Benefits (versus CLI-only apps): Launches gpg-agent automatically, has a GUI for management and PIN entry, doesn't require Yubikey modes to be changed during GPG setup, still installs the CLI apps

Now, generate the keys. Start a Terminal session, then issue the following commands and options:

```bash
> gpg2 --card-edit

[truncated...]

gpg/card> admin
Admin commands are allowed

gpg/card> generate
Make off-card backup of encryption key? (Y/n) n

[PIN Entry pops up, enter 123456, which is the default pin]

What keysize do you want for the Signature key? (2048) 4096 [Yubikey NEO max is 2048]
[PIN Entry pops up, enter 12345678, which is the default admin pin]
The card will now be re-configured to generate a key of 4096 bits

What keysize do you want for the Encryption key? (2048) 4096 [Yubikey NEO max is 2048]
[PIN Entry pops up, enter 12345678, which is the default admin pin]
The card will now be re-configured to generate a key of 4096 bits

What keysize do you want for the Authentication key? (2048) 4096 [Yubikey NEO max is 2048]
[PIN Entry pops up, enter 12345678, which is the default admin pin]
The card will now be re-configured to generate a key of 4096 bits

Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
Key does not expire at all
Is this correct? (y/N) Y

GnuPG needs to construct a user ID to identify your key.

Real name: <YOUR_NAME_HERE>
Email address: <YOUR_EMAIL_HERE>
Comment:
You selected this USER-ID:
    "YOUR_NAME_HERE <YOUR_EMAIL_HERE>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
```

The Yubikey will flash as it's creating the key. Mine took about 5 minutes.
When complete, it will say something like

```
gpg: key 00000000 marked as ultimately trusted
public and secret key created and signed.

[truncated...]
```

You should change your PIN and Admin PIN. You can do that here with `passwd` command
at the `gpg --card-edit` `gpg/card>` prompt while in admin mode (i.e. where we left off from the prior step):

```
gpg/card> passwd

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 1
[Enter 123456]
[Enter your new PIN]
[Enter your new PIN again]

PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? 3
[Enter 12345678]
[Enter your new Admin PIN]
[Enter your new Admin PIN again]

PIN changed.

1 - change PIN
2 - unblock PIN
3 - change Admin PIN
4 - set the Reset Code
Q - quit

Your selection? Q
```

### (Optional) Other GPG Setup

While you're here:
```
gpg/card> name
Cardholder's surname: [Your last name]
Cardholder's given name: [Your first name]
[Enter your admin PIN]

gpg/card> sex
Sex ((M)ale, (F)emale or space): [Your gender]

gpg/card> lang
Language preferences: [Your two letter language code, example: en)
```

You can see the configuration by typing `list` on the `gpg/card>` prompt.

https://www.yubico.com/support/knowledge-base/categories/articles/use-yubikey-openpgp/


## Yubikey for SSH logins

Should be able to generate a SSH key from the PGP key

## macOS Login / PIV Login

1. Follow [Yubico's PIV pairing instructions](https://www.yubico.com/support/knowledge-base/categories/articles/how-to-use-your-yubikey-with-macos-sierra/)
1. Follow [Yubico's Login Guide](https://www.yubico.com/wp-content/uploads/2016/02/Yubico_YubiKeyMacOSXLogin_en.pdf) with the suggested sections:
   - Configuring YubiKeys with the YubiKey Personalization Tool
   - Installing Yubico Pluggable Authentication Module (PAM) 
   - Configuring Yubico Pluggable Authentication Module (PAM) _(including all subsections in the chapter)_

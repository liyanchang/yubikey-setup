# Yubikey setup

*You bought a yubikey - now what?*

The goal is to outline the steps to configure your yubikey in a sane method
and to use it to maximize your security.

This guide is for users who are comfortable with the command line and various
technical jargon.

This is highly opinionated on how you should and should not use your yubikey
but is organized well enough that you should be able to modify if you have a
need.

The instructions are only for OSX 10.12.

## Insert the yubikey into your computer

Plug it into a USB port.

## Install some software

```bash
# For OSX
> brew install python3 swig ykpers libu2f-host libusb
> pip install yubikey-manager
```

## Turn off OTP - AKA the random letters when you accidentally touch it

This will turn off One-Time-Password. Most users will not find OTP useful and
will be confused by the random letters that will appear when they accidentally
touch the yubikey.

_Exception:_ If you use LastPass - LastPass does use OTP for their two factor
and not U2F or TOTP (Google Authenticator) so you will want to skip this step
if you are a LastPass user.

```bash
> ykman mode
Current connection mode is: OTP+U2F+CCID
Supported connections are: OTP, U2F, CCID
> ykman mode "U2F+CCID"
Set mode of YubiKey to U2F+CCID? [y/N]: Y
Mode set! You must remove and re-insert your YubiKey for this change to take effect.
```

Remove and re-insert the yubikey.

```bash
> ykman mode
Current connection mode is: U2F+CCID
Supported connections are: OTP, U2F, CCID
```

## Set up your U2F

### Github

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

### Gmail

Similar instructions. TODO: Specific details
1. Go to your [Google Sign-in & Security page](https://myaccount.google.com/security)
2. Click `Two-step verification` and you may be prompted for your password.
3. Click `Add Security Key` and follow the on-screen instructions. You may
   need to tap or touch your yubikey.

Yubico has a [video](https://www.yubico.com/why-yubico/for-individuals/gmail-for-individuals/)

### Dropbox

1. Go to your [Dropbox Security Settings](https://www.dropbox.com/account/#security)
2. Under `Security keys`, click `Add`
3. Follow the on-screen instructions. You'll probably be prompted for your
   password and touch the yubikey to complete registration.

Yubico has a [video and more detailed instructions](https://www.yubico.com/why-yubico/for-individuals/dropbox-for-individuals/)

### Dashlane, Salesforce, Bitbucket, Gitlab, GOV.UK Verify

Yubico has [instructions](https://www.yubico.com/about/background/fido/)

## Yubikey for GPG keysigning

1. Install GPG2 if you haven't already

```bash
> brew install gnupg gnupg2
```

2. Configure your GPG conf at `~/.gnupg/gpg.conf`

Suggested hardened [configuration](https://github.com/ioerror/duraconf/blob/master/configs/gnupg/gpg.conf)

Here's the minimum that makes sense:
```
use-agent
personal-cipher-preferences AES256 AES192 AES CAST5
personal-digest-preferences SHA512 SHA384 SHA256 SHA224
cert-digest-algo SHA512
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed
```

3. Configure SC daemon?

```
~/.gnupg/scdaemon.conf and add "pcsc-driver /System/Library/Frameworks/PCSC.framework/PCSC"
```

3. Generate Keys

TODO: Should be able to generate keys directly on the yubikey

https://www.yubico.com/support/knowledge-base/categories/articles/use-yubikey-openpgp/

```
> gpg2 --card-edit

gpg: OpenPGP card not available: Not supported
```


## Yubikey for SSH logins

Should be able to generate a SSH key from the PGP key

## Yubikey for PIV


## Yubikey for OSX login

Should be possible once I have a PIV cert on it.

TODO: Look into `yubiswitch` to see how it will lock the screen when the
yubikey is removed.


## Set up your yubikey at TOTP - a Google Authenticator replacement

- Go to [Dropbox Security Settings](https://www.dropbox.com/account/#security)
- Choose to enable two factor.
- Select `Use a mobile app`
- Click `enter your secret key manually` to display a 26 long base32 key.
- Copy the key below - don't forget to remove the spaces

```bash
% The `-t` will require a touch inorder for codes to be generated.
% This prevent malware from generating codes without your knowledge.
> ykman oath add -t <SERVICE_NAME> <32 DIGIT BASE32 KEY NO SPACES>
> ykman oath code <SERVICE_NAME>
Touch your YubiKey...
SERVICE_NAME 693720
```

- Repeat for other providers.

    The steps will be similar - the difference will be how to get the manual
    key instead of the QR code. When the QR code is displayed, there will
    often be a link:
        - Dropbox: `Enter your secret key manually`
        - Gmail: `Can't scan it?`
        - Github `Enter this text code`

Note: There is a way to configure a long press to enter the TOTP password, but
given that I expect you to have many TOTP codes, this seems pretty useless.



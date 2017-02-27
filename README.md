# Yubikey Setup

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

<!--
3. Configure SC Daemon?

Found this on forums, but did not end up solving or hurting but unclear if it
was necessary?

~/.gnupg/scdaemon.conf
```
disable-application openpgp nks
pcsc-driver /System/Library/Frameworks/PCSC.framework/PCSC
```

-->

3. Temporarily disable U2F

Having U2F enabled will result in `sharing violations` that results in `gpg2`
not being able to access the yubikey.

You will be able to renable U2F and it won't break any sites you already set
up with U2F.

```bash
> ykman mode
Current connection mode is: U2F+CCID
Supported connections are: OTP, U2F, CCID
> ykman mode "CCID"
Set mode of YubiKey to CCID? [y/N]: Y
Mode set! You must remove and re-insert your YubiKey for this change to take effect.
> ykman mode
Current connection mode is: CCID
Supported connections are: OTP, U2F, CCID
```

3. Generate Keys

_Note:_ If you have a Yubikey 4, you should use 4096 as your key length. NEO
owners should use 2048 as that is the maximum supported.

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

You should change your PIN and Admin PIN. You can do that here with `passwd`
at the `gpg/card>` prompt:

```

> gpg --card-edit

...truncated...

gpg/card> admin
Admin commands are allowed

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


## Notes

### Sharing Violation, Card Read Error


```bash
% Fix by toggling off U2F mode
> gpg2 --card-edit

gpg: OpenPGP card not available: Not supported
> gpg --card-status

gpg: OpenPGP card not available: general error
% Other commands
> pcsctest
> opensc-tool
```







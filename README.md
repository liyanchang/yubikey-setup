# YubiKey macOS Setup

*You bought a YubiKey - now what?*

The goal is to outline the steps to configure your YubiKey in a sane method
and to use it to maximize your security.

This guide is for users who are comfortable with the command line and various
technical jargon.

This is highly opinionated on how you should and should not use your YubiKey
but is organized well enough that you should be able to modify if you have a
need.

The instructions have been tested on macOS 10.12 (Sierra) with a YubiKey 4 and
YubiKey Neo. While there are sections that are OS independent, most of the
tricky bits are macOS specific.

To perform these instructions, the YubiKey should be plugged into your
computer's USB port.

## Install some software

```bash
# For OSX
> brew install python3 swig ykpers libu2f-host libusb
> pip install yubikey-manager
```

## Turn off OTP - AKA the random letters when you accidentally touch it

This will turn off One-Time-Password. Most users will not find OTP useful and
will be confused by the random letters that will appear when they accidentally
touch the YubiKey.

_Exception:_ LastPass supports OTP and TOTP as a two factor method and does not
support U2F. If you plan on using OTP with LastPass, you will want to skip this
step. If you choose TOTP (README.md#set-up-your-yubikey-at-totp---a-google-authenticator-replacement)
and don't set up OTP, I'd suggest you disable OTP.

```bash
> ykman mode
Current connection mode is: OTP+U2F+CCID
Supported connections are: OTP, U2F, CCID
> ykman mode "U2F+CCID"
Set mode of YubiKey to U2F+CCID? [y/N]: Y
Mode set! You must remove and re-insert your YubiKey for this change to take
effect.
```

Remove and re-insert the YubiKey.

```bash
> ykman mode
Current connection mode is: U2F+CCID
Supported connections are: OTP, U2F, CCID
```

## Use for Two Factor Authentication / U2F Setup

U2F is the recommended two factor method. It is phishing resistant unlike
TOTP/Google Authenticator. It is much harder to compromise than SMS/Voice call
methods.

The instructions below are specific to provider, but they are all similar
enough.

### GitHub

1. Go to your [GitHub Security Settings](https://github.com/settings/security)
2. Turn on `Two-factor Authentication` if it's not already enabled. You will
   need to set up either an SMS or TOTP (Google Authenticator) if it's not.
3. Under `Security keys, choose `Register new device`
4. Type in a name: `yourname-yubikey-nano4` or something else that will help
   you remember the key
5. Click `Add`
6. Follow the instructions on screen - you'll probably need to tap the YubiKey
   for it to register.

Yubico has more [detailed instructions](https://www.yubico.com/support/knowledge-base/categories/articles/use-yubikey-github/).

### Google

1. Go to your [Google Sign-in & Security page](https://myaccount.google.com/security)
2. Click `Two-step verification` and you may be prompted for your password.
3. Click `Add Security Key` and follow the on-screen instructions. You may
   need to tap or touch your YubiKey.

Yubico has a [video](https://www.yubico.com/why-yubico/for-individuals/gmail-for-individuals/)

### Dropbox

1. Go to your [Dropbox Security Settings](https://www.dropbox.com/account/#security)
2. Under `Security keys`, click `Add`
3. Follow the on-screen instructions. You'll probably be prompted for your
   password and touch the YubiKey to complete registration.

Yubico has a [video and more detailed instructions](https://www.yubico.com/why-yubico/for-individuals/dropbox-for-individuals/)

### Dashlane, Salesforce, Bitbucket, Gitlab, GOV.UK Verify

Yubico has [instructions](https://www.yubico.com/about/background/fido/)

## YubiKey for GPG keysigning

1. Install GPG2 if you haven't already

   ```bash
   > brew install gnupg gnupg2
   ```

2. Configure your GPG conf at `~/.gnupg/gpg.conf`

   Suggested hardened [configuration](https://github.com/ioerror/duraconf/blob/master/configs/gnupg/gpg.conf).
   Here's the minimum that makes sense:
   
   ```
   use-agent
   personal-cipher-preferences AES256 AES192 AES CAST5
   personal-digest-preferences SHA512 SHA384 SHA256 SHA224
   cert-digest-algo SHA512
   default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed
   ```

3. Temporarily disable U2F

   NOTE: This seemed not to be an issue with my most recent yubikey.

   Having U2F enabled will result in `sharing violations` that results in `gpg2`
   not being able to access the YubiKey. You will be able to renable U2F and it
   won't break any sites you already set up with U2F.

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

   _Note:_ If you have a YubiKey 4, you should use 4096 as your key length. NEO
   owners should use 2048 as that is the maximum supported.

   ```bash
   > gpg2 --card-edit

   [truncated...]

   gpg/card> admin
   Admin commands are allowed

   gpg/card> generate
   Make off-card backup of encryption key? (Y/n) n

   [PIN Entry pops up, enter 123456, which is the default pin]

   What keysize do you want for the Signature key? (2048) 4096 [YubiKey NEO max is 2048]
   [PIN Entry pops up, enter 12345678, which is the default admin pin]
   The card will now be re-configured to generate a key of 4096 bits

   What keysize do you want for the Encryption key? (2048) 4096 [YubiKey NEO max is 2048]
   [PIN Entry pops up, enter 12345678, which is the default admin pin]
   The card will now be re-configured to generate a key of 4096 bits

   What keysize do you want for the Authentication key? (2048) 4096 [YubiKey NEO max is 2048]
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

   The YubiKey will flash as it's creating the key. Mine took about 5 minutes.
   When complete, it will say something like

   ```bash
   gpg: key 00000000 marked as ultimately trusted
   public and secret key created and signed.

   [truncated...]
   ```

   You should change your PIN and Admin PIN. You can do that here with `passwd`
   at the `gpg/card>` prompt:

   ```bash
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

4. (Optional) Other GPG Setup

   While you're here:
   ```bash
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

## YubiKey for SSH logins

You can generate an SSH key from your PGP key and use it for SSH logins.

1. Identify your authentication key.

    ```bash
    > gpg2 --card-status | grep Authentication
    Authentication key: AAAA BBBB CCCC DDDD EEEE  FFFF GGGG HHHH IIII JJJJ
    ```

2. Generate the SSH key

    Take the last 16 digits and pass them to `gpg --export-ssh-key`.

    ```bash
    > gpg --export-ssh-key GGGGHHHHIIIIJJJJ
    ssh-rsa AAAAG4AFq6wm1eCcRclsVOYcJf8y
    ...
    ...
    G46wm1eCcRclsVOYcJf8yPr1b+kzUpGQLw==
    ```

3. Copy the public key and add it `~/.ssh/authorized_keys` the machine you want to SSH into
4. Attempt to login to the machine via SSH

## YubiKey for PIV

Yubico has a GUI tool called yubikey-piv-manager that can help set up your
YubiKey for PIV. While I have a preference for command-line tools, the GUI
sets everything up in one click and saves significant hassle.

1. Install YubiKey PIV Manager

    ```bash
    > brew cask install Caskroom/cask/yubikey-piv-manager
    ```

2. Navigate to `Setup for macOS` and click `yes`.

    Choose a 6-8 digit number. Don't use non-numeric characters. Yubikey will
    be fine, but macOS will not.

    The default settings are fine.

3. Remove and re-insert your YubiKey.

4. Pair with macOS

    When you insert your Yubikey, a prompt should appear asking if you would
    like to pair your smartcard. Click `Pair`. It will ask for your username
    and password as well as the pin you just created. It may also ask you for
    your keychain password - it's the same as your account password.

5. Login with your YubiKey and PIN

    The next time you login with your YubiKey inserted, macOS should prompt
    you for your PIN and not a password.


<!-- Notes from when I was trying to set it up by hand

```
> yubico-piv-tool -s 9a -A ECCP256 -a generate
-----BEGIN PUBLIC KEY-----
...
-----END PUBLIC KEY-----
Successfully generated a new private key.

```
yubico-piv-tool -s 9a -S '/CN=nano4/OU=yubikey/O=ldchang.com/' -P 123456 -a verify -a request

Successfully verified PIN.
Please paste the public key...
-----BEGIN PUBLIC KEY-----
...
-----END PUBLIC KEY-----
-----BEGIN CERTIFICATE REQUEST-----
...
-----END CERTIFICATE REQUEST-----
Successfully generated a certificate request.
```

```
yubico-piv-tool -s 9a -S '/CN=nano4/OU=yubikey/O=ldchang.com/' -P 123456 -a verify -a selfsign

Successfully verified PIN.
Please paste the public key...
-----BEGIN PUBLIC KEY-----
...
-----END PUBLIC KEY-----
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
Successfully generated a new self signed certificate.
```

```
> yubico-piv-tool -s 9a -a import-certificate
Please paste the certificate...
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
Successfully imported a new certificate.
```

-->

## YubiKey for OSX login

Once you have PIV credentials on your YubiKey, macOS should prompt you if you
want to use it for login.

TODO: Look into `yubiswitch` to see how it will lock the screen when the
YubiKey is removed.

## Set up your YubiKey at TOTP - a Google Authenticator replacement

You can have your YubiKey generate TOTP codes, just like Google Authenticator
or Authy.

If you use it as a replacement for Google Authenticator, remember that you'll
be unable to get the code if you don't have your YubiKey with you and a
computer with `ykman` or `Yubico Authenticator` installed or an Android phone
with `Yubico Authenticator` installed.

You can also use both a phone based app and a YubiKey, knowing that either
device will generate the same codes and will be able to access your account.

1. Go to [Dropbox Security Settings](https://www.dropbox.com/account/#security)
2. Choose to enable two factor.
3. Select `Use a mobile app`
4. Click `enter your secret key manually` to display a 26 digit long base32 key.
   Note: The link text will differ by provider. The length of the base32 key may
   also differ.
5. (Optional) If you also want to use your phone, you can scan the barcode or
   type in the code to `Google Authenticator`.
6. Copy the key below - don't forget to remove the spaces

    ```bash
    % The `-t` will require a touch inorder for codes to be generated.
    % This prevent malware from generating codes without your knowledge.
    % YubiKey Neo's do not support this feature. Just remove the `-t` flag.
    > ykman oath add -t <SERVICE_NAME> <32 DIGIT BASE32 KEY NO SPACES>
    > ykman oath code <SERVICE_NAME>
    Touch your YubiKey...
    SERVICE_NAME 693720
    ```

7. Repeat for other providers.

    The steps will be similar - the difference will be how to get the manual
    key instead of the QR code. When the QR code is displayed, there will
    often be a link to get the code. Here are some examples:

    - Dropbox: `Enter your secret key manually`
    - Gmail: `Can't scan it?`
    - Github `Enter this text code`


# Secure Debian with TOTP (SSH) login

To start we need to install the following package `libpam-google-authenticator` with the command.

```bash
apt install libpam-google-authenticator 
```

## Setup libpam-google-authenticator 

To generate the secret needed to generate the Time-based One Time Password (TOTP) run the following command:

```bash
google-authenticator
```

Then you will be greeted with the question **"Do you want authentication tokens to be time-based" type "y"** and press ENTER otherwise it will generate an HOTP key (MAC-based One Time Password) that we won't cover in this guide.

After you pressed ENTER an big QR code will pop-up in the console scan this with your prefferd TOTP app (e.g. ENTE auth, Aegis, Google authenticator etc) on your phone give it an memorable name and save it. Type the 6 numbers in to the console and press ENTER.

After this 5 recovery codes will show save these in an secure location.

You will get 3 other questions you could type "y" and press ENTER on all of them or read them and choose for your self.

## Setting up PAM to use the libpam-google-authenticator extention

> [!WARNING]
> If you modify the configs bellow when you don't have acces to your TOTP device or recovery codes is not recommended ensure you have an other way back in (e.g. first changing `/etc/pam.d/sshd` and testing before changing `/etc/pam.d/login` to ensure you don't lock your self out!)

To make sure PAM uses the TOTP codes we need to modify the following configs.

`/etc/pam.d/login` This handles the general login.

`/etc/pam.d/sshd` This handles the SSH login.

### Default Login
Open the following config `/etc/pam.d/login` and add the following lines to the bottom of the config
```bash
auth required pam_google_authenticator.so nullok
auth required pam_permit.so
```

The bottom of the config will now look like this

```bash
# Standard Un*x account and session
@include common-account
@include common-session
@include common-password
auth required pam_google_authenticator.so nullok
auth required pam_permit.so
```

You can logout and login to see your TOTP in action!

### SSH login
Open the following config `/etc/pam.d/sshd` and add the following lines to the bottom of the config
```bash
auth required pam_google_authenticator.so nullok
auth required pam_permit.so
```

The bottom of the config will now look like this

```bash
# Standard Un*x password updating.
@include common-password
auth required pam_google_authenticator.so nullok
auth required pam_permit.so
```

Now we need to allow `KbdInteractiveAuthentication` in the sshd config located at `/etc/ssh/sshd_config`

Uncomment `KbdInteractiveAuthentication` and set it to 'yes'

Run:
```bash
sudo systemctl restart sshd.service
```

#### Public key + TOTP

> [!WARNING]
> If you enable this you will not be able to login without using a public key!

To use Public key auth with TOTP make the following change.

Open the `/etc/ssh/sshd_config` and go to the bottom of the file  and add

```bash
AuthenticationMethods publickey,password publickey,keyboard-interactive
```

#### Forcing "KeyboardInterActiveAuthentication"

> [!NOTE]
> Logging in will then require Password + TOTP and Public key authentication will no longer work.

To force every user to use `KbdInteractiveAuthentication` uncomment:

`PubkeyAuthentication` and `PasswordAuthentication` and set them to 'no'

This way users will be forced to use keyboard interactive authentication.

Yubikey for LUKS, initcpio / Arch Linux style
================

How it works
----------------
initcpio hooks to allow you to use a yubikey in challenge-response HMAC
mode to do 2-factor authentication for LUKS.

password you type in -> yubikey hmacs it with its secret key -> yubikey
returns the password for LUKS

This has the property that someone with access to your yubikey cannot
trivially gain useful information (this is not the case if you config
it to output a static passphrase which you concatenate to a password
you type in)

Naturally, if someone knows your password and later has access to your
yubikey, they can can get the LUKS password.

Data loss warning
----------------

WARNING: if your yubikey gets lost/stolen/eaten/fails, there is no way
to recover the keyfile it generates when fed with the correct password.
Make sure to have a backup keyslot in your LUKS header (or to have
saved *somewhere* a LUKS header for that volume with the same master key
with a keyslot for which you know the passphrase).

Install
----------------

To set up the yubikey, you do:
ykpersonalize -v -1 -ochal-resp -ochal-hmac -ohmac-lt64 -ochal-btn-trig -oserial-api-visible

The -ochal-btn-trig option makes it such that instead of immediately
returning data in response to a challenge, it makes the yubikey's LED
blink and only returns data when you touch the yubikey's button.

To add the appropriate keyslot in the LUKS volume, do:
read -s PW
ykchalresp "$PW" | tr -d '\n' > mykeyfile
cryptsetup luksAddKey /dev/whatever mykeyfile
rm mykeyfile

Put the stuff in hooks/ into /etc/initcpio/hooks/ and the stuff in
install/ into /etc/initcpio/install/ and edit your /etc/mkinitcpio.conf
appropriately. Replace "/home/thz/bin/ykchalresp" in install/yubiluks
with the correct path to ykchalresp.

You need to put "yubiluks" *before* "encrypt" in the HOOKS line in your
/etc/mkinitcpio.conf file in order for the yubikey things to happen
before cryptsetup is invoked.


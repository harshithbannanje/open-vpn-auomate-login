# Automating Open VPN Login [Ubuntu]

Are you sick of entering OTP’s every time you login to open-vpn ?

Okay, here is a script that will make your life easier

    #!/usr/bin/expect -f

    set otp [exec oathtool --base32 --totp <totp-secret>]

    spawn sudo openvpn --config client.ovpn --up /etc/openvpn/update-resolv-conf --down /etc/openvpn/update-resolv-conf --script-security 2 --auth-retry interact

    expect {\[sudo\] password for <user>:}
    send -- "<sudo-password>\r"

    expect "Enter Auth Username:"
    send -- "<open-vpn-username>\r"

    expect "Enter Auth Password:"
    send -- "<open-vpn-password>\r"

    expect "CHALLENGE: Enter Authenticator Code"
    send -- "$otp\r"

    interact
    close

Before you proceed, you will need the open-vpn totp secret.

Open your Google Authenticator, click on transfer accounts then export accounts.

![](https://cdn-images-1.medium.com/max/2400/0*V390gC3kA1c_rcP8.jpg)

Click on Scan QR Code, now take a picture of the QR code (Screenshots won’t be allowed due to app security policy).

Once you have the QR Code, scan it using any app, you will see something like below

    otpauth-migration://offline?data=<some-data>

To decode the secret you will need a small library.

First install [Go](https://golang.org/dl/) version 1.6 or above

Run the following commands

    go env -w GO111MODULE=off
    go get github.com/dim13/otpauth

To extract your secret, run this command with your QR code data

    ~/go/bin/otpauth -link "otpauth-migration://offline?data=<some-data>" 

This will output your decoded data which includes the secret

    otpauth://totp/Example:alice@google.com?issuer=Example&secret=JBSWY3DPEHPK3PXP

Before you run the script make sure you have the following dependencies installed

    sudo apt-get install -y expect
    sudo apt-get install -y oathtool

Also make sure you have replaced these variables in the script

    <totp-secret>        : The secret extracted from the QR Code
    <user>               : The current user i.e whoami
    <sudo-password>      : Your sudo passowrd
    <open-vpn-username>  : Open VPN Username
    <open-vpn-password>  : Open VPN Password

I hope you will find this script useful. Please do share this article.

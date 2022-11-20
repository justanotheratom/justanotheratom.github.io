---
title: 'Running a Server from Dev Machine for development/testing purposes'
---

I am trying to develop a Telegram Bot in Python on my dev machine. And I want the webhook call to route to the Python Server running on my dev machine, for testing.

- Find my public IP Address
    
    Invoke-RestMethod -Uri https://ipinfo.io | Select -ExpandProperty ip

    50.35.66.240

- Find my laptop's private IP Address

    get-netipaddress | ? { $_.AddressFamily -eq 'IPv4' -and $_.PrefixOrigin -eq 'Dhcp' } | Select -ExpandProperty IPAddress

    192.168.86.25

- Download OpenSsl precompiled binaries for Windows

    https://wiki.openssl.org/index.php/Binaries

- Generate Self-Signed Certificate

    "[ req ]\`ndistinguished_name = req_dn\`n\`n[ req_dn ]" | Set-Content .\openssl.cnf

    .\openssl req -newkey rsa:2048 -sha256 -nodes -keyout tellthenaskbotprivate.key -x509 -days 365 -out tellthenaskbotpublic.pem -subj "/C=US/ST=Washington/L=Seattle/O=FUNGEE LLC/CN=tellthenaskbot.fungee.llc" -config (dir .\openssl.cnf | select -expandproperty fullname)

- Go to Google Domain Manager and add an A record for tellthenask.fungee.llc pointing to my public IP Address.

- Go to Google Home Wifi Advanced Settings Port Management, and add a port forwarding rule to forward https traffic on public IP address to laptop's private IP address.

- Run the Server with https:
 
    flask run --cert=.\tellthenaskbotpublic.pem --key=.\tellthenaskbotprivate.key -p 3000 -h 192.168.86.25

References:

https://core.telegram.org/bots/self-signed
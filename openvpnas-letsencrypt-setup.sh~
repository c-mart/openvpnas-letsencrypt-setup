#!/bin/bash

# Script configures OpenVPN AS for Let's Encrypt
# Takes two args: email address, and hostname for which cert is desired
# Usage: openvpnas-letsencrypt-setup.sh [email address] [domain]

# Download LE client
if [ ! -f /opt/certbot-auto ]; then
    cd /opt
    wget https://dl.eff.org/certbot-auto
    chmod a+x certbot-auto
fi

# Obtain certificate
if [ ! -f /etc/letsencrypt/live/$2/cert.pem ]; then
    /opt/certbot-auto certonly \
        --non-interactive \
        --agree-tos \
        --email $1 \
        --standalone \
        --pre-hook "service openvpnas stop" \
        --post-hook "service openvpnas start" \
        --domain $2
fi

# Move OpenVPN AS' default self-signed certificates to archive folder
if [ ! -L /usr/local/openvpn_as/etc/web-ssl/server.crt ];
    cd /usr/local/openvpn_as/etc/web-ssl
    mkdir old-selfsigned
    mv ca.crt server.crt server.key old-selfsigned/

    # Create symbolic links to new LE certificates
    ln -s /etc/letsencrypt/live/$2/fullchain.pem /usr/local/openvpn_as/etc/web-ssl/ca.crt 
    ln -s /etc/letsencrypt/live/$2/cert.pem /usr/local/openvpn_as/etc/web-ssl/server.crt 
    ln -s /etc/letsencrypt/live/$2/privkey.pem /usr/local/openvpn_as/etc/web-ssl/server.key

fi

service openvpnas restart

# Script certificate renewal with cron
echo "#!/bin/bash\n/opt/certbot-auto renew --quiet --pre-hook \"service openvpnas stop\" --post-hook \"service openvpnas start\"" > /etc/cron.weekly/le-renew
chmod +x /etc/cron.weekly/le-renew

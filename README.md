# Mise en place de SSL/TLS sur Apache avec certificat auto-signé (Ubuntu)

## 1. Installation d'Apache et modules SSL
```bash
sudo apt update
sudo apt install apache2 openssl -y
sudo a2enmod ssl headers
sudo systemctl restart apache2
```

## 2. Création d'un certificat auto-signé
```bash
sudo mkdir -p /etc/apache2/ssl
sudo openssl req -x509 -nodes -days 365   -newkey rsa:2048   -keyout /etc/apache2/ssl/apache.key   -out /etc/apache2/ssl/apache.crt
```
- Common Name → `localhost` ou l'IP du serveur
- Les autres champs peuvent rester par défaut

## 3. Configuration d'Apache pour HTTPS
Éditez `/etc/apache2/sites-available/default-ssl.conf` :
```apache
<IfModule mod_ssl.c>
    <VirtualHost _default_:443>
        ServerAdmin webmaster@localhost
        ServerName localhost
        DocumentRoot /var/www/html

        SSLEngine on
        SSLCertificateFile      /etc/apache2/ssl/apache.crt
        SSLCertificateKeyFile   /etc/apache2/ssl/apache.key

        # TLS sécurisé
        SSLProtocol -all +TLSv1.2 +TLSv1.3
        SSLCipherSuite TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256
        SSLHonorCipherOrder on
        SSLCompression off
        SSLSessionTickets off

        <Directory /var/www/html>
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
</IfModule>
```

## 4. Vérification des ports et activation du site SSL
- Vérifiez `/etc/apache2/ports.conf` :
```apache
Listen 80
Listen 443
```
- Activer le site SSL et redémarrer Apache :
```bash
sudo a2ensite default-ssl
sudo systemctl restart apache2
```

## 5. Tester HTTPS
- Test local :
```bash
curl -k https://localhost
```
- Vérifier la version TLS :
```bash
openssl s_client -connect localhost:443 -tls1_3
openssl s_client -connect localhost:443 -tls1_2
```

## Notes importantes
- **TLS 1.3** → la version la plus sûre
- **TLS 1.2** → encore supportée pour compatibilité
- **TLS 1.0 / 1.1 / SSL** → obsolètes, à désactiver
- Le certificat auto-signé déclenche un avertissement dans les navigateurs, valable pour tests internes

## Fichiers utiles
- Certificat : `/etc/apache2/ssl/apache.crt`
- Clé privée : `/etc/apache2/ssl/apache.key`
- Document root : `/var/www/html`
- Configuration SSL Apache : `/etc/apache2/sites-available/default-ssl.conf`

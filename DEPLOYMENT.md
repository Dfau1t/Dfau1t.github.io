# Deployment Guide for NexaStream

This document provides information about deploying and maintaining NexaStream.

## Infrastructure Overview

### Frontend Application

- **Type**: Static compiled Astro application
- **Main File**: `index.html`
- **Size**: ~254 KB (optimized)
- **Assets**:
  - CSS: Embedded (Tailwind CSS)
  - JavaScript: Minified client-side code
  - Images: Movie posters in `movie-posters/` directory

### Build Components

- `index.html` - Main application file
- `_astro/` - Compiled Astro assets
  - `affiliate.Cwr7ybFj.css` - Stylesheet
  - `client.vKsCJ4nQ.js` - Client-side JavaScript
  - `HomeApp.CRJ5Hnko.js` - Main application component
- `movie-posters/` - VOD content images

## Deployment Steps

### 1. Local Testing

Before deployment, test the application locally:

```bash
# Open in browser
file:///<path-to>/index.html

# Test all navigation links
# Verify all buttons are functional
# Check mobile responsiveness
```

### 2. Server Deployment

Deploy to your web server:

```bash
# Copy all files to web root
cp -r ./* /var/www/html/nexastream/

# Ensure proper file permissions
chmod -R 755 /var/www/html/nexastream/
chmod -R 644 /var/www/html/nexastream/*.html
chmod -R 644 /var/www/html/nexastream/_astro/*
```

### 3. Configuration

#### Web Server Setup (Nginx)

```nginx
server {
    listen 80;
    server_name nexa-stream.live www.nexa-stream.live;

    root /var/www/html/nexastream;
    index index.html;

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|webp)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name nexa-stream.live www.nexa-stream.live;

    ssl_certificate /etc/ssl/certs/nexastream.crt;
    ssl_certificate_key /etc/ssl/private/nexastream.key;

    root /var/www/html/nexastream;
    index index.html;

    # Enable compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
    gzip_min_length 1000;
}
```

#### Web Server Setup (Apache)

```apache
<VirtualHost *:80>
    ServerName nexa-stream.live
    ServerAlias www.nexa-stream.live
    DocumentRoot /var/www/html/nexastream

    <Directory /var/www/html/nexastream>
        AllowOverride All
        Require all granted
    </Directory>

    # Redirect to HTTPS
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>

<VirtualHost *:443>
    ServerName nexa-stream.live
    ServerAlias www.nexa-stream.live
    DocumentRoot /var/www/html/nexastream

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/nexastream.crt
    SSLCertificateKeyFile /etc/ssl/private/nexastream.key

    <Directory /var/www/html/nexastream>
        AllowOverride All
        Require all granted
    </Directory>

    # Enable compression
    <IfModule mod_deflate.c>
        AddOutputFilterByType DEFLATE text/html text/plain text/xml
        AddOutputFilterByType DEFLATE text/css
        AddOutputFilterByType DEFLATE application/javascript application/json
    </IfModule>

    # Cache headers
    <FilesMatch "\.(jpg|jpeg|png|gif|ico|js|css)$">
        Header set Cache-Control "max-age=2592000, public"
    </FilesMatch>
</VirtualHost>
```

### 4. SSL/TLS Certificate

Install an SSL certificate for secure connections:

```bash
# Using Let's Encrypt with Certbot
certbot certonly --webroot -w /var/www/html/nexastream -d nexa-stream.live -d www.nexa-stream.live

# Auto-renewal setup
certbot renew --quiet --no-eff-email
```

### 5. Performance Optimization

- **Compression**: Enable gzip compression on your server
- **Caching**: Set appropriate cache headers for static assets
- **CDN**: Consider using a CDN for global content delivery
- **Monitoring**: Set up uptime monitoring and alerts

### 6. Maintenance

#### Regular Backups

```bash
# Daily backup schedule
0 2 * * * tar -czf /backups/nexastream-$(date +\%Y\%m\%d).tar.gz /var/www/html/nexastream/
```

#### Log Monitoring

Monitor server logs for errors and access patterns:

```bash
# Check web server logs
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

#### Uptime Monitoring

Set up monitoring tools:
- Uptime Robot
- Pingdom
- New Relic
- Datadog

## Versioning & Updates

### File Structure

```
nexastream/
├── index.html              # Main application
├── _astro/
│   ├── affiliate.*.css
│   ├── client.*.js
│   └── HomeApp.*.js
├── movie-posters/          # VOD images
├── README.md
├── GETTING_STARTED.md
├── DEPLOYMENT.md
└── LICENSE
```

### Update Procedure

1. Test changes locally
2. Update version information
3. Deploy new files to staging environment
4. Verify functionality
5. Deploy to production
6. Monitor error logs

## Troubleshooting

### Site Not Loading

1. Check web server status: `systemctl status nginx`
2. Verify file permissions: `ls -la /var/www/html/nexastream/`
3. Check DNS resolution: `nslookup nexa-stream.live`
4. Review server logs

### Slow Performance

1. Check server resources: `top`, `free -h`
2. Enable compression
3. Clear browser cache
4. Check network latency
5. Review CDN settings

### SSL Certificate Issues

1. Verify certificate validity: `openssl s_client -connect nexa-stream.live:443`
2. Check certificate expiration
3. Renew certificate if needed
4. Check firewall rules

## Security Best Practices

1. **Keep server updated**: Regular OS and package updates
2. **Firewall**: Restrict access to necessary ports (80, 443)
3. **HTTPS**: Always use SSL/TLS encryption
4. **Headers**: Implement security headers
5. **Monitoring**: Set up intrusion detection
6. **Backups**: Maintain regular backups

## Support

For deployment assistance or questions:

- Email: support@nexastream.live
- Documentation: [Getting Started](./GETTING_STARTED.md)
- Technical Issues: Check server logs

---

**Last Updated**: May 2026

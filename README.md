# ChuWiki PHP 8.3 / CentOS 9

Updated version of ChuWiki compatible with:

- PHP 8.3
- Apache 2.4
- CentOS Stream 9
- SELinux enabled

## Installation

```bash
cd /tmp
unzip chuwiki-2.0-php8-centos9.zip

rsync -av /tmp/chuwiki-2.0/ /var/www/html/chuwiki/

chown -R apache:apache /var/www/html/chuwiki
chmod -R 775 /var/www/html/chuwiki/pages

semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/chuwiki/pages(/.*)?"
restorecon -Rv /var/www/html/chuwiki/pages

systemctl restart httpd

Bsahatkoum

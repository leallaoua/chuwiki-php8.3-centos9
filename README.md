# ChuWiki 2.0 — PHP 8.5

Mise à jour réalisée par **Abdelkrim Mecif** — [mecif@hotmail.com](mailto:mecif@hotmail.com).

Livrable actualisé le 26 octobre 2026 à partir de l’archive PHP8/CentOS9 fournie.
Les corrections de compatibilité réalisées sur Integria sont toutes représentées,
soit par un ajout, soit par une implémentation équivalente déjà présente.
Voir [CHANGELOG-PHP85.md](CHANGELOG-PHP85.md) pour la correspondance complète.

## Déploiement

PHP 8.5 avec zlib, Apache et PHP-FPM. La migration du serveur de référence vers
PHP 8.5.12 Remi a été effectuée séparément : ce dépôt contient le code applicatif,
pas les RPM ni les configurations privées du serveur.

Sur une installation existante, sauvegarder code, configuration et pages hors du
répertoire web. Copier uniquement les fichiers de code nécessaires. Ne pas
écraser `configuration.ini` ni `pages/` avec les exemples de cette archive.
Conserver les propriétaires, permissions et contextes SELinux adaptés au site.

Vérifier avant rechargement :

```bash
find sdk -name '*.php' -exec php -l {} \;
php-fpm -t
httpd -t
systemctl reload php-fpm
```

Tester accueil, navigation, recherche, historique, création, édition,
enregistrement et réouverture d’une page de test. Observer les nouvelles erreurs
avec `tail -n 0 -F /var/log/php-fpm/www-error.log` (Ctrl+C pour quitter).
Restaurer les fichiers sauvegardés puis recharger PHP-FPM en cas de régression.

## Installation et mise à jour

Le ZIP `chuwiki-3.0-php8.5.zip` contient le dossier `chuwiki-3.0-php8.5/`.
Déposer le ZIP dans `/var/www/html`, puis exécuter :

```bash
cd /var/www/html
# Sauvegarde si le répertoire existe déjà (hors du répertoire web).
if [ -d chuwiki-3.0-php8.5 ]; then
  sudo mkdir -p /var/backups/chuwiki
  sudo cp -a chuwiki-3.0-php8.5 /var/backups/chuwiki/chuwiki-3.0-php8.5.bak.$(date +%F-%H%M%S)
fi

# Extraction temporaire dans un dossier unique.
CHUWIKI_STAGE=$(mktemp -d /tmp/chuwiki-3.0-php8.5.XXXXXX)
sudo unzip /var/www/html/chuwiki-3.0-php8.5.zip -d "$CHUWIKI_STAGE"

if [ -d /var/www/html/chuwiki-3.0-php8.5 ]; then
  # Mise à jour : conserver les pages et la configuration en place.
  sudo rsync -av --exclude='pages/' --exclude='configuration.ini' "$CHUWIKI_STAGE/chuwiki-3.0-php8.5/" /var/www/html/chuwiki-3.0-php8.5/
else
  sudo rsync -av "$CHUWIKI_STAGE/chuwiki-3.0-php8.5/" /var/www/html/chuwiki-3.0-php8.5/
fi
sudo find /var/www/html/chuwiki-3.0-php8.5 -type d -exec chmod 755 {} \;
sudo find /var/www/html/chuwiki-3.0-php8.5 -type f -exec chmod 644 {} \;
sudo chown -R apache:apache /var/www/html/chuwiki-3.0-php8.5/pages
# Aucun redémarrage nécessaire pour les permissions.
# Si du code PHP a été remplacé, contrôler puis recharger PHP-FPM :
sudo php-fpm -t && sudo systemctl reload php-fpm
```

## Droits d’écriture après extraction — correctif du 26 octobre 2026

Une extraction effectuée avec root peut laisser `pages/` appartenant à root.
Le site s’ouvre alors, mais la sauvegarde échoue avec « Impossible d’écrire cette
page ». Sur Integria, PHP-FPM utilise le compte `apache` : appliquer après
extraction les commandes suivantes, en adaptant le chemin si nécessaire :

```bash
chown -R apache:apache /var/www/html/chuwiki-3.0-php8.5/pages
find /var/www/html/chuwiki-3.0-php8.5/pages -type d -exec chmod 755 {} \;
find /var/www/html/chuwiki-3.0-php8.5/pages -type f -exec chmod 644 {} \;
```

Seul le dossier des pages reçoit ces droits d’écriture ; le code peut rester
propriété de root. Aucun redémarrage des services n’est nécessaire pour ce
changement. Les propriétaires Unix ne sont pas garantis par le ZIP : cette étape
reste à exécuter sur le serveur après extraction.

Validation sur Integria : sauvegarde des pages confirmée et aucune nouvelle
entrée dans les journaux PHP-FPM/Apache pendant le test. SELinux y est désactivé.
Sur un autre serveur avec SELinux actif, prévoir aussi le contexte d’écriture
adapté (voir le README CentOS). Ne pas désactiver SELinux pour contourner ce point.

Pour observer uniquement les nouveaux messages pendant une sauvegarde :

```bash
tail -n 0 -F /var/log/php-fpm/www-error.log /var/log/httpd/error_log
```

Arrêter le suivi avec Ctrl+C.

## Publication GitHub

Décompresser puis déposer le contenu de `chuwiki-3.0-php8.5/` à la racine du dépôt.
Le fichier de configuration et la page Accueil fournis sont des exemples :
ne pas les remplacer par les données internes de production pour publication.
Les licences et les adaptations antérieures sont conservées.
Les anciens README PHP8/PHP83 décrivent les adaptations historiques ; ce README
et le changelog PHP85 font référence pour cette mise à jour.

## Validation et limites

Sur Integria, l’utilisateur a confirmé l’ouverture et la création des pages,
avec absence de nouvelles entrées PHP pendant les essais finaux.
Cette archive a une base différente : elle conserve notamment le support de
`latest`, `.wiki` et des chemins de thèmes. Elle n’est pas une copie du serveur.
Le runtime PHP n’est pas disponible dans l’environnement de préparation :
aucun contrôle de syntaxe ou test PHP local n’est annoncé comme exécuté.
Un workflow GitHub vérifie la syntaxe sous PHP 8.5 au prochain push.
Ces adaptations de compatibilité ne constituent pas un audit de sécurité complet.

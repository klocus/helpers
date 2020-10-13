# Bezpieczeństwo

## Git

Plik `.gitignore`:
```
### WordPress ###
# ignore everything in the root except the "wp-content" directory.
!wp-content/

# ignore everything in the "wp-content" directory, except:
# "mu-plugins", "plugins", "themes" and OPTIONALLY "uploads" directory
wp-content/*
!wp-content/mu-plugins/
!wp-content/plugins/
!wp-content/themes/
!wp-content/languages/
wp-content/languages/*.json
#!wp-content/uploads/ 

# ignore these plugins
wp-content/plugins/hello.php

# ignore specific themes
wp-content/themes/twenty*/

# ignore node dependency directories
node_modules/

# ignore log files and databases
*.log
*.sql
*.sqlite
```

Zwróć uwagę na opcjonalne pozostawienie katalogu `wp-content/uploads`. Jeżeli z powodu konfiguracji po stronie PA, pliki mediów są zmuszone do synchronizacji, odkomentuj linię z tym katalogiem.

## Baza Danych

Po wprowadzeniu istotnych zmian w ustawieniach WP, należy wykonać zrzut bazy danych do głównego katalogu projektu i wysłać na repozytorium.

Do importu oraz eksportu bazy danych zalecane jest użycie [WP-CLI](https://wp-cli.org/). Przykład:

```
wp db import db-dump.sql
```

Następnie wystarczy zaktualizować wpisy zawierające adres strony:
```
wp search-replace '://localhost' '://my-domain.com'
```

Proces ten można zautomatyzować przy pomocy wtyczki [WP Sync DB](https://github.com/cloudverve/wp-sync-db), która pozwala migrować dane z lokalnego serwera na serwer produkcyjny przy pomocy jednego przycisku.

## Media

Można wskazać adres serwera produkcyjnego, na którym są przechowywane media.

Apache:
```
# Use live site assets.
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteCond %{REQUEST_URI} /wp-content/uploads/.*\.(png|gif|jpe?g|bmp)$  [NC]
RewriteRule (.*) https://serwer-produkcyjny.pl%{REQUEST_URI} [L,R=301]
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
```

Nginx:
```
server {
    location ~* .(png|jpg|jpeg|gif|bmp)$ {
        expires 24h;
        log_not_found off;
        try_files $uri $uri/ @production;
    }

    location @production {
        resolver 8.8.8.8;
        proxy_pass http://serwer-produkcyjny.pl/$uri;
    }
}
```

## Komunikacja PHP z JavaScript

W celu komunikacji PHP z JavaScriptem, dobrze jest skorzystać z funkcji `wp_localize_script`. Tworzy ona obiekt JS, który może zawierać właściwości o wartościach generowanych przez PHP.

Przykładowo, do miejsca, w którym ładowany jest skrypt o nazwie `twist/js` wystarczy dodać poniższy kod, aby móc użyć `WPURLS.site` w JavaScript, czyli uzyskać wartość adresu strony ustawionej w panelu administracyjnym.

```
wp_localize_script('twist/js', 'WPURLS', [
  'site' => get_option('siteurl'),
]);
```

# WP OGÓLNE

## Własne Pola

Do tworzenia własnych pól po stronie panelu administracyjnego zalecamy korzystać z [Carbon Fields](https://carbonfields.net/) lub [TypeRocket](https://typerocket.com/).

## Przenosiny na inny serwer

W narzędziu administracyjnym bazy MySQL wykonaj poniższe zapytania ([generator](https://codepen.io/klocek/full/abzxvJw)):

```sql
UPDATE wp_options SET option_value = REPLACE(option_value, 'http://oldurl', 'http://newurl') WHERE option_name = 'home' OR option_name = 'siteurl';

UPDATE wp_posts SET guid = REPLACE(guid, 'http://oldurl', 'http://newurl');

UPDATE wp_posts SET post_content = REPLACE(post_content, 'http://oldurl', 'http://newurl');

UPDATE wp_posts SET post_excerpt = REPLACE(post_excerpt, 'http://oldurl', 'http://newurl');

UPDATE wp_postmeta SET meta_value = REPLACE(meta_value, 'http://oldurl', 'http://newurl');

UPDATE wp_termmeta SET meta_value = REPLACE(meta_value, 'http://oldurl', 'http://newurl');

UPDATE wp_comments SET comment_content = REPLACE(comment_content, 'http://oldurl', 'http://newurl');

UPDATE wp_comments SET comment_author_url = REPLACE(comment_author_url, 'http://oldurl', 'http://newurl');
```

WP BEZPIECZENSTWO
## Limit niepoprawnych logowań

WordPress domyślnie pozwala na nielimitowaną liczbę prób niepoprawnego logowania. Należy ją ograniczyć do 3, blokując możliwość logowania na kilkadziesiąt minut (20 - 30) z danego adresu IP. Jeżeli atakujący osiągnął 3 - 4 blokady, należy zwiększyć czas blokady do 24 godzin. Idealną wtyczką do tego zadania jest [Limit Login Attempts Reloaded](https://pl.wordpress.org/plugins/limit-login-attempts-reloaded/).

## Ukrycie loginów użytkowników

W WordPressie istnieją 2 metody na uzyskanie loginu administratorów. Jest to rzecz nieporządana, ponieważ prócz silnego hasła, należy również zadbać o unikalny login, różny od "admin".

### Metoda nr 1

Wprowadzając adres `https://mydomain.com/?author=1` z parametrem `author` oraz numerem `ID`, WordPress wykona przekierowanie do adresu z loginem użytkownika. W tym przypadku do `https://mydomain.com/author/login-admina/`.

**Zapobieganie:**

Wystarczy dodać odpowiednie reguły na końcu pliku `.htaccess`:
```
RewriteEngine On
RewriteCond %{REQUEST_URI} !^/wp-admin [NC]
RewriteCond %{QUERY_STRING} author=\d
RewriteRule ^ /? [L,R=301]
```

Lub akcję w pliku `functions.php` motywu:
```php
add_action('template_redirect', function() {
	$is_author_set = get_query_var('author', '');
	if ($is_author_set != '' && !is_admin()) {
		wp_redirect(home_url(), 301);
		exit;
	}
});
```

Co ważne, oba powyższe sposoby nie blokują adresu URL z parametrem `author` administratorowi.

### Metoda nr 2

Standarodwo w WordPressie włączone jest REST API z nieograniczonym dostępem, z którego można zebrać wiele ciekawych informacji, w tym nieporządanego loginu administratora. Wprowadzając adres `http://mydomain.com/wp-json/wp/v2/users/1` uzyskamy dane JSON:

```
{"id":1,"name":"Jan Kowalski","url":"","description":"","link":"https:\/\/mydomain.com\/author\/login-admina\/","slug":"login-admina","avatar_urls":{"24":"http:\/\/1.gravatar.com\/avatar\/4b3cc48f9156b9d65fae9742eebe623f?s=24&d=mm&r=g","48":"http:\/\/1.gravatar.com\/avatar\/4b3cc48f9156b9d65fae9742eebe623f?s=48&d=mm&r=g","96":"http:\/\/1.gravatar.com\/avatar\/4b3cc48f9156b9d65fae9742eebe623f?s=96&d=mm&r=g"},"meta":[],"_links":{"self":[{"href":"https:\/\/mydomain.com\/wp-json\/wp\/v2\/users\/3"}],"collection":[{"href":"https:\/\/mydomain.com\/wp-json\/wp\/v2\/users"}]}}
```

**Zapobieganie:**

Należy zablokować REST API dla niezalogowanych użytkowników w pliku `functions.php` motywu dodając odpowiedni filtr:
```php
add_filter('rest_authentication_errors', function($result) {
    if (!empty($result)) {
        return $result;
    }
    if (!is_user_logged_in()) {
        return new WP_Error('rest_not_logged_in', 'You are not currently logged in.', ['status' => 401]);
    }
    return $result;
});
```
**Uwaga!** Wyłączenie REST API może wpłynąć na działanie niektórych wtyczek, np. na *Contact Form 7*!

## Wyłączenie edycji z poziomu PA

Oddając skończony projekt klientowi, ważne jest, aby wyłączyć możliwość edycji plików wtyczek oraz motywu przez panel administracyjny. Wystarczy dodać poniższą definicję do pliku `wp-config.php`:

```php
define('DISALLOW_FILE_EDIT', true);
```

## Zabezpieczenie adresu logowania

### Metoda nr 1

Zainstaluj plugin [Rename wp-login.php](https://wordpress.org/plugins/rename-wp-login/). Można to zrobić ręcznie, lecz nasze zmiany w pliku `wp-login.php` mogą zostać nadpisane przez aktualizację WordPressa.

Zalecamy zmienić nazwę na `trui-login`.

### Metoda nr 2

Dodaj akcję do pliku `functions.php`, która wymaga określonego parametru w adresu URL podczas logowania:
```php
add_action('login_head', function() {
    if (!isset($_GET['mysecretstring'])) {
        exit;
    }
});
```

## Pluginy i baza danych

Staraj się ograniczyć liczbę używanych pluginów do absolutnego minimum. Sprawdź czy danej rzeczy nie da się załatwić przy pomocy filtru lub akcji w pliku motywu.

Podczas instalacji WordPressa stosuj inny niż domyślny prefiks dla bazy danych.

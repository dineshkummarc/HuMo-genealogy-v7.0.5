# HuMo-genealogy AI coding guide

## Big-picture architecture
- Public entrypoint is index.php (front site); admin entrypoint is admin/index.php. Both bootstrap sessions, autoload Composer, connect DB, load language, then dispatch to page handlers.
- Front flow: index.php builds $config via include/config.php, then selects a controller in app/Controller/* based on the routed page, gets data from app/Model/*, and finally renders via views/*. layout.php is the shared page shell.
- Routing is custom: app/Routing/Router.php has a manual routes_array; route order matters (see comments). IndexModel uses it to resolve $page and URL rewrite behavior.
- URL generation must go through include/ProcessLinks.php, which mirrors routing and respects $humo_option["url_rewrite"]. Example usage is in views/list_names.php.

## Data + DB conventions
- DB connection is centralized in include/db_login.php and uses PDO. It supports Docker env vars (MYSQL_DATABASE, MYSQL_USER, MYSQL_PASSWORD) and forces sql_mode tweaks (ONLY_FULL_GROUP_BY, NO_ZERO_DATE) after connect.
- Most queries are string-built; sanitize user input with Genealogy\Include\SafeTextDb->safe_text_db() (see include/SafeTextDb.php) when interpolating into SQL.
- Many models extend app/Model/BaseModel.php to access $dbh, $db_functions, $tree_id, $user, $humo_option, $uri_path, $selectedFamilyTree. Construct these via include/config.php.

## Localization
- Translations are handled by languages/gettext.php and global __() calls in views; language selection is done in IndexController and admin/index.php.

## Key directories
- app/Controller and app/Model form the front-end MVC-ish layer.
- views/ holds front-end templates; admin/views/ holds admin templates.
- include/ contains shared utilities (DB, sanitization, link building, GEDCOM processing, etc.).
- assets/, css/, images/ provide static resources (Bootstrap, jQuery, etc.).

## External dependencies
- Composer autoload is required (vendor/autoload.php). composer.json shows phpmailer/phpmailer as the primary dependency and PSR-4 namespaces for Genealogy\\App, Genealogy\\Include, Genealogy\\Languages, Genealogy\\Admin.

## Tests (minimal)
- There are basic PHPUnit experiments in tests/app/*. Example: tests/app/listModelTest.php uses ListModel directly but lacks a full test bootstrap.

## Examples to follow when adding features
- Add a new front page: register a route in app/Routing/Router.php, add a controller in app/Controller, model in app/Model (extend BaseModel), and a view in views/. Wire the controller in index.php switch logic and render via views/layout.php.
- Generate links using ProcessLinks rather than hardcoding URLs to keep url_rewrite compatibility.

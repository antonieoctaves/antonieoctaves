# PHP — 30-Minute Presentation Cheatsheet
### Project: Project-SokoHub (group `webers-grp`) · stack: XAMPP = Apache + PHP 8 + MySQL

---

## 1. Your 30-second pitch (memorise these three sentences)

> "SokoHub is an online marketplace where vendors list farm produce and buyers order it. It is built in **plain PHP 8** on **Apache** with a **MySQL** database — no frameworks — so we can show exactly how a request travels from the browser to the database and back. We run it locally on **XAMPP**, and we have a **system-check page and automated smoke tests** so anyone in the group can verify their machine is configured correctly."

Then: "Let me show you the request flow, then the code."

---

## 2. The ONE thing you must understand: what happens when a page is opened

Say this out loud, in order — it is the most-asked exam question:

1. Browser sends `GET http://localhost/Project-SokoHub/index.php` to **Apache** (port 80).
2. Apache sees the `.php` extension → hands the file to the **PHP interpreter** (mod_php). Apache never sends PHP code to the browser.
3. PHP runs top to bottom: loads `config/config.php` → `config/db.php` → `includes/functions.php`.
4. PHP asks **MySQL** (via PDO) for the products: `SELECT … FROM products`.
5. PHP mixes the rows into the HTML template (`includes/header.php` … `foreach` … `footer.php`).
6. PHP sends **pure HTML** back to Apache → browser. The browser only ever sees the *result*, never the PHP source.

**One-liner if asked "what is PHP?"**
> "A server-side scripting language: the code runs on the server and produces HTML. Because it runs before the page is sent, it can talk to the database, keep sessions and protect secrets — things JavaScript in the browser cannot do."

---

## 3. Eight PHP concepts — each one exists in your code (point at the file)

| # | Concept | Say this | Where to point |
|---|---------|----------|----------------|
| 1 | **PHP is embedded in HTML** | Tags `<?php … ?>` open/close; outside them is plain HTML | `index.php` line 48 (`<?php foreach`) |
| 2 | **`require_once` / template reuse** | Shared header + footer included on every page → no copy-paste | `index.php` lines 11, 30, 60 |
| 3 | **Functions & type declarations** | `declare(strict_types=1)`, typed params/returns (`?PDO`, `: string`, `never`) = errors caught early | `includes/functions.php` 6–9, `config/db.php` 56 |
| 4 | **Superglobals** | `$_SERVER` (host, document root), `$_GET`, `$_POST`, `$_SESSION` — built-in arrays PHP fills for you | `config/config.php` 40–60 |
| 5 | **PDO + prepared statements** | SQL is sent separately from the data, so input can never become SQL → **stops SQL injection** | `config/db.php` 73–76, 102–113 |
| 6 | **Sessions & cookies** | `session_start()` keeps the logged-in user across pages; `session_regenerate_id(true)` on login stops session fixation | `config/config.php` 29–31, `includes/auth.php` 42–46 |
| 7 | **Output escaping** | every echoed user value goes through `e()` = `htmlspecialchars()` → **stops XSS** | `includes/functions.php` 9–12 |
| 8 | **Graceful error handling** | `try/catch (Throwable)`; if MySQL is down the page still renders with demo data instead of a white screen | `config/db.php` 78–84 |

---

## 4. Security — say these four, they impress

- **SQL injection** → prepared statements with bound parameters (`PDO::ATTR_EMULATE_PREPARES => false`).
- **XSS** → `e()` around every output value.
- **CSRF** → `csrf_token()` hidden field + `csrf_verify()` on POST forms.
- **Secrets exposed over HTTP** → `.htaccess` returns 403 for `config/`, `database/`, `includes/`, `storage/`; `config/db.local.php` is git-ignored so passwords never reach GitHub.

---

## 5. Database in 20 seconds (6 tables)

`users` → `vendors` → `products` → `orders` → `order_items`, plus `categories`.

Say:
> "It is normalised to third normal form: a vendor is a user with a shop, a product belongs to a vendor and a category, an order belongs to a buyer and a vendor, and `order_items` is the junction table that stores quantity and the **price at the time of purchase**."

Details worth dropping:
- **Foreign keys** with `ON DELETE CASCADE` (delete a vendor → their products go too) and `ON DELETE RESTRICT` on `order_items.product_id` (you cannot delete a product someone already bought — preserves history).
- **`DECIMAL(10,2)` for money, never `FLOAT`** — floats round wrong.
- **`utf8mb4`** — full Unicode including emoji and Swahili diacritics.
- **`ENUM`** for `role` (buyer/vendor/admin) and order `status` (pending→paid→shipped→delivered/cancelled).
- **Indexes** on foreign keys and unique on `email` and product `slug`.

---

## 6. Live demo — 4 clicks, 90 seconds

1. **`http://localhost/Project-SokoHub/`** → "Apache is serving PHP; this page queried MySQL and rendered 6 products. If the yellow banner shows, MySQL isn't set up and the page falls back to demo data instead of crashing."
2. **`/health.php`** → "Our diagnostics page: PHP version, `pdo_mysql` extension, MySQL connection, whether all 6 tables exist, writable folders. Every red row tells you exactly how to fix it." *(This is your strongest slide — it shows engineering discipline.)*
3. **`/docs/test/run_tests.php`** → "Automated smoke tests: files present, helpers correct, database reachable, tables created. Green = the group member's machine is ready."
4. **`http://localhost/phpmyadmin`** → open `sokohub` → `products` → "This is where the data the page displayed actually lives."

Have a **screenshot of each** ready in case Apache dies on stage. (In XAMPP Control Panel: Start Apache → Start MySQL, wait for green.)

---

## 7. Likely questions — short answers

| Question | Answer |
|---|---|
| What is XAMPP? | A free installer bundling **A**pache (web server), **M**ySQL/MariaDB (database), **P**HP + **P**erl. `htdocs` is the web root: anything there is reachable at `http://localhost/...`. |
| Why plain PHP and not Laravel? | The unit is Web II / DBMS — showing the raw request/response and SQL is the point. A framework would hide the mechanics we must demonstrate. |
| PHP vs JavaScript? | PHP runs **on the server before** the page is sent; JS runs **in the browser after**. PHP can touch the database and files; JS cannot. |
| GET vs POST? | GET puts data in the URL (visible, bookmarkable, for reading). POST sends it in the request body (for creating/updating, and for passwords). |
| Why PDO, not `mysqli`? | PDO has one API for many databases, supports named parameters and exceptions, and lets us force real server-side prepared statements. |
| How do you stop SQL injection? | Bound parameters — the query template and the data travel separately, so data is never parsed as SQL. |
| Session vs cookie? | The cookie only stores a session **ID**; the actual data stays on the server in the session. Safer, and it can hold arrays. |
| What does 200 / 404 / 500 mean? | 200 OK, 404 not found, 500 server error. Our preview server returns them correctly. |
| Where are passwords stored? | `password_hash()` bcrypt hashes in `users.password_hash` — never plain text. Verify with `password_verify()`. |
| What is CRUD? | Create, Read, Update, Delete — the four operations we map onto SQL `INSERT/SELECT/UPDATE/DELETE`. |
| What is MVC? | Model-View-Controller separation. We use a lighter version: data access in `config/db.php`, views in `includes/`, logic in page files. |
| Did you finish every feature? | "No — the scaffold, database and diagnostics are done and tested. Login, cart and checkout are the next sprint," then list who owns what. |

---

## 8. If something breaks during the demo (say it calmly)

- **Blank page** → "PHP errors are enabled in `config/config.php`; the log is `storage/logs/php-error.log`." Then refresh.
- **"Object not found"** → capitalisation of the URL: `/Project-SokoHub/`.
- **"Access denied for user root"** → "MySQL password isn't in `config/db.local.php`."
- **"Unknown database sokohub"** → "`database/schema.sql` hasn't been imported." Then hit `/health.php` and let it tell the story.
- **Apache won't start** → port 80 taken; change to 8080 in `httpd.conf`, or just show the screenshots.

---

## 9. Before you walk in (5-minute checklist)

- [ ] Apache **and** MySQL green in XAMPP Control Panel.
- [ ] `database/schema.sql` + `seed.sql` imported → home page shows 6 products, **no** yellow banner.
- [ ] `/health.php` all green, `/docs/test/run_tests.php` all PASS.
- [ ] Screenshots saved of all four pages (offline backup).
- [ ] You can name each file's job without opening it: `index.php` view, `config/` settings + database, `includes/` layout + helpers, `database/` schema, `docs/test/` tests.
- [ ] Agree with the group **who answers which area** (frontend / database / security).

**Golden rule:** if you don't know, say *"That part is <name>'s area, but what I know is …"* and give the closest fact you do know. Never invent API names.

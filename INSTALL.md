# Sham portal 1.1: install, run and update

One installation runs five sites from one MySQL database:

| Address | What it is |
|---|---|
| sham.et | Company site: all products by field, news and updates, about, contact, legal pages, RSS, news by e-mail |
| media.sham.et | Sham MediaOS site: its own pages, news, releases, editions and prices |
| taligaa.sham.et | Taligaa site |
| store.sham.et | Store (buy, renew, receipts), customer accounts (keys, receipts, downloads) and the admin at /admin |
| licensing.sham.et | Licence server: check-ins, signed update checks and downloads for Sham MediaOS installations |

More products are added in the admin (Products → New); each gets pages, news, releases and, if you want, its own subdomain.

Requirements: PHP 8.1 or newer with pdo_mysql, sodium, openssl, curl, mbstring and gd (standard on cPanel), MySQL 5.7+ or MariaDB 10.3+.

## 1. Install with the setup wizard (recommended, no manual steps)

1. cPanel → Security → **Manage API Tokens** → Create. Give it a name such as `sham-setup` and an expiry date of tomorrow. Copy the token.
2. cPanel → File Manager → `public_html` → Upload **`sham-setup.php`** (one file).
3. Open **https://sham.et/sham-setup.php**. It checks the server, then asks for:
   - your cPanel user name and the API token,
   - the five addresses (filled in for you),
   - your owner account (name, e-mail, password),
   - optionally the licence signing key (the contents of `D:\JMN\keys\sham-licensing-ed25519.pem`).
4. Press **Install**. In about a minute the wizard:
   - creates the MySQL database and its user with a random password,
   - creates the subdomains store, licensing, media and taligaa with `public_html` as their folder,
   - starts AutoSSL for the HTTPS certificates,
   - unpacks the code to `~/sham-portal` (outside the web root) and the public files to `public_html` (an existing `index.php` or `.htaccess` is kept under a new name; the PHP version cPanel chose is kept),
   - creates the tables, the first content and your owner account,
   - imports the signing key if you gave it,
   - adds the cron job,
   - deletes itself.
5. Delete the API token in cPanel (it is no longer needed).

Other panels (Plesk, DirectAdmin, …): choose "Another panel" in the wizard. Create an empty database with a user that has all privileges, and point the four subdomains at the same folder as the main domain, then type the database details. The wizard does the rest.

Safety: only someone who has a working cPanel token or database password can run the wizard. It stops working 12 hours after it was uploaded, and for good once the portal is installed.

### Alternative: the package

`sham-portal-1.1.0.zip` holds the same code for those who prefer to upload folders: extract it in the home folder, copy `sham-portal/public/*` (with `.htaccess`) into `public_html`, create the database and subdomains yourself, put 32 random letters in `sham-portal/app/install-token.txt` and open https://sham.et/install.

## 2. First sign-in

1. https://store.sham.et/admin → sign in → scan the QR code with an authenticator app. **Save the ten recovery codes.**
2. **Signing key** (if you did not give it to the wizard): paste `sham-licensing-ed25519.pem`. The page must say Sham MediaOS installations trust it.
3. **Settings → Company**: TIN, VAT number, registration number, phone, e-mails, hosting details (they fill in the legal pages and receipts).
4. **Settings → Website**: home page headline and text, the announcement bar, social links.
5. **Settings → Store / Payment methods / E-mail**: rates, VAT, merchant keys (each page shows the webhook address), SMTP, then "Send a test e-mail".
6. **Admins**: add staff with a role: *Sales* (orders, licences, enquiries) or *Editor* (news, pages, products, media, releases). Everyone signs in with an authenticator app.

## 3. Running the sites

- **News and updates**: write posts for the company or a product, with a cover image, schedule them, and send them to subscribers. Visitors subscribe on /news (double opt-in; every e-mail has a one-click unsubscribe).
- **Products**: name, field, colour, logo, text, regulatory note, editions and prices, and any number of extra pages (Features, Documentation …) shown in the product's menu.
- **Pages**: about, legal texts and any other company page; tick "menu" or "footer" to show them.
- **Media library**: images are re-saved by the server (hidden content and camera data removed); PDFs and ZIPs are offered as downloads. Paste the Markdown it gives into any text.
- **Releases**: publish each version with notes and the file (or put large files in `sham-portal/app/storage/releases/incoming`). Installations of Sham MediaOS see the new version through the update check, and customers download it from https://store.sham.et/account, only within their update period.
- **Customer accounts**: customers sign in at store.sham.et/account with a one-time link sent to the e-mail on their licence and find their keys, receipts and downloads.

## 4. Updating the portal

Admin → **System** → Update the portal: upload the new `sham-portal-x.y.z.zip` and enter an authenticator code. The portal keeps a copy of the current code, copies the new files (never `config.php`, the storage folder or your uploads) and applies the database changes by itself. "Previous code" puts an earlier copy back.

## 5. Backups

Admin → System → **Download backup** gives the whole database as a compressed SQL file. Keep it together with a copy of `sham-portal/app/config.php`: config.php holds the secret that decrypts the stored payment keys and the signing key. cPanel → Backup is a good second copy.

## 6. Scheduled work

Every 10 minutes the portal confirms pending payments, closes unpaid orders, sends expiry reminders and news e-mails in small batches, and tidies up. The wizard adds the cron job; without one, the portal does the same work by itself while people visit the sites.

## 7. Checks

- Admin → System lists the health checks (PHP, extensions, HTTPS, signing key, scheduled work, install token removed).
- https://licensing.sham.et/health answers `{"ok":true,…,"signing":true}`.

## Moving to a new server

Run the wizard on the new server once, or copy `sham-portal/` (with `config.php`) and `public_html/`, restore the database backup, and change `db` in `config.php` if the database name or user changed.

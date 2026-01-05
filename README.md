# SalePro — Point-of-Sale / Inventory Management Web Application

This repository contains SalePro, a Laravel-based web application for retail, inventory and sales management. This README provides a comprehensive guide to setting up, developing, testing, and deploying the application on a developer workstation (Windows/XAMPP) as well as common operational tasks.

---

## Table of Contents

- [Requirements](#requirements)
- [Quick Start (Windows / XAMPP)](#quick-start-windows--xampp)
- [Full Setup](#full-setup)
	- [Clone repository](#clone-repository)
	- [Environment variables](#environment-variables)
	- [Composer dependencies](#composer-dependencies)
	- [Node assets](#node-assets)
	- [Database setup, migrations and seeders](#database-setup-migrations-and-seeders)
	- [Storage and permissions](#storage-and-permissions)
- [Running the application](#running-the-application)
- [Testing](#testing)
- [Common artisan tasks](#common-artisan-tasks)
- [Application structure](#application-structure)
- [Deployment checklist](#deployment-checklist)
- [Contributing guide](#contributing-guide)
- [Troubleshooting](#troubleshooting)
- [Credits & License](#credits--license)

---

## Requirements

- PHP 8.0+ (verify with `php -v`)
- Composer (latest stable)
- Node.js 14+ and npm (or Yarn)
- MySQL / MariaDB (or other supported DB)
- Git
- XAMPP (Windows) or equivalent LAMP stack for local development

If you plan to run services like queues or websockets, also install Redis and Supervisor (or use a service provider on production).

## Quick Start (Windows / XAMPP)

1. Install XAMPP and ensure Apache & MySQL are running.
2. Place the repository in `C:\xampp\htdocs\salepro` (already present).
3. Copy `.env.example` to `.env` and configure DB credentials, mail, and other keys.
4. From an elevated PowerShell (open as Administrator) run:

```powershell
Push-Location 'C:\xampp\htdocs\salepro'
composer install --no-interaction --prefer-dist
cp .env.example .env
php artisan key:generate
php artisan migrate --seed
php artisan storage:link
npm install
npm run dev
```

5. Open browser to `http://localhost/salepro` or configure a virtual host to point to the `public` directory for a cleaner URL.

Note: On Windows you might need to use `copy` instead of `cp` or perform the `.env` copy via Explorer.

## Full Setup

### Clone repository

If you haven't already cloned:

```bash
git clone https://github.com/techsalaf/salepro.git
cd salepro
```

### Environment variables

- Duplicate `.env.example` and update the following at minimum:
	- `APP_NAME`, `APP_ENV`, `APP_URL`
	- `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`
	- `MAIL_*` settings for password resets and notifications
	- Payment provider keys (`PAYSTACK_*`, `FLUTTERWAVE_*`, `STRIPE_*`, `RAZORPAY_*`, `XENDIT_*`, etc.) if used

Run `php artisan key:generate` to populate `APP_KEY`.

### Composer dependencies

Install PHP dependencies:

```bash
composer install
php artisan migrate
php artisan db:seed
```

If you see package installation issues, run `composer diagnose` and consider increasing memory by running:

```powershell
php -d memory_limit=-1 composer install
```

### Node assets

Install JS/CSS dependencies and build assets:

```bash
npm install
npm run dev      # development build w/ hot-reload
npm run prod     # production minified assets
```

Alternatively use `yarn`.

### Database setup, migrations and seeders

- Configure DB in `.env`.
- Run migrations:

```
php artisan migrate
```

- Seed test/sample data (if seeders exist):

```
php artisan db:seed
```

If you need to refresh the DB:

```
php artisan migrate:refresh --seed
```

### Storage and permissions

Create the symbolic link to make storage accessible from `public`:

```
php artisan storage:link
```

On Windows, link creation should work; if not, manually copy `storage/app/public` contents into `public/storage` for testing.

## Running the application

- Using XAMPP: place the project under `htdocs` and navigate to `http://localhost/salepro`.
- Using Laravel's built-in server for development:

```
php artisan serve --host=127.0.0.1 --port=8000
```

This runs a development server at `http://127.0.0.1:8000`.

### Background workers and scheduler

- Run queue workers:

```
php artisan queue:work --tries=3
```

- Schedule (add to cron on Linux; on Windows use Task Scheduler):

```
* * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1
```

## Testing

Run PHPUnit tests:

```
vendor\\bin\\phpunit
```

Or via Composer script:

```
composer test
```

If tests require a separate testing database, create one and configure the `phpunit.xml` or `phpunit.xml.bak` accordingly.

## Common artisan tasks

- Cache config: `php artisan config:cache`
- Clear cache: `php artisan cache:clear`
- Clear views: `php artisan view:clear`
- Optimize autoloaders: `composer dump-autoload -o`

## Application structure (high level)

- `app/` — core application logic (Controllers, Models, Services, Traits)
- `Modules/` — modular features (e.g., `Modules/Manufacturing`)
- `routes/` — application routes (`web.php`, `api.php`)
- `resources/views/` — Blade templates
- `public/` — web root (assets, entry `index.php`)
- `database/` — migrations, seeders, factories
- `storage/` — logs, compiled views, file storage

Key files to inspect when developing:

- `app/Providers` for service bindings
- `config/` directory for environment sensitive settings
- `routes/web.php` and `routes/api.php` for endpoints

## Development notes specific to this repo

- There is an `unzipper.php` helper at the repository root used for package extraction or custom import flows — inspect it if you work on installers or importer utilities.
- Modules are under the `Modules/` directory; each module commonly includes its own `Controllers`, `Models`, `Migrations` and `Routes`.

## Deployment checklist

Before deploying to production, ensure:

- `APP_ENV=production` and `APP_DEBUG=false` in `.env`
- Proper `APP_KEY` is set
- Cached config & routes: `php artisan config:cache` and `php artisan route:cache`
- Assets built with `npm run prod` and deployed to `public/`
- `storage` and `bootstrap/cache` writable by web server
- Database backups exist and migration plan verified
- Queues configured and workers running (Supervisor systemd on Linux)

## Contributing guide

- Fork the repo and create a feature branch: `git checkout -b feature/short-description`
- Follow PSR-12 code style and run `composer test` before submitting PRs.
- Provide clear PR descriptions and reference related issues.

## Troubleshooting

- 500 errors: check `storage/logs/laravel.log` and ensure `APP_DEBUG=true` for local debugging.
- Permissions issues: ensure `storage` and `bootstrap/cache` are writable.
- Composer memory errors: run with increased memory: `php -d memory_limit=-1 composer install`.

## Useful commands summary

```
composer install
npm install
cp .env.example .env
php artisan key:generate
php artisan migrate --seed
php artisan storage:link
npm run dev
php artisan serve
```

## Credits & License

This project uses Laravel and many community packages. Check `composer.json` and `package.json` for full credits. The project is distributed under the MIT license unless otherwise specified in repository metadata.

---

For further assistance, please refer to the [official Laravel documentation](https://laravel.com/docs) or open an issue in this repository. Happy coding!
# Reltroner HR App — Local Testing Stack (Laragon, Laravel 12)

This README documents our chat-driven debugging session on setting up a **stable testing toolchain** for a Laravel 12 project on **Windows + Laragon**, after hitting version conflicts when trying to install Pest v4.

---

## Context

While preparing developer tooling, we ran into strict version constraints:

```

composer require --dev ^
pestphp/pest:^4.1 ^
pestphp/pest-plugin-laravel:^4.1 ^
phpunit/phpunit:^12.5 ^
nunomaduro/collision:^8.8.2 ^
filp/whoops:^2.18.1 -W

````

Composer reported:

- `pestphp/pest 4.1` requires **PHPUnit 12**.
- `nunomaduro/collision 8.8.2` requires `symfony/console >= 7.3`.
- The current Laravel skeleton pins **PHPUnit 11** and `symfony/console 7.2.x`.
- Result: dependency conflicts and rollbacks.

**Decision:** use **PHPUnit 11** (supported by Laravel 12) first, skip Pest for now. You still get clean, fast tests locally and in CI. We can migrate to Pest later after bumping Symfony & Collision.

---

## 0) Clean caches (Windows PowerShell)

```powershell
cd C:\laragon\www\reltronerhrapp
del .\bootstrap\cache\*.php -Force
php artisan optimize:clear
````

---

## 1) Install safe dev dependencies

```powershell
composer require --dev "phpunit/phpunit:^11.5.33" "nunomaduro/collision:^8.6" -W
```

Notes:

* `phpunit/phpunit:^11.5.33` → stable on Laravel 12 (PHP 8.2/8.3).
* `nunomaduro/collision:^8.6` → compatible with current Symfony 7.2.x.

---

## 2) Create a testing environment (SQLite)

```powershell
Copy-Item .env.example .env.testing
New-Item -ItemType Directory -Path database -Force | Out-Null
New-Item -ItemType File -Path database/testing.sqlite -Force | Out-Null
```

Edit **`.env.testing`** (minimal):

```dotenv
APP_ENV=testing
APP_KEY=base64:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=sqlite
DB_DATABASE=database/testing.sqlite

CACHE_DRIVER=array
SESSION_DRIVER=array
QUEUE_CONNECTION=sync
MAIL_MAILER=array
FILESYSTEM_DISK=local
```

---

## 3) Example Feature tests

> Adjust routes/columns to match your app (RBAC, employees table, etc.).

**`tests/Feature/AuthTest.php`**

```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Support\Facades\Hash;
use Tests\TestCase;

class AuthTest extends TestCase
{
    public function test_login_page_loads(): void
    {
        $this->get('/login')->assertStatus(200);
    }

    public function test_guest_is_redirected_from_dashboard(): void
    {
        $this->get('/dashboard')->assertRedirect('/login');
    }

    public function test_user_can_login(): void
    {
        $user = User::factory()->create([
            'email' => 'admin@example.com',
            'password' => Hash::make('secret123'),
        ]);

        $resp = $this->post('/login', [
            'email' => 'admin@example.com',
            'password' => 'secret123',
        ]);

        $resp->assertRedirect('/dashboard');
        $this->assertAuthenticatedAs($user);
    }
}
```

**`tests/Feature/RbacAccessTest.php`**

```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Tests\TestCase;

class RbacAccessTest extends TestCase
{
    private function makeUserWithRole(string $role): User
    {
        $u = User::factory()->create();

        // Replace with your RBAC implementation:
        // Example column on users table:
        $u->role = $role;
        $u->save();

        // If using spatie/permission:
        // $u->assignRole($role);

        return $u;
    }

    public function test_admin_can_access_users_index(): void
    {
        $admin = $this->makeUserWithRole('admin');

        $this->actingAs($admin)
            ->get('/users')     // replace with your route
            ->assertStatus(200);
    }

    public function test_employee_cannot_access_users_index(): void
    {
        $emp = $this->makeUserWithRole('employee');

        $this->actingAs($emp)
            ->get('/users')     // replace with your route
            ->assertForbidden(); // or ->assertRedirect() based on middleware behavior
    }
}
```

**`tests/Feature/SmokeRoutesTest.php`**

```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Tests\TestCase;

class SmokeRoutesTest extends TestCase
{
    public function test_critical_pages_load_for_admin(): void
    {
        $admin = User::factory()->create(); // assign admin role if required
        $this->actingAs($admin);

        $routes = [
            '/dashboard',
            '/employees',
            '/tasks',
            '/attendance',
            '/payroll',
            '/leave',
        ];

        foreach ($routes as $r) {
            $this->get($r)->assertSuccessful(); // any 2xx
        }
    }
}
```

**`tests/Feature/EmployeeCrudTest.php`**

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use App\Models\User;
use Tests\TestCase;

class EmployeeCrudTest extends TestCase
{
    use RefreshDatabase;

    public function test_admin_can_create_employee(): void
    {
        $admin = User::factory()->create();
        $this->actingAs($admin);

        $payload = [
            'name' => 'John Tester',
            'email' => 'john.tester@example.com',
            'position' => 'QA',      // adapt to your schema
            'salary' => 5000000,     // adapt to your schema
        ];

        $this->post('/employees', $payload) // replace with your route
            ->assertRedirect();

        $this->assertDatabaseHas('employees', [
            'email' => 'john.tester@example.com',
        ]);
    }

    public function test_validate_required_fields_on_create(): void
    {
        $admin = User::factory()->create();
        $this->actingAs($admin);

        $this->post('/employees', []) // empty
            ->assertSessionHasErrors(['name', 'email']); // adjust to your validation rules
    }
}
```

*(Optional)* **`tests/Feature/AttendanceRuleTest.php`**

```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Support\Carbon;
use Tests\TestCase;

class AttendanceRuleTest extends TestCase
{
    public function test_prevent_double_checkin_same_day(): void
    {
        $user = User::factory()->create();
        $this->actingAs($user);

        $this->post('/attendance/check-in', [
            'lat' => '-6.2',
            'lng' => '106.8',
            'time' => Carbon::now()->toDateTimeString(),
        ])->assertRedirect();

        $this->post('/attendance/check-in', [
            'lat' => '-6.2',
            'lng' => '106.8',
            'time' => Carbon::now()->toDateTimeString(),
        ])->assertStatus(422); // or assertSessionHasErrors([...]) depending on your controller
    }
}
```

---

## 4) Run tests

```powershell
php artisan test
```

---

## 5) Optional: GitHub Actions CI (PHPUnit)

Create `.github/workflows/ci-tests.yml`:

```yaml
name: CI - Laravel (PHPUnit)

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          extensions: mbstring, intl, pdo_sqlite, dom, curl
          coverage: none

      - name: Cache composer
        uses: actions/cache@v4
        with:
          path: vendor
          key: composer-${{ hashFiles('**/composer.lock') }}

      - name: Install dependencies
        run: composer install --no-interaction --prefer-dist

      - name: Prepare .env & sqlite
        run: |
          cp .env.example .env.testing
          mkdir -p database
          :> database/testing.sqlite

      - name: Run migrations
        env:
          APP_ENV: testing
        run: php artisan migrate --force

      - name: Run tests (PHPUnit)
        env:
          APP_ENV: testing
        run: php artisan test --testsuite=Feature
```

---

## Troubleshooting

* **Composer complains about Pest 4 / PHPUnit 12 / Collision 8.8+:**

  * Stick to `phpunit/phpunit:^11.5.33` and `nunomaduro/collision:^8.6` for now.
* **Route assertions fail (Forbidden vs Redirect):**

  * Match your auth/permission middleware behavior.
* **SQLite path issues on Windows:**

  * Ensure `DB_DATABASE=database/testing.sqlite` (relative to project root) and that the file exists.

---

## Upgrade path to Pest (later)

When ready to adopt Pest v4:

1. Upgrade Symfony Console to **≥ 7.3**, Collision to **≥ 8.8.2**, and PHPUnit to **12.x**.
2. Then:

```powershell
composer require --dev pestphp/pest:^4 pestphp/pest-plugin-laravel:^4 phpunit/phpunit:^12 -W
php artisan pest:install
```

---

## Commands Snapshot (from the session)

```
PS C:\laragon\www\reltronerhrapp> del .\bootstrap\cache\*.php -Force
PS C:\laragon\www\reltronerhrapp> php artisan optimize:clear
PS C:\laragon\www\reltronerhrapp> composer require --dev "phpunit/phpunit:^11.5.33" "nunomaduro/collision:^8.6" -W
# (previous strict requires with Pest 4 failed due to dependency constraints)
```

---

**Status:** ✅ Stable PHPUnit-based testing workflow on Laravel 12 (Laragon/Windows).

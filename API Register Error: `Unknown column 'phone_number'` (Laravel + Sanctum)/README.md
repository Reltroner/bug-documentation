# API Register Error: `Unknown column 'phone_number'` (Laravel + Sanctum)

This repo documents a registration failure caused by trying to insert a non-existent column (`phone_number`) into the `users` table when building a Laravel API with Sanctum.

---

## ❗ What Happened

**Request**
```

POST [http://127.0.0.1:8000/api/auth/register](http://127.0.0.1:8000/api/auth/register)
Content-Type: application/json

````

**Payload**
```json
{
  "name": "test",
  "email": "test@example.com",
  "password": "password",
  "password_confirmation": "password",
  "device_name": "web",
  "is_admin": true
}
````

**Response (500)**

```json
{
  "message": "SQLSTATE[42S22]: Column not found: 1054 Unknown column 'phone_number' in 'field list' (Connection: mysql, SQL: insert into `users` (`name`, `email`, `password`, `phone_number`, `updated_at`, `created_at`) values (test, test@example.com, $2y$12$..., ?, 2025-09-10 07:13:12, 2025-09-10 07:13:12))",
  "exception": "Illuminate\\Database\\QueryException",
  "file": "vendor\\laravel\\framework\\src\\Illuminate\\Database\\Connection.php",
  "line": 824
}
```

**Code Path (excerpt)**

* `app/Http/Controllers/AuthController.php` → `register()`
* `User::create([... 'phone_number' => ...])`
* MySQL throws `42S22` because `users.phone_number` does not exist.

---

## 🔍 Root Cause

The controller validates and writes a `phone_number` (and optionally `is_admin`) to the `users` table, but the table schema does not include those columns yet.

---

## ✅ Fix Options

### Option A — Add the Missing Columns (recommended if you need phone numbers / roles)

1. Create a migration:

```bash
php artisan make:migration add_phone_number_and_is_admin_to_users_table --table=users
```

2. Implement the migration:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::table('users', function (Blueprint $table) {
            $table->string('phone_number', 20)->nullable()->after('email');
            $table->boolean('is_admin')->default(false)->after('password'); // optional role flag
        });
    }

    public function down(): void {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn(['phone_number', 'is_admin']);
        });
    }
};
```

3. Run migrations:

```bash
php artisan migrate
```

4. Allow mass assignment in the model (if using guarded/fillable):

```php
// app/Models/User.php
protected $fillable = ['name', 'email', 'password', 'phone_number', 'is_admin'];

protected $casts = [
    'is_admin' => 'boolean',
    'email_verified_at' => 'datetime',
];
```

> After this, the original request payload will work (and `is_admin` will persist if you include it).

---

### Option B — Stop Writing `phone_number` (if you don’t need it yet)

Remove `phone_number` everywhere in the controller:

**Before**

```php
$data = $request->validate([
    'name'         => ['required','string','max:255'],
    'email'        => ['required','email','unique:users,email'],
    'password'     => ['required','string','min:8','confirmed'],
    'phone_number' => ['nullable','string','max:20'],
]);

$user = User::create([
    'name'         => $data['name'],
    'email'        => $data['email'],
    'password'     => Hash::make($data['password']),
    'phone_number' => $data['phone_number'] ?? null,
]);
```

**After**

```php
$data = $request->validate([
    'name'     => ['required','string','max:255'],
    'email'    => ['required','email','unique:users,email'],
    'password' => ['required','string','min:8','confirmed'],
]);

$user = User::create([
    'name'     => $data['name'],
    'email'    => $data['email'],
    'password' => Hash::make($data['password']),
]);
```

> If you keep `is_admin` in the request but don’t persist it, it will be ignored (or validate/deny it explicitly).

---

## 🔐 Bonus: Correct `login()` Usage with Sanctum

`Auth::attempt()` should only receive `email` and `password`. Don’t pass `device_name` into the attempt:

```php
public function login(Request $request)
{
    $validated = $request->validate([
        'email'       => ['required','email'],
        'password'    => ['required','string'],
        'device_name' => ['required','string'],
    ]);

    if (! \Illuminate\Support\Facades\Auth::attempt($request->only('email','password'))) {
        throw \Illuminate\Validation\ValidationException::withMessages([
            'email' => 'Credentials are invalid.',
        ]);
    }

    /** @var \App\Models\User $user */
    $user = $request->user();

    // Revoke previous token on the same device (optional best-practice)
    $user->tokens()->where('name', $validated['device_name'])->delete();

    // Granular abilities for mobile/web
    $abilities = ['attendance:create', 'attendance:view', 'profile:read'];

    $token = $user->createToken($validated['device_name'], $abilities)->plainTextToken;

    return response()->json([
        'token' => $token,
        'token_type' => 'Bearer',
        'expires_in_minutes' => (int) config('sanctum.expiration'),
        'user' => $user,
    ]);
}
```

---

## 🧪 Test Flow

1. **Register**

```bash
curl -X POST http://127.0.0.1:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name":"test",
    "email":"test@example.com",
    "password":"password",
    "password_confirmation":"password",
    "device_name":"web",
    "is_admin": true
  }'
```

2. **Use the token**

```bash
curl http://127.0.0.1:8000/api/auth/me \
  -H "Authorization: Bearer <PASTE_TOKEN_HERE>"
```

Expected: user JSON payload.

---

## 🧰 Environment Notes

* **Stack**: Laravel, Sanctum, MySQL (Laragon dev)
* If running locally without HTTPS and using SPA cookies/CSRF:

  ```
  SESSION_SECURE_COOKIE=false
  APP_URL=http://localhost
  ```
* Ensure migrations are in the correct order for foreign keys (see other README in this repo if applicable).

---

## 📝 Summary

* Error `SQLSTATE[42S22] Unknown column 'phone_number'` occurs because the `users` table lacks that column.
* **Fix A:** Add the column(s) via migration (recommended if you actually need them).
* **Fix B:** Remove the field from validation and `User::create()` until you add schema support.
* Validate login flow and token creation with Sanctum using only `email` + `password` for `Auth::attempt()`.

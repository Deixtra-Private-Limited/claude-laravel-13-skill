# Laravel 12 → 13 Upgrade Guide

Estimated Upgrade Time: ~10 Minutes

Laravel 13 has minimal breaking changes. Most apps can upgrade without changing much code.

---

## Automated Upgrade

Install Laravel Boost ^2.0 and use the `/upgrade-laravel-v13` slash command in Claude Code, Cursor, OpenCode, Gemini, or VS Code.

Or use [Laravel Shift](https://laravelshift.com) for automated upgrades.

---

## High Impact Changes

### Update Dependencies (`composer.json`)
```json
{
    "require": {
        "laravel/framework": "^13.0"
    },
    "require-dev": {
        "laravel/boost": "^2.0",
        "laravel/tinker": "^3.0",
        "phpunit/phpunit": "^12.0",
        "pestphp/pest": "^4.0"
    }
}
```

### Update Laravel Installer
```bash
composer global update laravel/installer
```

### PreventRequestForgery Middleware
CSRF protection middleware has been enhanced and renamed to `PreventRequestForgery` with origin-aware request verification. This is backward compatible but may affect apps that override CSRF behavior.

---

## Medium Impact Changes

### Cache `serializable_classes` Configuration
Default app cache config now includes `serializable_classes` set to `false`. This prevents arbitrary PHP object unserialization (protects against gadget chain attacks if APP_KEY leaks).

If you store PHP objects in cache, explicitly allow-list them:
```php
// config/cache.php
'serializable_classes' => [
    App\Data\CachedDashboardStats::class,
    App\Support\CachedPricingSnapshot::class,
],
```

---

## Low Impact Changes

### Cache Prefixes and Session Cookie Names
Default prefixes changed from underscores to hyphens:
```
// Before: laravel_cache_, laravel_database_, laravel_session
// After:  laravel-cache-, laravel-database-, laravel_session (snake_case)
```

To keep old behavior, explicitly set `CACHE_PREFIX`, `REDIS_PREFIX`, `SESSION_COOKIE` in `.env`.

### Collection Model Serialization
Eager-loaded relations now restored during unserialization of Eloquent collections.

### Container::call and Nullable Class Defaults
`Container::call` now respects nullable class parameter defaults when no binding exists:
```php
$container->call(function (?Carbon $date = null) {
    return $date;
});
// Laravel 12: Carbon instance
// Laravel 13: null
```

### Domain Route Registration Precedence
Domain-specific routes now have stricter precedence. Register them before catch-all routes.

### JobAttempted Event Exception Payload
The `JobAttempted` event now includes the exception payload when a job fails.

### Manager `extend` Callback Binding
`Manager::extend()` callback `$this` binding behavior has changed.

### MySQL DELETE with JOIN, ORDER BY, LIMIT
Laravel now compiles full `DELETE ... JOIN` including `ORDER BY` and `LIMIT` for MySQL. If your queries relied on these being silently ignored, they may now throw `QueryException`.

### Pagination Bootstrap View Names
Bootstrap pagination view names have been updated.

### Polymorphic Pivot Table Name Generation
Now uses consistent alphabetical ordering for generated pivot table names.

### QueueBusy Event Property Rename
The `QueueBusy` event has renamed a property.

### Str Factories Reset Between Tests
`Str` factories now reset between tests.

---

## New Contracts Methods

If you implement any of these contracts, add the new methods:

- `Illuminate\Contracts\Cache\Store` → add `touch($key, $seconds)`
- `Illuminate\Contracts\Bus\Dispatcher` → add `dispatchAfterResponse($command, $handler = null)`
- `Illuminate\Contracts\Routing\ResponseFactory` → add `eventStream()`
- `Illuminate\Contracts\Auth\MustVerifyEmail` → add `markEmailAsUnverified()`

---

## New Model Booting Restriction

Creating a model instance inside `boot()` or trait `boot*()` methods now throws `LogicException`:
```php
// This is no longer allowed:
protected static function boot()
{
    parent::boot();
    (new static())->getTable(); // Throws LogicException
}
```

Move such logic outside the boot cycle.

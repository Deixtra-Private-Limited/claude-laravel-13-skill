---
name: laravel-13-expert
description: Comprehensive Laravel 13.x expert skill covering the entire framework — installation, configuration, routing, controllers, middleware, Blade views, Eloquent ORM, migrations, queues, jobs, events, mail, notifications, caching, Redis, scheduling, service container, service providers, authentication, authorization, AI SDK (agents, tools, structured output, streaming, embeddings, images, audio), MCP servers, Laravel Boost, JSON:API resources, vector/semantic search, PHP attributes (#[Middleware], #[Authorize], #[Tries], etc.), queue routing, Cache::touch(), and all Laravel 13 breaking changes and upgrade patterns. Trigger this skill whenever the user mentions Laravel 13, Laravel AI SDK, Laravel MCP, Laravel Boost, Laravel agents, Laravel vector search, Laravel embeddings, whereVectorSimilarTo, JSON:API resources, PHP attributes in Laravel, queue routing, PreventRequestForgery, Cache::touch, or any Laravel backend development task. Also trigger for upgrading from Laravel 12 to 13, or any question about what's new in Laravel 13. This skill supersedes the older laravel-backend-expert skill for Laravel 13 projects.
---

# Laravel 13 Expert

Complete reference for building production-grade Laravel 13.x applications. Laravel 13 was released March 17, 2026 and requires PHP 8.3+.

## When to read reference files

This skill is organized with a main SKILL.md (this file) covering core concepts, and reference files for deep-dive topics. Read the appropriate reference file when the task involves:

- **AI SDK, agents, tools, embeddings, images, audio, MCP** → Read `references/ai-features.md`
- **Upgrading from Laravel 12 to 13** → Read `references/upgrade-guide.md`

---

## What's New in Laravel 13

Laravel 13 focuses on AI-native workflows, stronger defaults, and more expressive APIs. Key additions:

### First-Party AI SDK (`laravel/ai`)
A unified API for text generation, tool-calling agents, embeddings, audio, images, and vector-store integrations. Provider-agnostic (OpenAI, Anthropic, Gemini, xAI, Mistral, Ollama, etc.).

```php
use App\Ai\Agents\SalesCoach;

$response = SalesCoach::make()->prompt('Analyze this sales transcript...');
return (string) $response;
```

### Laravel MCP (`laravel/mcp`)
Build Model Context Protocol servers so AI clients can interact with your Laravel app. Define servers, tools, resources, and prompts.

```php
use Laravel\Mcp\Facades\Mcp;
Mcp::web('/mcp/weather', WeatherServer::class);
```

### Laravel Boost (`laravel/boost`)
MCP server that gives AI agents Laravel-specific context, 15+ tools, 17,000+ pieces of vectorized docs, and AI guidelines.

```bash
composer require laravel/boost --dev
php artisan boost:install
```

### JSON:API Resources
First-party JSON:API compliant resource serialization with relationship inclusion, sparse fieldsets, and links.

### Queue Routing
Central routing rules for jobs by class:

```php
Queue::route(ProcessPodcast::class, connection: 'redis', queue: 'podcasts');
```

### Expanded PHP Attributes
Declarative controller and job configuration:

```php
#[Middleware('auth')]
class CommentController
{
    #[Middleware('subscribed')]
    #[Authorize('create', [Comment::class, 'post'])]
    public function store(Post $post) { /* ... */ }
}
```

Job attributes: `#[Tries(3)]`, `#[Backoff(5)]`, `#[Timeout(120)]`, `#[FailOnTimeout]`

### Cache TTL Extension
```php
Cache::touch('key', seconds: 3600); // Extend TTL without re-storing
```

### Semantic / Vector Search
Native vector query support with pgvector:

```php
$documents = DB::table('documents')
    ->whereVectorSimilarTo('embedding', 'Best wineries in Napa Valley')
    ->limit(10)
    ->get();
```

### PreventRequestForgery Middleware
Enhanced CSRF protection with origin-aware request verification.

### Embeddings via Str helper
```php
use Illuminate\Support\Str;
$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings();
```

---

## Installation & Setup

### Requirements
- PHP 8.3 - 8.5
- Composer
- Node/NPM or Bun

### Quick Install
```bash
# Install PHP + Composer + Laravel installer
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"  # macOS
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)" # Linux

# Create new app
laravel new example-app
cd example-app
npm install && npm run build
composer run dev
```

The `composer run dev` command starts the local dev server, queue worker, and Vite dev server. App accessible at http://localhost:8000.

### Default Database
Laravel 13 defaults to SQLite. The installer creates `database/database.sqlite` and runs migrations automatically. To switch to MySQL:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

Then run `php artisan migrate`.

### Laravel Herd
Native dev environment for macOS/Windows. Includes PHP, Nginx, Composer. Parked directories at `~/Herd` auto-served on `.test` domain.

---

## Configuration

All config in `config/` directory. Key patterns:

- Environment-based via `.env` file (never commit to source control)
- Access with `env('KEY', 'default')` in config files, `config('app.name')` elsewhere
- Cache config in production: `php artisan config:cache`

### Laravel 13 Cache Changes
Default cache/Redis prefixes now use hyphens instead of underscores:
```
// Old: laravel_cache_
// New: laravel-cache-
```

If relying on generated defaults, explicitly set `CACHE_PREFIX`, `REDIS_PREFIX`, `SESSION_COOKIE` in `.env`.

### New: `serializable_classes` Cache Config
Default is `false` to prevent deserialization attacks. If storing PHP objects in cache, allow-list them:

```php
'serializable_classes' => [
    App\Data\CachedStats::class,
],
```

---

## Directory Structure

Standard Laravel 13 structure:

```
app/
├── Ai/                    # NEW in 13 - AI agents and tools
│   ├── Agents/
│   └── Tools/
├── Console/Commands/
├── Events/
├── Http/
│   ├── Controllers/
│   ├── Middleware/
│   └── Requests/
├── Jobs/
├── Listeners/
├── Mail/
├── Mcp/                   # NEW in 13 - MCP servers
│   └── Servers/
├── Models/
├── Notifications/
├── Policies/
├── Providers/
├── Repositories/
├── Services/
└── View/Components/
routes/
├── ai.php                 # NEW in 13 - MCP server routes
├── web.php
├── console.php
└── channels.php
config/
├── ai.php                 # NEW in 13 - AI provider config
└── ...
```

---

## Routing

```php
// Basic routes
Route::get('/users', [UserController::class, 'index']);
Route::post('/users', [UserController::class, 'store']);

// Resource routes
Route::resource('posts', PostController::class);
Route::apiResource('posts', PostController::class);

// Route groups
Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
});

// Route model binding
Route::get('/posts/{post}', fn (Post $post) => view('posts.show', compact('post')));
```

### Laravel 13: Domain Route Registration Precedence
Domain routes now have stricter precedence. Register domain-specific routes before catch-all routes.

---

## Controllers

### With PHP Attributes (Laravel 13)
```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Routing\Attributes\Controllers\Authorize;
use Illuminate\Routing\Attributes\Controllers\Middleware;

#[Middleware('auth')]
class PostController
{
    public function index()
    {
        $posts = Post::latest()->paginate(15);
        return view('posts.index', compact('posts'));
    }

    #[Middleware('subscribed')]
    #[Authorize('create', Post::class)]
    public function store(StorePostRequest $request)
    {
        $post = $request->user()->posts()->create($request->validated());
        return redirect()->route('posts.show', $post);
    }

    #[Authorize('update', 'post')]
    public function update(UpdatePostRequest $request, Post $post)
    {
        $post->update($request->validated());
        return redirect()->route('posts.show', $post);
    }
}
```

### Form Requests
```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'content' => ['required', 'string'],
            'category_id' => ['required', 'exists:categories,id'],
            'tags' => ['nullable', 'array'],
            'tags.*' => ['exists:tags,id'],
        ];
    }
}
```

---

## Eloquent ORM

### Model Basics
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Post extends Model
{
    use HasFactory, SoftDeletes;

    protected $fillable = ['title', 'slug', 'content', 'category_id', 'published_at'];

    protected function casts(): array
    {
        return [
            'published_at' => 'datetime',
            'metadata' => 'array',
        ];
    }

    // Relationships
    public function author()
    {
        return $this->belongsTo(User::class, 'user_id');
    }

    public function category()
    {
        return $this->belongsTo(Category::class);
    }

    public function tags()
    {
        return $this->belongsToMany(Tag::class);
    }

    public function comments()
    {
        return $this->hasMany(Comment::class);
    }

    // Scopes
    public function scopePublished($query)
    {
        return $query->whereNotNull('published_at')->where('published_at', '<=', now());
    }
}
```

### Laravel 13 Eloquent Changes
- **Model booting**: Creating a model instance inside `boot()` now throws `LogicException`
- **Polymorphic pivot table names**: Generation now uses alphabetical ordering consistently
- **Collection serialization**: Eager-loaded relations now restored during unserialization

---

## Migrations

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->foreignId('category_id')->constrained();
            $table->string('title');
            $table->string('slug')->unique();
            $table->text('content');
            $table->json('metadata')->nullable();
            $table->timestamp('published_at')->nullable();
            $table->softDeletes();
            $table->timestamps();

            $table->index(['published_at', 'created_at']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('posts');
    }
};
```

### Vector Column (Laravel 13 + pgvector)
```php
Schema::create('documents', function (Blueprint $table) {
    $table->id();
    $table->text('content');
    $table->vector('embedding', dimensions: 1536); // pgvector column
    $table->timestamps();
});
```

---

## Queues & Jobs

### Laravel 13: PHP Attributes for Jobs
```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\Attributes\Backoff;
use Illuminate\Queue\Attributes\FailOnTimeout;
use Illuminate\Queue\Attributes\Timeout;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

#[Tries(3)]
#[Timeout(120)]
#[Backoff(5)]
#[FailOnTimeout]
class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(public Podcast $podcast) {}

    public function handle(): void
    {
        // Process the podcast...
    }
}
```

### Queue Routing (Laravel 13)
```php
// In AppServiceProvider::boot()
use Illuminate\Support\Facades\Queue;

Queue::route(ProcessPodcast::class, connection: 'redis', queue: 'podcasts');
Queue::route(SendNewsletter::class, connection: 'sqs', queue: 'emails');
```

### Dispatching
```php
ProcessPodcast::dispatch($podcast);
ProcessPodcast::dispatch($podcast)->onQueue('podcasts');
ProcessPodcast::dispatch($podcast)->delay(now()->addMinutes(10));

// Job chaining
Bus::chain([
    new ProcessPodcast($podcast),
    new OptimizePodcast($podcast),
    new ReleasePodcast($podcast),
])->dispatch();

// Job batching
Bus::batch([
    new ProcessPodcast($podcast1),
    new ProcessPodcast($podcast2),
])->then(function (Batch $batch) {
    // All jobs completed...
})->catch(function (Batch $batch, Throwable $e) {
    // First batch job failure...
})->dispatch();
```

---

## Caching

### Cache::touch (Laravel 13)
```php
use Illuminate\Support\Facades\Cache;

// Extend TTL without re-storing the value
Cache::touch('user:123:profile', seconds: 3600);

// Standard cache operations
Cache::put('key', 'value', now()->addMinutes(10));
Cache::remember('users', 3600, fn () => User::all());
Cache::forever('config', $data);
Cache::forget('key');

// Atomic locks
Cache::lock('processing')->get(function () {
    // Process exclusively...
});
```

---

## Events & Listeners

```php
// Define event
class OrderShipped
{
    public function __construct(public Order $order) {}
}

// Define listener
class SendShipmentNotification
{
    public function handle(OrderShipped $event): void
    {
        $event->order->user->notify(new OrderShippedNotification($event->order));
    }
}

// Register in EventServiceProvider or auto-discovery
// Dispatch
event(new OrderShipped($order));
```

---

## Mail & Notifications

```php
// Mailable
class WelcomeEmail extends Mailable
{
    public function __construct(public User $user) {}

    public function content(): Content
    {
        return new Content(markdown: 'emails.welcome');
    }
}

// Send
Mail::to($user)->send(new WelcomeEmail($user));
Mail::to($user)->queue(new WelcomeEmail($user));

// Notification
$user->notify(new InvoicePaid($invoice));
Notification::send($users, new InvoicePaid($invoice));
```

---

## Authentication & Authorization

### Guards & Providers
Standard config in `config/auth.php`. Laravel 13 supports multi-guard setups for admin panels.

### Policies with Attributes (Laravel 13)
```php
// In controller - using #[Authorize] attribute
#[Authorize('update', 'post')]
public function update(Request $request, Post $post) { /* ... */ }

// In Blade
@can('update', $post)
    <a href="{{ route('posts.edit', $post) }}">Edit</a>
@endcan
```

---

## CSRF Protection

### PreventRequestForgery (Laravel 13)
Enhanced middleware that adds origin-aware request verification alongside token-based CSRF. This is now the default.

---

## Testing

```php
// Feature test
class PostTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_create_post(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)
            ->post('/posts', [
                'title' => 'Test Post',
                'content' => 'Content here',
            ]);

        $response->assertRedirect();
        $this->assertDatabaseHas('posts', ['title' => 'Test Post']);
    }
}
```

### Testing AI SDK (Laravel 13)
```php
use Laravel\Ai\Facades\Ai;

Ai::fake([
    SalesCoach::class => 'Great sales technique!',
]);

$response = SalesCoach::make()->prompt('Analyze this...');
$this->assertEquals('Great sales technique!', (string) $response);
```

---

## Artisan Commands (Updated for Laravel 13)

```bash
# New in Laravel 13
php artisan make:agent SalesCoach           # Create AI agent
php artisan make:agent SalesCoach --structured  # With structured output
php artisan make:mcp-server WeatherServer   # Create MCP server
php artisan boost:install                   # Install Laravel Boost
php artisan vendor:publish --tag=ai-routes  # Publish MCP routes

# Standard commands
php artisan make:model Post -mfsc
php artisan make:controller PostController --resource
php artisan make:middleware CheckRole
php artisan make:policy PostPolicy --model=Post
php artisan make:job ProcessPodcast
php artisan make:event OrderShipped
php artisan make:listener SendNotification --event=OrderShipped
php artisan make:mail WelcomeEmail --markdown=emails.welcome
php artisan make:notification InvoicePaid
php artisan make:command SendReport

# Database
php artisan migrate
php artisan migrate:fresh --seed
php artisan db:seed
php artisan db:show

# Queue
php artisan queue:work
php artisan queue:restart
php artisan queue:retry all

# Production
php artisan optimize
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

---

## Deployment Checklist

```bash
composer install --optimize-autoloader --no-dev
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan optimize
php artisan migrate --force
```

Ensure `APP_ENV=production`, `APP_DEBUG=false` in `.env`.

---

## Support Policy

| Version | PHP | Release | Bug Fixes Until | Security Until |
|---------|-----|---------|-----------------|----------------|
| 11 | 8.2-8.4 | Mar 2024 | Sep 2025 | Mar 2026 |
| 12 | 8.2-8.5 | Feb 2025 | Aug 2026 | Feb 2027 |
| 13 | 8.3-8.5 | Mar 2026 | Q3 2027 | Mar 2028 |

---

## IDE Support

Recommended editors for Laravel 13:
- **VS Code / Cursor** with official Laravel VS Code Extension — syntax highlighting, Eloquent autocomplete, Artisan integration, Inertia.js support
- **PhpStorm** — built-in Laravel framework support, Blade templates, smart autocompletion
- **Firebase Studio** — cloud-based, zero-setup Laravel dev environment

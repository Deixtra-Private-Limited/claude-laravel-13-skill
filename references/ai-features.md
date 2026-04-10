# Laravel 13 AI Features Reference

Deep-dive reference for the Laravel AI SDK, MCP servers, and Laravel Boost.

---

## AI SDK (`laravel/ai`)

### Installation

```bash
composer require laravel/ai
php artisan vendor:publish --provider="Laravel\Ai\AiServiceProvider"
php artisan migrate  # Creates agent_conversations and agent_conversation_messages tables
```

### Configuration (`config/ai.php`)

Set API keys in `.env`:
```env
ANTHROPIC_API_KEY=
OPENAI_API_KEY=
GEMINI_API_KEY=
XAI_API_KEY=
MISTRAL_API_KEY=
OLLAMA_API_KEY=
COHERE_API_KEY=
ELEVENLABS_API_KEY=
JINA_API_KEY=
VOYAGEAI_API_KEY=
```

### Custom Base URLs
Route requests through proxies or gateways:
```php
'providers' => [
    'openai' => [
        'driver' => 'openai',
        'key' => env('OPENAI_API_KEY'),
        'url' => env('OPENAI_BASE_URL'),
    ],
],
```

### Provider Support Matrix

| Feature | Providers |
|---------|-----------|
| Text | OpenAI, Anthropic, Gemini, Azure, Groq, xAI, DeepSeek, Mistral, Ollama |
| Images | OpenAI, Gemini, xAI |
| TTS | OpenAI, ElevenLabs |
| STT | OpenAI, ElevenLabs, Mistral |
| Embeddings | OpenAI, Gemini, Azure, Cohere, Mistral, Jina, VoyageAI |
| Reranking | Cohere, Jina |
| Files | OpenAI, Anthropic, Gemini |

### Provider Enum
```php
use Laravel\Ai\Enums\Lab;

Lab::Anthropic;
Lab::OpenAI;
Lab::Gemini;
// etc.
```

---

## Agents

Agents are PHP classes encapsulating instructions, conversation context, tools, and output schema.

### Creating Agents
```bash
php artisan make:agent SalesCoach
php artisan make:agent SalesCoach --structured  # With structured output schema
```

### Agent Structure
```php
<?php

namespace App\Ai\Agents;

use App\Ai\Tools\RetrievePreviousTranscripts;
use App\Models\User;
use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\Conversational;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Messages\Message;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, Conversational, HasTools, HasStructuredOutput
{
    use Promptable;

    public function __construct(public User $user) {}

    public function instructions(): string
    {
        return 'You are a sales coach...';
    }

    public function messages(): iterable
    {
        return $this->user->history()
            ->latest()->limit(50)->get()->reverse()
            ->map(fn ($m) => new Message($m->role, $m->content))
            ->all();
    }

    public function tools(): iterable
    {
        return [new RetrievePreviousTranscripts];
    }

    public function schema(JsonSchema $schema): array
    {
        return [
            'feedback' => $schema->string()->required(),
            'score' => $schema->integer()->min(1)->max(10)->required(),
        ];
    }
}
```

### Prompting
```php
// Basic prompt
$response = (new SalesCoach)->prompt('Analyze this transcript...');
return (string) $response;

// With dependency injection via make()
$agent = SalesCoach::make(user: $user);

// Override provider/model
$response = (new SalesCoach)->prompt(
    'Analyze this transcript...',
    provider: Lab::Anthropic,
    model: 'claude-haiku-4-5-20251001',
    timeout: 120,
);

// Structured output access
$response = (new SalesCoach)->prompt('Analyze...');
$score = $response['score'];
$feedback = $response['feedback'];
```

### Conversation Memory (RemembersConversations)
```php
use Laravel\Ai\Concerns\RemembersConversations;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\Conversational;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, Conversational
{
    use Promptable, RemembersConversations;

    public function instructions(): string
    {
        return 'You are a sales coach...';
    }
}

// Start conversation
$response = (new SalesCoach)->forUser($user)->prompt('Hello!');
$conversationId = $response->conversationId;

// Continue existing conversation
$response = (new SalesCoach)
    ->continue($conversationId, as: $user)
    ->prompt('Tell me more about that.');
```

### Attachments
```php
use Laravel\Ai\Files;

$response = (new SalesCoach)->prompt(
    'Analyze the attached transcript...',
    attachments: [
        Files\Document::fromStorage('transcript.pdf'),
        Files\Document::fromPath('/path/to/file.md'),
        $request->file('transcript'), // UploadedFile
    ]
);

// Images
$response = (new ImageAnalyzer)->prompt(
    'What is in this image?',
    attachments: [
        Files\Image::fromStorage('photo.jpg'),
        Files\Image::fromPath('/path/to/photo.jpg'),
        $request->file('photo'),
    ]
);
```

### Streaming
```php
// Return SSE stream from route
Route::get('/coach', function () {
    return (new SalesCoach)->stream('Analyze this transcript...');
});

// With callback on completion
return (new SalesCoach)
    ->stream('Analyze this transcript...')
    ->then(function (StreamedAgentResponse $response) {
        // $response->text, $response->events, $response->usage
    });
```

### Broadcasting
Agents can broadcast streaming responses to a channel for real-time frontend updates via WebSockets/Reverb.

### Queueing
Agents can be dispatched as queued jobs for background processing.

### Agent Tools
Define tools as PHP classes that agents can invoke during processing. Tools have input schemas, output schemas, and validation.

### Agent Middleware
Custom middleware can wrap agent interactions for logging, rate limiting, etc.

### Anonymous Agents
Quick one-off agents without a dedicated class.

---

## Images

```php
use Laravel\Ai\Image;

$image = Image::of('A donut on the kitchen counter')->generate();
$rawContent = (string) $image;

// Save to disk
Storage::put('images/donut.png', (string) $image);
```

## Audio (TTS)

```php
use Laravel\Ai\Audio;

$audio = Audio::of('I love coding with Laravel.')->generate();
$rawContent = (string) $audio;
```

## Transcription (STT)

Transcribe audio files to text using OpenAI, ElevenLabs, or Mistral.

## Embeddings

```php
use Illuminate\Support\Str;

// Generate embeddings from string
$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings();

// Querying with vector similarity (pgvector)
$results = DB::table('documents')
    ->whereVectorSimilarTo('embedding', 'Best wineries in Napa Valley')
    ->limit(10)
    ->get();
```

### Caching Embeddings
Embeddings can be cached to avoid regenerating them for identical inputs.

## Reranking

Reorder search results by semantic relevance using Cohere or Jina.

## Files & Vector Stores

Upload files to AI providers and manage vector stores for retrieval-augmented generation (RAG).

---

## AI SDK Testing

```php
use Laravel\Ai\Facades\Ai;

// Fake agent responses
Ai::fake([
    SalesCoach::class => 'Great sales technique!',
]);

$response = SalesCoach::make()->prompt('Analyze this...');
$this->assertEquals('Great sales technique!', (string) $response);

// Fake images
Ai::fakeImages([
    'A donut*' => '/path/to/fake-image.png',
]);

// Fake audio, transcriptions, embeddings, reranking, files, vector stores
// all follow the same Ai::fake*() pattern
```

### Failover
Configure failover providers so requests automatically retry on different providers if one fails.

---

## Laravel MCP (`laravel/mcp`)

Build MCP servers that expose your Laravel app to AI clients.

### Installation
```bash
composer require laravel/mcp
php artisan vendor:publish --tag=ai-routes  # Creates routes/ai.php
```

### Creating Servers
```bash
php artisan make:mcp-server WeatherServer
```

Generated class in `app/Mcp/Servers/`:
```php
<?php

namespace App\Mcp\Servers;

use Laravel\Mcp\Server;
use Laravel\Mcp\Server\Attributes\Instructions;
use Laravel\Mcp\Server\Attributes\Name;
use Laravel\Mcp\Server\Attributes\Version;

#[Name('Weather Server')]
#[Version('1.0.0')]
#[Instructions('This server provides weather information.')]
class WeatherServer extends Server
{
    protected array $tools = [
        // GetCurrentWeatherTool::class,
    ];

    protected array $resources = [
        // WeatherGuidelinesResource::class,
    ];

    protected array $prompts = [
        // DescribeWeatherPrompt::class,
    ];
}
```

### Server Registration (`routes/ai.php`)
```php
use App\Mcp\Servers\WeatherServer;
use Laravel\Mcp\Facades\Mcp;

// Web server (HTTP POST, remote AI clients)
Mcp::web('/mcp/weather', WeatherServer::class);

// Local server (command-line, local AI tools)
Mcp::local(WeatherServer::class);
```

### MCP Tools
Tools are the primary way AI clients interact with your app:
```php
// Create a tool
php artisan make:mcp-tool GetCurrentWeather

// Tool class has:
// - Input schema (validated arguments)
// - Output schema
// - handle() method with dependency injection
// - Annotations for behavior hints
// - Conditional registration
```

### MCP Resources
Static or template-based data that AI clients can read (docs, configs, etc.).

### MCP Prompts
Pre-built prompts with arguments that AI clients can invoke.

### Authentication
MCP servers support OAuth 2.1 and Laravel Sanctum for authentication.

### Authorization
Use Laravel's authorization layer (policies, gates) to control access to MCP tools/resources.

### Testing MCP Servers
Use the MCP Inspector tool or write unit tests for tools/resources/prompts.

---

## Laravel Boost (`laravel/boost`)

### Installation
```bash
composer require laravel/boost --dev
php artisan boost:install  # Auto-detects IDE and AI agents
```

Works with Laravel 10, 11, 12, and 13 (PHP 8.1+).

### What It Provides

**15+ MCP Tools:**
- Application introspection (PHP/Laravel versions, packages, config)
- Database inspection and read-only queries
- Route listing with middleware and parameters
- Artisan command discovery
- Log analysis
- Browser logs
- Tinker integration
- Documentation search (17,000+ pieces, semantic search)

**AI Guidelines:**
Composable, version-aware guidelines for Laravel and 16+ packages:
- Livewire 2.x, 3.x, 4.x
- Inertia.js (React, Svelte, Vue)
- Tailwind CSS 3.x, 4.x
- Filament 3.x, 4.x
- PHPUnit, Pest PHP, Pint
- And more

**Agent Skills:**
On-demand knowledge modules activated when needed. Available at agentskills.io.

### Custom AI Guidelines
Add `.blade.php` or `.md` files to `.ai/guidelines/*` directory — automatically included with Boost's guidelines.

### Updating
```bash
php artisan boost:update  # Regenerates guidelines and configs
```

### Agent Setup
- **Cursor**: Open MCP Settings → toggle `laravel-boost`
- **Claude Code**: Automatic via `.mcp.json`
- **Other IDEs**: Supported via published config files

### Upgrading with Boost
Use `/upgrade-laravel-v13` slash command in Claude Code/Cursor with Boost ^2.0 installed.

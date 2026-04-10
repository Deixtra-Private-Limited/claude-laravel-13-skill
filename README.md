# claude-laravel-13-skill
# Laravel 13 Expert Skill

A comprehensive Claude Skill that turns Claude into a Laravel 13.x expert — covering the entire framework from routing and Eloquent to the new AI SDK, MCP servers, and Laravel Boost.

Built for developers who want Claude to generate production-ready Laravel 13 code that follows modern best practices, uses the latest PHP attributes, and leverages Laravel's newest features out of the box.

## What This Skill Covers

- **Core Framework** — Routing, controllers, middleware, Blade, service container, service providers
- **Database & ORM** — Eloquent, migrations, relationships, query builder, vector/semantic search
- **Background Work** — Queues, jobs, events, listeners, scheduling, notifications, mail
- **Performance** — Caching, Redis, optimization patterns
- **Security** — Authentication, authorization, gates, policies
- **Laravel 13 New Features** — AI SDK (agents, tools, structured output, streaming, embeddings, images, audio), MCP servers, Laravel Boost, JSON:API resources, PHP attributes like `#[Middleware]`
- **File Handling** — Uploads, storage, S3 integration

## Installation

### For Claude Code / Claude Desktop

1. Clone this repository:
```bash
   git clone https://github.com/YOUR_USERNAME/laravel-13-expert-skill.git
```

2. Copy the skill folder to your Claude skills directory:
```bash
   cp -r laravel-13-expert-skill ~/.claude/skills/
```

3. Restart Claude. The skill will auto-load whenever you ask about Laravel.

### For Claude.ai (Web)

Upload the `SKILL.md` file and any referenced assets through the Skills section in your Claude.ai settings.

## Usage

Just ask Claude anything Laravel-related and the skill will activate automatically:

- *"Build me a Laravel 13 API with JSON:API resources for a blog"*
- *"Create an AI agent using Laravel's AI SDK that summarizes PDFs"*
- *"Set up an MCP server in Laravel 13"*
- *"Write a queued job that processes CSV uploads with chunking"*
- *"Generate an Eloquent model with vector search for product recommendations"*

## Why Laravel 13?

Laravel 13 introduces major additions that older training data doesn't cover well:

- First-party **AI SDK** for agents, tools, and structured output
- Native **MCP server** support
- **Laravel Boost** for faster bootstrapping
- New **PHP attribute-based** routing and middleware
- Improved JSON:API resource handling

This skill makes sure Claude generates code for *these* features instead of defaulting to Laravel 10/11 patterns.

## Built By

Made by [Abdul](https://deixtra.com) at **Deixtra (Pvt) Ltd** — a digital solutions company building POS, LMS, e-commerce, and inventory systems for clients across Pakistan, UAE, and GCC markets.

## Contributing

PRs welcome! If you find a Laravel 13 feature that's missing or a pattern that could be improved, open an issue or submit a pull request.

## License

MIT — free to use, modify, and distribute.

---

⭐ If this skill saves you time, give it a star on GitHub!

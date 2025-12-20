# Options Pattern (PHP)

## What it is

The **Options Pattern** is a design approach to manage application configuration by grouping related settings into a **dedicated configuration object** (an â€œoptionsâ€ object) instead of spreading environment variable reads or string-based configuration keys across the codebase.

In PHP, this typically means:
- defining a class that represents a coherent configuration domain (SMTP, Redis, Payments, Feature Flags, etc.)
- loading configuration values once (usually at bootstrap)
- passing (or injecting) that object into the services that depend on it

The intent is pragmatic:
- make configuration explicit
- remove hidden runtime dependencies
- fail fast if configuration is invalid
- improve testability and readability

Configuration becomes a first-class dependency, not a side effect.

---

## How it works

The Options Pattern usually follows this flow:

1. Define an Options class
   - One class per logical configuration group
   - Explicit properties (no magic strings)

2. Build the Options object
   - Load from env variables, config files, or a secrets manager
   - Validate early (fail fast)

3. Inject or pass the Options object
   - Services receive it via constructor
   - Services do not call `getenv()` directly

4. Use configuration through the object
   - Consistent access
   - Self-documented dependencies

This fits naturally with Dependency Injection and Clean Architecture: configuration stays at the boundary (bootstrap/composition root), while services remain pure and testable.

---

## Applications

The Options Pattern is especially useful for:

- External services configuration
  - SMTP, Stripe, PayPal, OpenAI, S3
- Infrastructure components
  - Redis, RabbitMQ, Elasticsearch
- Feature flags and toggles
- Retry / timeout policies
- Multi-environment setups (dev / staging / production)

Use it when configuration:
- has multiple parameters
- must be consistent
- needs validation
- is reused across multiple services

---

## Example (PHP)

### Options class

```php
<?php

declare(strict_types=1);

final class SmtpOptions
{
    public function __construct(
        public readonly string $host,
        public readonly int $port,
        public readonly string $username,
        public readonly string $password,
        public readonly bool $useTls,
    ) {}
}
```

### Factory / loader (build once, validate early)

```php
<?php

declare(strict_types=1);

final class SmtpOptionsFactory
{
    public static function fromEnv(): SmtpOptions
    {
        $host = getenv('SMTP_HOST') ?: '';
        $port = (int) (getenv('SMTP_PORT') ?: 0);
        $username = getenv('SMTP_USER') ?: '';
        $password = getenv('SMTP_PASS') ?: '';
        $useTls = filter_var(
            getenv('SMTP_TLS') ?: 'false',
            FILTER_VALIDATE_BOOLEAN
        );

        if ($host === '' || $port <= 0) {
            throw new RuntimeException('Invalid SMTP configuration (host/port).');
        }

        return new SmtpOptions(
            host: $host,
            port: $port,
            username: $username,
            password: $password,
            useTls: $useTls
        );
    }
}
```

### Service consuming options (no getenv/config calls inside)

```php
<?php

declare(strict_types=1);

final class EmailService
{
    public function __construct(private SmtpOptions $options) {}

    public function send(string $to, string $subject): void
    {
        echo sprintf(
            'Sending via %s:%d (TLS=%s) to %s with subject "%s"' . PHP_EOL,
            $this->options->host,
            $this->options->port,
            $this->options->useTls ? 'yes' : 'no',
            $to,
            $subject
        );
    }
}
```

### Composition root (bootstrap)

```php
<?php

declare(strict_types=1);

$smtpOptions = SmtpOptionsFactory::fromEnv();
$emailService = new EmailService($smtpOptions);

$emailService->send('user@example.com', 'Hello');
```

---

## Advantages and limits

### Advantages

- Centralized configuration loading and parsing
- Explicit dependencies (constructor tells what the service needs)
- Fail-fast validation (configuration errors show up early)
- Easier unit testing (you can pass fake options in tests)
- Eliminates scattered `getenv()`/`config()` calls and magic strings

### Limits

- More boilerplate (Options class + loader/binding)
- Can be overkill for very small scripts
- Not a replacement for secrets management (Vault, AWS Secrets Manager, etc.)
- Requires discipline: if people keep calling `getenv()` in services, the benefit collapses

---

## Evolutions

Historically, many PHP applications started with:
- global arrays and include-based config files
- ini files
- then environment variables and `.env` files (12-factor style)

The Options Pattern is the next step:
- configuration is modeled as structured objects
- validation happens at startup
- dependencies are explicit and testable

Modern setups often combine Options Pattern with:
- secrets managers (Vault, AWS Secrets Manager, Azure Key Vault)
- immutable options objects (readonly properties)
- environment-specific loaders (dev/stage/prod)
- centralized configuration providers
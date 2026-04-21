# Project guidance for Claude Code

This repo ships a set of Claude Code subagents for working on a **Symfony** PHP service. When Claude is invoked in a project that uses these agents, it should:

## Orchestration

- For any non-trivial code change, delegate to the `symfony-coder` subagent.
- After code is written or modified, delegate to the `code-reviewer` subagent before reporting the task complete.
- When adding or changing behavior, delegate to the `test-writer` subagent to cover it.
- The main agent is the orchestrator — it reads user intent, plans, dispatches subagents, and reconciles their output. It should not write production PHP directly unless the change is truly trivial (e.g. a typo in a comment).

## Conventions this codebase follows

- **PHP 8.2+**, strict types on every file (`declare(strict_types=1);`).
- **Symfony 6.x / 7.x** — use attributes (`#[Route]`, `#[AsCommand]`, `#[AsMessageHandler]`), not annotations.
- **PSR-12** style, enforced by `php-cs-fixer` if configured.
- **Doctrine ORM** — migrations in `migrations/`, repositories in `src/Repository/`, no raw SQL in application code.
- **PHPUnit or Pest** — detect which one; do not introduce the other.
- **Messenger** for async work; avoid ad-hoc queues.

## Guardrails

- Never add a Composer dependency without the user's explicit approval.
- Never commit `.env.local`, secrets, or generated `var/` / `vendor/` directories.
- Never modify the database schema with raw SQL; use Doctrine migrations.
- Never delete or disable tests without a stated reason.
- Before declaring a task done: run the linter, run the test suite that covers changed code, and pass the `code-reviewer` gate.

## Useful commands

```bash
# Lint
vendor/bin/php-cs-fixer fix --dry-run --diff
vendor/bin/phpstan analyse

# Tests
php bin/phpunit
# or
vendor/bin/pest

# Symfony
php bin/console cache:clear
php bin/console doctrine:migrations:migrate --no-interaction
php bin/console make:migration
```

# Agent Instructions

Always-on guidance for AI coding agents working with me. Keep this file short — it is
loaded into context on every request. Task-specific depth belongs in skills under
[`.github/skills/`](./.github/skills).

## About me

Backend software developer. Primary stack: **Java, Spring / Spring Boot, JUnit**.
I value clean, well-tested, maintainable code over clever shortcuts.

## Working style

- Make the change, then briefly confirm what you did. Don't over-explain.
- Prefer editing existing code over rewriting; keep diffs focused and minimal.
- Don't add features, comments, or abstractions I didn't ask for.
- When something is ambiguous, ask a focused question instead of guessing.

## Always expected

- Write tests for new behavior (JUnit). Cover edge cases and failure paths.
- Use constructor injection; avoid field injection.
- Surface security concerns (input validation, secrets, injection) proactively.
- Follow existing project conventions over personal defaults.

## On-demand depth

For Java / Spring / JUnit specifics (layering, naming, test patterns), use the
**java-spring-backend** skill.

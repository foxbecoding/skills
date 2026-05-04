## Module Layout

Follow the **Rust 2018+ module convention** — never use `mod.rs`:

- A module with submodules is declared as `foo.rs`
- Its submodules live in a `foo/` directory alongside `foo.rs`

```
src/
├── crawler.rs          ← declares the module + its submodules
├── crawler/
│   ├── static_crawler.rs
│   └── js_crawler.rs
```

Never use `foo/mod.rs` — that is the old convention.

## Error Handling

- Use `thiserror` for library/module errors — derive `Error` on explicit error enums
- Use `anyhow` for application-level error handling and propagation
- Always use `?` propagation — never `.unwrap()` or `.expect()` in production paths
- Error variants must be meaningful — never use a catch-all string error in library code

```rust
// library error
#[derive(Debug, thiserror::Error)]
pub enum CrawlError {
    #[error("fetch failed: {0}")]
    Fetch(#[from] reqwest::Error),
    #[error("page timeout after {0}s")]
    Timeout(u64),
}

// application entry point
async fn run() -> anyhow::Result<()> {
    let result = fetch_page(url).await?;
    Ok(())
}
```

## Async

- Use `async`/`await` with `tokio` consistently throughout
- Never mix blocking calls into async code — use `tokio::task::spawn_blocking` for CPU-bound or blocking I/O work
- Prefer `tokio::time::timeout` over manual timers

## Idiomatic Patterns

**Iterators and combinators over manual loops**
```rust
// good
let active: Vec<_> = sources.iter().filter(|s| s.is_active).collect();
let ids: Vec<u32> = jobs.iter().map(|j| j.id).collect();
let total = scores.iter().sum::<f64>();

// avoid
let mut active = vec![];
for s in &sources {
    if s.is_active { active.push(s); }
}
```

**Pattern matching — exhaust variants, avoid if-let chains**
```rust
match result {
    Ok(content) => process(content),
    Err(CrawlError::Timeout(secs)) => warn!("timed out after {secs}s"),
    Err(e) => return Err(e.into()),
}
```

**Use the type system to make invalid states unrepresentable**
```rust
// good — state encoded in types
enum CrawlJob {
    Static { url: Url },
    JavaScript { url: Url, wait_ms: u64 },
}

// avoid — booleans that must stay in sync
struct CrawlJob {
    url: String,
    is_js: bool,
    wait_ms: Option<u64>,
}
```

**Ownership and borrowing**
- Pass `&T` for read-only access, `&mut T` for mutation, `T` only when ownership transfer is needed
- Prefer returning owned types from public APIs — callers shouldn't fight lifetimes at the boundary
- Use `Cow<str>` when a function sometimes allocates and sometimes borrows

**Struct construction — use `Default` and builder patterns for complex types**
```rust
#[derive(Default)]
struct CrawlerConfig {
    timeout_secs: u64,
    max_retries: u32,
    user_agent: String,
}
```

**Closures over function pointers when capturing context**
```rust
let threshold = config.min_score;
let significant: Vec<_> = changes.iter().filter(|c| c.score >= threshold).collect();
```

**`if let` and `while let` for single-variant matching**
```rust
if let Some(job) = queue.pop() {
    process(job).await?;
}
```

**Use `?` in `Option` chains with `.ok_or`**
```rust
let host = url.host_str().ok_or(CrawlError::InvalidUrl)?;
```

## Logging

Use `tracing` for all structured logging — never `println!` or `eprintln!` in production code:

```rust
use tracing::{info, warn, error, debug, instrument};

#[instrument(skip(client))]
async fn fetch(client: &Client, url: &str) -> Result<String, CrawlError> {
    info!(url, "fetching page");
    ...
}
```

## Doc Comments

All public functions, types, and modules must have doc comments (`///`).

**Functions and methods** — structured sections, concise entries:

```rust
/// Brief one-line description of what the function does.
///
/// # Arguments
///
/// * `value` - Description of the parameter.
///
/// # Returns
///
/// Description of the return value.
///
/// # Errors
///
/// Returns [`CrawlError::Timeout`] if the page load exceeds the configured limit.
///
/// # Example
///
/// ```rust
/// let result = crawler.fetch("https://example.com").await?;
/// ```
pub async fn fetch(url: &str) -> Result<String, CrawlError> { ... }
```

**Structs and enums** — go beyond a one-liner. Add a second paragraph explaining the type's purpose, role in the system, key invariants, and relationships to other types. Field and variant comments stay concise:

```rust
/// A crawl job pulled from the Redis queue.
///
/// Represents a single unit of work for the crawler. The variant determines
/// which fetcher is used — `Static` uses reqwest directly, `JavaScript` routes
/// through the chromiumoxide → browserless/chrome CDP pipeline. Created by the
/// scheduler and consumed by `Dispatcher`.
pub enum CrawlJob {
    /// Plain HTTP fetch — no JS rendering required.
    Static { url: Url },
    /// JS-rendered fetch via headless Chrome.
    JavaScript { url: Url, wait_ms: u64 },
}
```

Skip sections (`# Arguments`, `# Errors`, etc.) that do not apply. Use `# Example` only when usage is non-obvious.

## Clippy

Run `clippy` and resolve all warnings before committing:

```bash
cargo clippy -- -D warnings
```

Any `#[allow(...)]` suppression must have a comment explaining why.

## What to Avoid

- `mod.rs` — use the 2018+ convention
- `.unwrap()` / `.expect()` in non-test, non-prototype code
- `println!` / `eprintln!` — use `tracing` instead
- Blocking calls inside async functions — use `spawn_blocking`
- String errors in library code — use typed error enums with `thiserror`
- Unnecessary `clone()` — pass references instead
- `type_alias_impl_trait` workarounds when `impl Trait` in return position works
- Manual `loop` + `push` when an iterator combinator expresses the same thing

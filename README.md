# backoff-futures

[![Build Status](https://dev.azure.com/jakubadamw/github/_apis/build/status/jakubadamw.backoff-futures?branchName=master)](https://dev.azure.com/jakubadamw/github/_build/latest?definitionId=3&branchName=master)
[![crates.io: backoff-futures](https://img.shields.io/crates/v/backoff-futures.svg)](https://crates.io/crates/backoff-futures)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A retry and backoff mechanism for `std::future::Future`.

## Adding as a dependency

### Manually

```toml
[dependencies]
backoff-futures = "0.1"
```

### With `cargo-edit`

```
cargo add backoff-futures
```

## Usage

```rust
#![feature(async_await)]

fn isahc_error_to_backoff(err: isahc::Error) -> backoff::Error<isahc::Error> {
    match err {
        isahc::Error::Aborted | isahc::Error::Io(_) | isahc::Error::Timeout =>
            backoff::Error::Transient(err),
        _ =>
            backoff::Error::Permanent(err)
    }
}

async fn get_example_contents() -> Result<String, backoff::Error<isahc::Error>> {
    use isahc::ResponseExt;

    let mut response = isahc::get_async("https://example.org")
        .await
        .map_err(isahc_error_to_backoff)?;

    response
        .text_async()
        .await
        .map_err(|err: std::io::Error| backoff::Error::Transient(isahc::Error::Io(err)))
}

async fn get_example_contents_with_retry() -> Result<String, isahc::Error> {
    use backoff_futures::BackoffExt;

    let mut backoff = backoff::ExponentialBackoff::default();
    get_example_contents.with_backoff(&mut backoff)
        .await
        .map_err(|err| match err {
            backoff::Error::Transient(err) | backoff::Error::Permanent(err) => err
        })
}
```

## License
[MIT](https://choosealicense.com/licenses/mit/)


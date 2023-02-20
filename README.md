Original repository.   
https://github.com/uber-go/zap.   

# :zap: zap [![GoDoc][doc-img]][doc] [![Build Status][ci-img]][ci] [![Coverage Status][cov-img]][cov]

Blazing fast, structured, leveled logging in Go.

## Installation

`go get -u go.uber.org/zap`

Note that zap only supports the two most recent minor versions of Go.

## Quick Start

In contexts where performance is nice, but not critical, use the
`SugaredLogger`. It's 4-10x faster than other structured logging
packages and includes both structured and `printf`-style APIs.

```go
logger, _ := zap.NewProduction()
defer logger.Sync() // flushes buffer, if any
sugar := logger.Sugar()
sugar.Infow("failed to fetch URL",
  // Structured context as loosely typed key-value pairs.
  "url", url,
  "attempt", 3,
  "backoff", time.Second,
)
sugar.Infof("Failed to fetch URL: %s", url)
```

When performance and type safety are critical, use the `Logger`. It's even
faster than the `SugaredLogger` and allocates far less, but it only supports
structured logging.

```go
logger, _ := zap.NewProduction()
defer logger.Sync()
logger.Info("failed to fetch URL",
  // Structured context as strongly typed Field values.
  zap.String("url", url),
  zap.Int("attempt", 3),
  zap.Duration("backoff", time.Second),
)
```

See the [documentation][doc] and [FAQ](FAQ.md) for more details.

## Performance

For applications that log in the hot path, reflection-based serialization and
string formatting are prohibitively expensive &mdash; they're CPU-intensive
and make many small allocations. Put differently, using `encoding/json` and
`fmt.Fprintf` to log tons of `interface{}`s makes your application slow.

Zap takes a different approach. It includes a reflection-free, zero-allocation
JSON encoder, and the base `Logger` strives to avoid serialization overhead
and allocations wherever possible. By building the high-level `SugaredLogger`
on that foundation, zap lets users _choose_ when they need to count every
allocation and when they'd prefer a more familiar, loosely typed API.

As measured by its own [benchmarking suite][], not only is zap more performant
than comparable structured logging packages &mdash; it's also faster than the
standard library. Like all benchmarks, take these with a grain of salt.<sup
id="anchor-versions">[1](#footnote-versions)</sup>

Log a message and 10 fields:

| Package             |    Time     | Time % to zap | Objects Allocated |
| :------------------ | :---------: | :-----------: | :---------------: |
| zerolog             |   389 ns/op |     -53%      |    1 allocs/op    |
| :zap: zap           |   830 ns/op |      +0%      |    5 allocs/op    |
| :zap: zap (sugared) |  1377 ns/op |     +66%      |   10 allocs/op    |
| go-kit              |  4311 ns/op |     +419%     |   57 allocs/op    |
| apex/log            | 29235 ns/op |    +3422%     |   63 allocs/op    |
| logrus              | 31084 ns/op |    +3645%     |   79 allocs/op    |
| log15               | 31085 ns/op |    +3645%     |   74 allocs/op    |

Log a message with a logger that already has 10 fields of context:

| Package             |    Time     | Time % to zap | Objects Allocated |
| :------------------ | :---------: | :-----------: | :---------------: |
| zerolog             |    32 ns/op |     -27%      |    0 allocs/op    |
| :zap: zap           |    44 ns/op |      +0%      |    0 allocs/op    |
| :zap: zap (sugared) |    74 ns/op |     +68%      |    1 allocs/op    |
| go-kit              |  5072 ns/op |   +11427%     |   56 allocs/op    |
| log15               | 22634 ns/op |   +51341%     |   70 allocs/op    |
| apex/log            | 28775 ns/op |   +65298%     |   53 allocs/op    |
| logrus              | 29047 ns/op |   +65916%     |   68 allocs/op    |

Log a static string, without any context or `printf`-style templating:

| Package             |    Time    | Time % to zap | Objects Allocated |
| :------------------ | :--------: | :-----------: | :---------------: |
| standard library    |   10 ns/op |     -76%      |    1 allocs/op    |
| zerolog             |   31 ns/op |     -26%      |    0 allocs/op    |
| :zap: zap           |   42 ns/op |      +0%      |    0 allocs/op    |
| :zap: zap (sugared) |   67 ns/op |     +60%      |    1 allocs/op    |
| go-kit              |  354 ns/op |     +743%     |    9 allocs/op    |
| apex/log            | 1982 ns/op |    +4619%     |    6 allocs/op    |
| logrus              | 3451 ns/op |    +8117%     |   23 allocs/op    |
| log15               | 4744 ns/op |   +11195%     |   20 allocs/op    |

These benchmarks were ran on an AWS EC2 `m5.8xlarge` instance in November 2022.

## Development Status: Stable

All APIs are finalized, and no breaking changes will be made in the 1.x series
of releases. Users of semver-aware dependency management systems should pin
zap to `^1`.

## Contributing

We encourage and support an active, healthy community of contributors &mdash;
including you! Details are in the [contribution guide](CONTRIBUTING.md) and
the [code of conduct](CODE_OF_CONDUCT.md). The zap maintainers keep an eye on
issues and pull requests, but you can also report any negative conduct to
oss-conduct@uber.com. That email list is a private, safe space; even the zap
maintainers don't have access, so don't hesitate to hold us to a high
standard.

<hr>

Released under the [MIT License](LICENSE.txt).

<sup id="footnote-versions">1</sup> In particular, keep in mind that we may be
benchmarking against slightly older versions of other packages. Versions are
pinned in the [benchmarks/go.mod][] file. [↩](#anchor-versions)

[doc-img]: https://pkg.go.dev/badge/go.uber.org/zap
[doc]: https://pkg.go.dev/go.uber.org/zap
[ci-img]: https://github.com/uber-go/zap/actions/workflows/go.yml/badge.svg
[ci]: https://github.com/uber-go/zap/actions/workflows/go.yml
[cov-img]: https://codecov.io/gh/uber-go/zap/branch/master/graph/badge.svg
[cov]: https://codecov.io/gh/uber-go/zap
[benchmarking suite]: https://github.com/uber-go/zap/tree/master/benchmarks
[benchmarks/go.mod]: https://github.com/uber-go/zap/blob/master/benchmarks/go.mod

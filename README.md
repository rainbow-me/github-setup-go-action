# GitHub Action: Setup Go with Cache

This action is a better version of the [default Setup Go action](https://github.com/actions/setup-go).
It caches the Go modules (`~/go/pkg/mod`) and Go build (`~/.cache/go-build`) folders, but specifies a restore key in
such way that the cache can be used even when changing the `go.mod` file.

For optimal performance, this action should be run in workflows that run both on PRs and on the main branch.
This way, any branch that originates from the main branch can use the cache built on the main branch.

## Example usage:

Use after checkout, so it can detect the `go.sum` file.

```yaml
steps:
  - uses: actions/checkout@v5

  - name: Setup Go
    uses: rainbow-me/github-setup-go-action@v3
    with:
      cache-key: test
```

Note: by default it expects a `go.mod` and `go.sum` file in the project root, from which the Go version is inferred and
a hash for the cache key is created.

## Params

- `cache-key`: this parameter is mandatory, and it allows specifying a name for the cache. Different workflows should
  use different names. For example, if running one workflow for building, one for testing and one for linting, you
  should use `build`, `test` and `lint` keys to avoid these workflows overwriting the same cache.
- `go-version-file`: optional, defaults to `go.mod` - defines the path of the go.mod file from which the Go version to
  be installed can be inferred
- `cache-dependency-path`: optional, defaults to `go.sum` - defines the path of the file that should be hashed to
  determine the cache key
- `personal-access-token`: optional, if present will also set up git to pull repos with a personal access token, should
  be used when the workflow requires access to private repos

## Further details

The cache key is created as such:

```yaml
key: ${{ runner.os }}-gocache-${{ inputs.cache-key }}-${{ hashFiles(inputs.cache-dependency-path) }}
```

Example value: `Linux-gocache-test-0bf2ed6fb8b0f2910bcdeb35b592d675318ef859477b1b3bd2765c501e00d99a`

It falls back to `Linux-gocache-test*` in case the exact hash could not be matched. This allows using caches from the
main branch even in case of a change of dependencies, so the pipeline only needs to download new dependencies instead of
invalidating the whole cache.

Note that the build cache also includes test caches, which speeds up testing of packages with unchanged code, unless
the test cache is explicitly disabled in your workflow. 
# Go Meta Linter

The number of tools for statically checking Go source for errors and warnings
is impressive.

This is a tool that concurrently runs a whole bunch of those linters and
normalises their output to a standard format. It is intended for use with
editor/IDE integration.

Currently supported linters are: golint, go tool vet, gotype, errcheck,
varcheck and defercheck. Additional linters can be added through the
command line with `--linter=NAME:COMMAND:PATTERN` (see [below](#details)).

## Quickstart

Install gometalinter:

```
$ go get github.com/alecthomas/gometalinter
```

Install all known linters:

```
$ gometalinter --install
Installing errcheck -> go get github.com/kisielk/errcheck
Installing structcheck -> go get github.com/opennota/check/cmd/structcheck
Installing vet -> go get golang.org/x/tools/cmd/vet
Installing deadcode -> go get github.com/remyoudompheng/go-misc/deadcode
Installing golint -> go get github.com/golang/lint/golint
Installing gotype -> go get golang.org/x/tools/cmd/gotype
Installing defercheck -> go get github.com/opennota/check/cmd/defercheck
Installing varcheck -> go get github.com/opennota/check/cmd/varcheck
Installing gocyclo -> go get github.com/fzipp/gocyclo
```

Run it:

```
$ cd $GOPATH/src/github.com/alecthomas/gometalinter/example
$ gometalinter
stutter.go:22::error: Repeating defer a.Close() inside function duplicateDefer
stutter.go:12:6:warning: exported type MyStruct should have comment or be unexported
stutter.go:16:6:warning: exported type PublicUndocumented should have comment or be unexported
stutter.go:21:15:warning: error return value not checked (defer a.Close())
stutter.go:22:15:warning: error return value not checked (defer a.Close())
stutter.go:27:6:warning: error return value not checked (doit()           // test for errcheck)
stutter.go:9::warning: unused global variable unusedGlobal
stutter.go:13::warning: unused struct field MyStruct.Unused
stutter.go:29::error: unreachable code
stutter.go:26::error: missing argument for Printf("%d"): format reads arg 1, have only 0 args
```

## Details

```
$ gometalinter --help
usage: main [<flags>] [<path>]

Aggregate and normalise the output of a whole bunch of Go linters.

Default linters:

  errcheck (github.com/kisielk/errcheck)
      errcheck {path}
      (?P<path>[^:]+):(?P<line>\d+):(?P<col>\d+)\t(?P<message>.*)
  varcheck (github.com/opennota/check/cmd/varcheck)
      varcheck {path}
      PATH:LINE:MESSAGE
  gocyclo (github.com/fzipp/gocyclo)
      gocyclo -over {mincyclo} {path}
      (?P<cyclo>\d+)\s+\S+\s\S+\s+(?P<path>[^:]+):(?P<line>\d+):(?P<col>\d+)
  golint (github.com/golang/lint/golint)
      golint {path}
      PATH:LINE:COL:MESSAGE
  gotype (golang.org/x/tools/cmd/gotype)
      gotype {path}
      PATH:LINE:COL:MESSAGE
  structcheck (github.com/opennota/check/cmd/structcheck)
      structcheck {path}
      PATH:LINE:MESSAGE
  defercheck (github.com/opennota/check/cmd/defercheck)
      defercheck {path}
      PATH:LINE:MESSAGE
  deadcode (github.com/remyoudompheng/go-misc/deadcode)
      deadcode {path}
      deadcode: (?P<path>[^:]+):(?P<line>\d+):(?P<col>\d+):\s*(?P<message>.*)
  vet (golang.org/x/tools/cmd/vet)
      go vet {path}
      PATH:LINE:MESSAGE

Severity override map (default is "error"):

  golint -> warning
  varcheck -> warning
  structcheck -> warning
  deadcode -> warning
  gocyclo -> warning
  errcheck -> warning

Flags:
  --help             Show help.
  --fast             Only run fast linters.
  -i, --install      Attempt to install all known linters.
  -u, --update       Pass -u to go tool when installing.
  -D, --disable=LINTER
                     List of linters to disable.
  -d, --debug        Display messages for failed linters, etc.
  -j, --concurrency=16
                     Number of concurrent linters to run.
  --exclude=REGEXP   Exclude messages matching this regular expression.
  --cyclo-over="10"  Report functions with cyclomatic complexity over N (using gocyclo).
  --linter=NAME:COMMAND:PATTERN
                     Specify a linter.
  --message-overrides=LINTER:MESSAGE
                     Override message from linter. {message} will be expanded to the original message.
  --severity=LINTER:SEVERITY
                     Map of linter severities.

Args:
  [<path>]  Directory to lint.
```

Additional linters can be configured via the command line:

```
$ gometalinter --linter='vet:go tool vet -printfuncs=Infof,Debugf,Warningf,Errorf {paths}:PATH:LINE:MESSAGE' .
stutter.go:22::error: Repeating defer a.Close() inside function duplicateDefer
stutter.go:21:15:warning: error return value not checked (defer a.Close())
stutter.go:22:15:warning: error return value not checked (defer a.Close())
stutter.go:27:6:warning: error return value not checked (doit()           // test for errcheck)
stutter.go:9::warning: unused global variable unusedGlobal
stutter.go:13::warning: unused struct field MyStruct.Unused
stutter.go:12:6:warning: exported type MyStruct should have comment or be unexported
stutter.go:16:6:warning: exported type PublicUndocumented should have comment or be unexported
```


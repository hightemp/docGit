# Как получить текущее название ветки в Git?

```console
$ git branch | grep \* | cut -d ' ' -f2
```

```console
$ git rev-parse --abbrev-ref HEAD
```
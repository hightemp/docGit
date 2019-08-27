# Как получить текущее название ветки в Git?

```console
$ git branch | grep \* | cut -d ' ' -f2
```

```console
$ git rev-parse --abbrev-ref HEAD
```

**********
[ветка](/tags/%D0%B2%D0%B5%D1%82%D0%BA%D0%B0.md)

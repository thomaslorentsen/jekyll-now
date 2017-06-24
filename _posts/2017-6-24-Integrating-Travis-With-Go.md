---
layout: post
title: Integrating Travis With Go
---

If you are using a Makefile to build the Go project then your ```.travis.yml``` config needs to look like this.
```yaml
language: go

go:
  - 1.8

install: go get -t ./...

script:
  - go test ./...
  - make
```

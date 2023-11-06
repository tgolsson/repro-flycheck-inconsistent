This is a reproduction of an issue with flycheck where it reports inconsistent results depending on how it is invoked.

Using latest `buf` from here: https://github.com/bufbuild/buf/releases/tag/v1.27.2

Running `flycheck-compile` generates the following output, which matches running in a shell:

``` shell
-*- mode: compilation; default-directory: "~/Repositories/flycheck-buf-repro/" -*-
Compilation started at Mon Nov  6 15:59:06

/home/ts/bin/buf lint --config /home/ts/Repositories/flycheck-buf-repro/buf.yaml --error-format\=text /home/ts/Repositories/flycheck-buf-repro/MyPackage/common.proto

Compilation finished at Mon Nov  6 15:59:06, duration 0.04 s
```

Removing one of the config options makes the above fail with the expected output , so it's
definitely using the configuration file and running correctly.

Whereas having the buffer open generates the following output:

``` shell
 common.proto     3   1 error           Files with package "MyPackage" must be within a directory "MyPackage" relative to root but were in directory ".". (buf)
 common.proto     3   1 error           Package name "MyPackage" should be lower_snake.case, such as "my_package". (buf)
 common.proto     3   1 error           Package name "MyPackage" should be suffixed with a correctly formed version, such as "MyPackage.v1". (buf)
```

Which are the issues being ignored by `buf.yaml`.

This is the checker definition:

``` shell
(flycheck-def-config-file-var buf-buf-yaml buf '("buf.yaml"))

(flycheck-define-checker buf
  "A protobuf checker using buf."
  :command ("buf" "lint" (config-file "--config" buf-buf-yaml) "--error-format=text" source)
  :error-patterns ((error line-start (file-name) ":" line ":" column ":" (message) line-end))
  :modes (protobuf-mode)
  :working-directory (lambda (checker) (locate-dominating-file buffer-file-name "buf.yaml")))

(add-to-list 'flycheck-checkers 'buf)
```

Not quite sure why these are different, and I'm a bit unsure how to
proceed with debugging since `flycheck-compile` is the goto method for
that.

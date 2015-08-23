# Clake - Clake is a [Rake](https://github.com/ruby/rake)-like program implemented in Common Lisp.
[![Circle CI](https://circleci.com/gh/takagi/clake.svg?style=shield)](https://circleci.com/gh/takagi/clake)
[![Coverage Status](https://coveralls.io/repos/takagi/clake/badge.svg?branch=master&service=github)](https://coveralls.io/github/takagi/clake?branch=master)

## API

See [Document](https://rudolph-miller.github.io/clake/overview.html).  
This HTML is generated by [Codex](https://github.com/CommonDoc/codex).

### [Function] clake

    CLAKE &key target pathname

Loads a Clakefile specified with `pathname` to execute a task of name `target` declared in the loaded Clakefile. If `target` is not given, "default" is used for the default task name. If `pathname` is not given, a file of name `Clakefile` in the current directory is searched.

### [Function] sh

    SH command &key echo

Spawns a subprocess that runs the specified `command` given as a string. When `echo` is t, prints `command` to the standard output before runs it. Actually it is a very thin wrapper of `uiop:run-program` provided for UNIX terminology convenience.

### [Function] echo

    ECHO string

Write the given `string` into the standard output followed by a new line, provided for UNIX terminology convenience.

## Command line interface

Clake can be also used via command line interface, Clakefile in the current directory loaded. If no targets are specified, "default" is used for the default task name to be executed.

**SYNOPSIS**

    clake [ -f clakefile ] [ options ] ... [ targets ] ...

**OPTIONS**

    -f FILE
        Use FILE as a clakefile.
    -h
        Print usage.

## Clake-tools

Clake has a complementary program named `clake-tools` to provide some useful goodies.

**SYNOPSIS**

    clake-tools COMMAND

**COMMANDS**

    init    Create an empty Clakefile with boilerplates in current directory.

## Current design policy

For now trusting rake's design against preceding GNU make to follow after it.

## API design exploration

Here shows some API design examples. 

    ;; Do a task named "default" for Clakefile in the current directory.
    (clake:clake)
    
    ;; Do a task specifying its name for Clakefile in the current directory.
    (clake:clake "hello")
    
    ;; Do a task in namespace.
    (clake:clake "hello:say")
    
    ;; Also do a file task in namespace.
    (clake:clake "hello:hello")
    
    ;; Shell command interface is provided for UNIX terminology convenience.
    (clake:sh "pwd")

    ;; Also some functions are provided for UNIX terminology convenience.
    (clake:echo "Hello world.")

    ;; cl-interpol is pre-loaded for UNIX-style interpolation.
    (let ((executable "hello"))
      (clake:sh #?"gcc -o ${executable} hello.c"))

### Beyond Lisp / UNIX gap

    ;; TBD: 以下のどちらを採用するか？
    ;;      ・Unix 的に、カレントディレクトリや asdf:system-source-directory の直下の Clakefile を探す
    ;;      ・Lisp 的に、先にタスク定義を load しておく。ASDF
    ;;      -> 1) REPL からは Lisp 的に、シェルからは Unix 的に振る舞うのがよい、とする
    ;;            パケージについてどうしよう？ターゲット名であると同時に、ファイル名でもある
    ;;           ファイル名について「基準となるディレクトリ」という概念が必要。もはや Lisp 世界の話ではない
    ;;         2) Unix の世界を Lisp から間接的に操作するものなので、Unix 的に振る舞えばよい、とする

## Clakefile design exploration

Here shows what and how Clakefile should express.

    (in-package :cl-user)
    (defpackage :clake.user
      (:use :cl :clake :cl-syntax :osicat))
    (in-package :clake.user)
    
    (use-syntax :interpol)

    ;; Default task.
    (task "default" ("hello"))
    
    ;; A task that print hello.
    (task "hello" ()
      (echo "do task hello!"))

    ;; Tasks that build an executable with dependency.
    (defparameter cc "gcc")
    
    (file "hello" ("hello.o" "message.o")
      (sh #?"#{cc} -o hello hello.o message.o"))

    (file "hello.o" ("hello.c")
      (sh #?"#{cc} -c hello.c"))

    (file "message.o" ("message.c")
      (sh #?"#{cc} -c message.c"))

    (task "clean" ()
      (sh "rm -f hello hello.o message.o"))

    ;; Namespaces
    (namespace "hello"
    
      ;; TBD: How should depencency in a namespace behave?
      ;;      Does this "hello" dependency refer a task only in "hello" namespace?
      (task "say" ("hello")
        (sh "./hello"))
      
      (file "hello" ("hello.c")
        (sh #?"#{cc} -o hello hello.c")))

    ;; Can't use colons in a task name because they are reserved for namespace delimiter.
    (task "hel:lo" ())

    ;; Make a directory.
    (directory "some/directory")

    ;; Dependency tasks are specified with relative task name in the current
    ;; namespace. If absolute task names are needed, just begin with colon.
    (namespace "hello"

      ;; "bar" and ":hello:bar" refer the same task here. ":bar" refers top
      ;; level "bar" task out of "hello" namespace.
      (task "foo" ("bar" ":bar" ":hello:bar")
        (echo "hello:foo"))

      (task "bar" ()
        (echo "hello:bar")))

    (task "bar" ()
      (echo "bar"))

## Task kinds

There are some kinds of "Task" representing a sequence of shell commands.

|Name|Clakefile|Description|
|---|---|---|
|Task|task|This represents a base concept processing a sequence of shell commands.|
|File task|file|This represents a task resolving file dependency with up-to-date check.|
|Directory task|directory|Make a directory.|

## Design requirements
- dynamic task definition
- expected number of tasks in a Clakefile is ~100
- up-to-date check
- duplicated task names allowed, executed respectively
- make(1)'s internal macros
- ros interface
- modularity
- parallel task execution

## Author

* Rudolph Miller (chopsticks.tk.ppfm@gmail.com)

## Copyright

Copyright (c) 2015 Rudolph Miller (chopsticks.tk.ppfm@gmail.com)

## License

Licensed under the MIT License.

# How to build Lagopus Book

There are two Makefiles available corresponding to Lagopus switch and router.
You must specify which book you want to build like `make -f [Makefile-switch|Makefile-router] <target>`. For example:

```
> Build Lagopus switch book for Target "latexpdf"
$ make -f Makefile-switch latexpdf

> Build Lagopus router book for Target "latexpdf"
$ make -f Makefile-router latexpdf
```

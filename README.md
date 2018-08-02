Name
====

resty-cli - Fancy command-line utilities for OpenResty.

Table of Contents
=================

* [Name](#name)
* [Description](#description)
* [Synopsis](#synopsis)
* [Installation](#installation)
* [Test Suite](#test-suite)
* [Authors](#authors)
* [Copyright and License](#copyright-and-license)

Description
===========

The `resty` command-line utility can be used to run OpenResty's Lua scripts directly off the command-line just like the `lua` or `luajit` command-line utilities. It can be used to create various command-line utilities using OpenResty Lua.

This tool works by creating a head-less `nginx` instance,
disabling [daemon](http://nginx.org/en/docs/ngx_core_module.html#daemon), [master_process](http://nginx.org/en/docs/ngx_core_module.html#master_process), [access_log](http://nginx.org/en/docs/http/ngx_http_log_module.html#access_log), and other things it does
not need. No `server {}` is configured hence *no* listening sockets
are involved at all.

The Lua code is initiated by the [init_worker_by_lua](https://github.com/openresty/lua-nginx-module#init_worker_by_lua)
directive and run in the context of [ngx.timer](https://github.com/openresty/lua-nginx-module#ngxtimerat) callback. So all of
[ngx_lua](https://github.com/openresty/lua-nginx-module#readme)'s Lua APIs available in the [ngx.timer](https://github.com/openresty/lua-nginx-module#ngxtimerat) callback context are
also available in the `resty` utility. We may remove some of the
remaining limitations in the future though.

Synopsis
========

```
$ export PATH=/usr/local/openresty/bin:$PATH

$ which resty
/usr/local/openresty/bin/resty

$ resty -h
resty [options] [lua-file [args]]

Options:
    -c NUM              Set maximal connection count (default: 64).
    -e PROG             Run the inlined Lua code in "prog".

    --errlog-level LEVEL
                        Set nginx error_log level.
                        Can be debug, info, notice, warn, error, crit, alert,
                        or emerg.

    --gdb               Use GDB to run the underlying C process.

    --gdb-opts OPTS     Pass extra command-line options to GDB.

    --help              Print this help.

    --http-conf CONF    Specifies nginx.conf snippet inserted into the http {}
                        configuration block (multiple instances are supported).

    --http-include PATH Include the specified file in the nginx http
                        configuration block (multiple instances are supported).

    -I DIR              Add dir to the search paths for Lua libraries.

    -j dump             Use LuaJIT's jit.dump module to output detailed info of
                        the traces generated by the JIT compiler.

    -j off              Turn off the LuaJIT JIT compiler.

    -j v                Use LuaJIT's jit.v module to output brief info of the
                        traces generated by the JIT compiler.

    -l LIB              Require library "lib".

    --main-conf CONF    Specifies nginx.conf snippet inserted into the nginx
                        main {} configuration block (multiple instances are
                        supported).

    --main-include PATH Include the specified file in the nginx main
                        configuration block (multiple instances are supported).

    --nginx             Specify the nginx path (this option might be removed
                        in the future).

    --ns IP             Specify a custom name server (multiple instances are
                        supported).

    --resolve-ipv6      Make the nginx resolver lookup both IPv4 and IPv6
                        addresses.

    --rr                Use Mozilla rr to record the execution of the
                        underlying C process.

    --shdict 'NAME SIZE'
                        Create the specified lua shared dicts in the http
                        configuration block (multiple instances are supported).

    -V                  Print version numbers and nginx configurations.

    --valgrind          Use valgrind to run nginx.

    --valgrind-opts OPTS
                        Pass extra options to valgrind.

For bug reporting instructions, please see:

    <https://openresty.org/en/community.html>

Copyright (C) Yichun Zhang (agentzh). All rights reserved.

$ resty -e 'print("hello")'
hello

$ time resty -e 'ngx.sleep(3) print("done\n")'
done

real 0m3.085s
user 0m0.071s
sys 0m0.010s

$ resty -e 'ngx.say(ngx.md5("hello"))'
5d41402abc4b2a76b9719d911017c592

$ resty -e 'io.stderr:write("hello world\n")' > /dev/null
hello world
```

To run the code example in [lua-resty-mysql](https://github.com/openresty/lua-resty-mysql)'s
"Synopsis" documentation section:

```
$ resty mysql-example.lua
connected to mysql.
table cats created.
3 rows inserted into table cats (last insert id: 1)
result: [{"name":"Bob","id":"1"},{"name":"","id":"2"},{"name":null,"id":"3"}]
```

where `mysql-example.lua` is from [lua-resty-mysql](https://github.com/openresty/lua-resty-mysql)'s docs.

Stdin also works (in addition to stdout and stderr):

```
$ resty -e 'print("got: ", io.stdin:read("*l"))'
hiya
got: hiya
```

where the first "hiya" line was entered from the keyboard.

"Light threads" also work:

```
$ time resty -e 'local ths = {}
                 for i = 1, 3 do
                     ths[i] = ngx.thread.spawn(function ()
                                  ngx.sleep(3) ngx.say("done ", i)
                              end)
                 end
                 for i = 1, #ths do ngx.thread.wait(ths[i]) end'
done 1
done 2
done 3

real 0m3.073s
user 0m0.053s
sys 0m0.015s
```

User command-line arguments are also passed:

```
$ resty -e 'print(arg[1], ", ", arg[2])' hello world
hello, world
```

If you need to use a shared dict:

```
$ resty --shdict='dogs 1m' -e 'local dict = ngx.shared.dogs
                               dict:set("Tom", 56)
                               print(dict:get("Tom"))'
56
```

To check version numbers:

```
$ resty -V
resty 0.01
nginx version: openresty/1.7.7.2
built by gcc 4.8.3 20140911 (Red Hat 4.8.3-7) (GCC)
TLS SNI support enabled
configure arguments: --prefix=/usr/local/openresty/nginx ...
```

Installation
============

The `resty` command-line utility is bundled and enabled in OpenResty 1.7.7.2+ by
default and is installed under `<openresty-prefix>/bin/` by default.
If the OpenResty prefix is the default value (`/usr/local/openresty`),
then you can just add `/usr/local/openresty/bin` to your PATH environment:

```bash
export PATH=/usr/local/openresty/bin:$PATH
```

If you are using the official OpenResty
[pre-built Linux packages](https://openresty.org/en/linux-packages.html), you should install the
[openresty-resty](https://openresty.org/en/rpm-packages.html#openresty-resty) package and/or
the [openresty-doc](https://openresty.org/en/rpm-packages.html#openresty-doc) (or `openresty-restydoc` on
Debian/Ubuntu) package
since the [openresty](https://openresty.org/en/rpm-packages.html#openresty) binary package itself does not
contain the `resty` or `restydoc` utilities.

[Back to TOC](#table-of-contents)

Test Suite
==========

This project comes with a test suite powered by Perl's testing toolchain.

To run the test suite on your side, first install some of the CPAN modules required by the test
scaffold:

```bash
sudo cpan Test::Base IPC::Run
```

Also, you need to make your nginx executable visible in your PATH environment:

```bash
export PATH=/path/to/your/nginx/sbin:$PATH
```

Then you can run the tests with the `prove` command provided by Perl:

```bash
prove -r t
```

To run an individual test file, just specify the test file on the command-line, for instance,

```bash
prove t/resty/user-args.t
```

To run an individual test case in a test file, just (temporarily) add the line

```
--- ONLY
```

to the end of that test block, then run that particular test file as mentioned above.

[Back to TOC](#table-of-contents)

Authors
=======

* Yichun Zhang (agentzh) <agentzh@gmail.com>, CloudFlare Inc.

* Guanlan Dai <guanlan@cloudflare.com>, CloudFlare Inc.

[Back to TOC](#table-of-contents)

Copyright and License
=====================

This module is licensed under the BSD license.

Copyright (C) 2014-2018, by Yichun "agentzh" Zhang (章亦春) <agentzh@gmail.com>, OpenResty Inc.
Copyright (C) 2014-2016, by Guanlan Dai.

All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

[Back to TOC](#table-of-contents)


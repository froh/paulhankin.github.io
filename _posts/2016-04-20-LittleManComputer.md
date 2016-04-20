---
layout: post
title: Little Man Computer
---

I had never seen this mini-assembler-based educational computer before. [wikipedia.org/Little_man_computer](https://en.wikipedia.org/wiki/Little_man_computer).

I couldn't find a good online emulator, so I wrote one: [Little Man Computer Emulator](http://paulhankin.github.io/lmc/lmc.html).

Enter the program on the left, click "Assemble", enter some inputs if your program needs them, and then step
through the execution.

It's probably got some bugs since it was a quick hack, but it worked on the examples I tried it on.

<!--more-->

Here's some example programs.

* [Max of 3 numbers](http://paulhankin.github.io/lmc/lmc.html?program=ICAgIElOUAogICAgU1RBIE0wCiAgICBJTlAKICAgIFNUQSBNMQogICAgSU5QCiAgICBTVEEgTTIKCiAgICBTVUIgTTEKICAgIEJSUCBKMQogICAgTERBIE0xCiAgICBTVEEgTTIKSjEgIExEQSBNMgogICAgU1VCIE0wCiAgICBCUlAgSjIKICAgIExEQSBNMAogICAgU1RBIE0yCkoyICBMREEgTTIKICAgIE9VVAogICAgSExUCk0wICBEQVQKTTEgIERBVApNMiAgREFU&input=MjEzCjk4Nwo4OAo=)
* [Multiply 2 numbers](http://paulhankin.github.io/lmc/lmc.html?program=ICAgICBJTlAKICAgICBTVEEgUjAKICAgICBJTlAKICAgICBTVEEgUjEKCkxPT1AgTERBIFIxCiAgICAgQlJaIEVORAogICAgIFNVQiBPTkUKICAgICBTVEEgUjEKICAgICBMREEgUkVTCiAgICAgQUREIFIwCiAgICAgU1RBIFJFUwogICAgIEJSQSBMT09QCgpFTkQgIExEQSBSRVMKICAgICBPVVQKCiAgICAgLy8gVGVtcG9yYXJ5IHN0b3JhZ2UKUjEgICBEQVQKUjAgICBEQVQKUkVTICBEQVQKCiAgICAgLy8gQ29uc3RhbnRzCk9ORSAgREFUIDE=&input=NAo1)
* [Shift left](http://paulhankin.github.io/lmc/lmc.html?program=ICAgICBJTlAKICAgICBTVEEgUjAKICAgICBJTlAKICAgICBTVEEgUjEKTE9PUCBMREEgUjEKICAgICBCUlogRU5ECiAgICAgU1VCIE9ORQogICAgIFNUQSBSMQogICAgIExEQSBSMAogICAgIEFERCBSMAogICAgIFNUQSBSMAogICAgIEJSQSBMT09QCkVORCAgTERBIFIwCiAgICAgT1VUCgpSMSAgIERBVApSMCAgIERBVApPTkUgIERBVCAx&input=MTQKMw==)
* [Copy N inputs to an array](http://paulhankin.github.io/lmc/lmc.html?program=ICAgICBJTlAKICAgICBTVEEgQwpMICAgIExEQSBDCiAgICAgQlJaIEMKICAgICBTVUIgT05FCiAgICAgU1RBIEMKICAgICBMREEgVAogICAgIEFERCBPTkUKICAgICBTVEEgVAogICAgIEFERCBTVEFPUAogICAgIFNUQSBTVEFJCiAgICAgSU5QClNUQUkgREFUCiAgICAgQlJBIEwKCkMgICAgREFUCk9ORSAgREFUIDEKU1RBT1AgREFUIDMwMApUICAgIERBVCA0OQo=&input=NQoxMAoxMDEKMTQKOTk4CjgK)

The last is the most interesting: LMC doesn't have indirect addressing -- the only store instruction is `STA xx` where `xx` is a fixed address. To work around that, the code modifies itself, writing the appropriate `STA` instruction once per loop before it's executed.

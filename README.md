quickstack is a tool to take call stack traces with minimal overheads.

There are a couple of tools to take stack traces such as gdb, pstack, but these tools
have serious overheads. In many cases, target process stops for 0.2-N seconds.
So it is dangerous to use such tools in production environment.
quickstack makes it possible to take stack traces in less than 1 milliseconds. This is
much smaller overhead so you can frequently take stack traces in production environment.

quickstack internally scans stack frames and guesses caller functions.
For 64bit applications, it is highly recommended to build with -fno-omit-frame-pointer.

## How to build:

* Install binutils 2.22+ and elfutils-libelf-devel
* Install cmake
* cmake .
* make
* make install

Install dependencies (Ubuntu):
sudo apt-get install binutils-dev
sudo apt-get install libelf-dev
sudo apt-get install libiberty-dev

## Direct download:

a prebuild binary is provided in `bin/` directory, built under Linux version 2.6.32.el6.x86_64 (gcc version 4.4.4 20100726 (Red Hat 4.4.4-13) (GCC) )


## Example:

quick print process's all backtrace

[user$] quickstack -p `pidof mysqld`

print specific thread backtrace

[user$] quickstack -p `pidof mysqld` -i=32311

slow (VERY SLOW) print process's all backtrace with top 3 frames with line number

[user$] quickstack -f -p `pidof mysqld` -q 3

## License:

BSD 3-Clause License

## MISC

For better understanding pstack result, following script `pstack-agg.py` is recommonded:



```
#!/usr/bin/env python2
"""
# author yuanqi
quickstack <pid> | pstack-agg.py
"""

import sys
import re
import itertools
import pprint

def help():
    print __doc__

if __name__ == '__main__':
    rexp = '^Thread (\d+).*?\n(.*?)(?=\nThread)'
    tid_stack_list = re.findall(rexp, sys.stdin.read() + '\nThread', re.M|re.S)
    key_func = lambda x: x[1]
    for tid_list, stack_trace in sorted(([[tid for tid, stack in grp], key] for key, grp in itertools.groupby(sorted(tid_stack_list, key = key_func), key_func)), key=lambda x: len(x[0]), reverse=True):
        print 'TID: %s'%(' '.join(tid_list))
        print stack_trace
```

Save above script as pstack-agg.py. Usage:

```
quickstack <pid> | pstack-agg.py
```


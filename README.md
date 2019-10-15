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
python -c '
import sys
with open(sys.argv[1]) as f:
    s = f.read()
pstack = {}
v = ""
k = ""
t = ""
start = False
for l in s.splitlines(True):
    if start and l.startswith("#"):
        k += l.split()[1]
        v += l
    elif start and l.startswith("Thread"):
        if k in pstack:
            pstack[k][0].append(t)
        else:
            pstack[k] = [[t], v]
        t = l.split()[-1][:-3]
        v = ""
        k = ""
    elif not start and l.startswith("Thread "):
        t = l.split()[-1][:-3]
        v = ""
        k = ""
        start = True
if k in pstack:
    pstack[k][0].append(t)
else:
    pstack[k] = [[t], v]

res = pstack.values()
res.sort(key=lambda x: len(x[0]))
for v in res:
    print len(v[0]), v[0]
    print v[1]
'  pstack_output.stack > aggregated_output.stack
```

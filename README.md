# LFE OTP Tutorials

## Introduction

This repository contains the code for the series of LFE tutorials on OTP which
were announced with [this blog post](http://blog.lfe.io/tutorials/2015/05/23/1720-new-series-lfe-otp-tutorials/),
with more being linked from there as they are completed.

## Getting Started

* Make sure Erlang is installed and ``erl`` is in your ``$PATH``.
* Make sure ``git `` is installed.
* Download the code: ``git clone https://github.com/oubiwann/lfe-otp-tutorials.git``

Each tutorial is its own mini-project, complete with source code, ``Makefile``,
etc. As such, there are initial steps you will take with each tutorial (e.g.,
downloading dependencies and compileing the source).


## Intro to OTP: Servers

### 0 - Closures and Message-passing Processes

```bash
$ cd tut00
$ make repl
```

```lisp
> (set st (tut00:lambda-state 0))
#Fun<tut00.0.29422318>
> (set st (funcall st 'inc))
#Fun<tut00.0.29422318>
> (funcall st 'amount?)
1
```

```lisp
> (set st (spawn 'tut00 'process-state `(,(self) 0)))
<0.35.0>
> (! st 'inc)
inc
> (! st 'amount?)
amount?
> (c:flush)
Shell got 1
ok
```

### 1 - Converting simple server to ``gen_server``

```bash
$ cd tut01
$ make repl
```

```lisp
> (tut01-server:start)
#(ok <0.35.0>)
> (tut01-server:amount?)
0
> (tut01-server:inc)
ok
> (tut01-server:amount?)
1
> (tut01:start)
#(ok <0.40.0>)
> (tut01:amount?)
0
> (tut01:inc)
ok
> (tut01:amount?)
1
```

## Distributed LFE

### Two Nodes, Same Machine

```lfe
(repl1@cahwsx01)> (net_adm:ping 'repl2@cahwsx01)
pong
```
```lfe
(repl2@cahwsx01)> (net_adm:ping 'repl1@cahwsx01)
pong
```
```lfe
(repl1@cahwsx01)> (net_adm:names)
#(ok (#("repl1" 54162) #("repl2" 54168)))
(repl1@cahwsx01)> (rpc:call 'repl2@cahwsx01 'math 'pi '())
3.141592653589793
(repl1@cahwsx01)> (rpc:multicall 'math 'pi '())
#((3.141592653589793 3.141592653589793) ())
(repl1@cahwsx01)> (rpc:parallel_eval '(#(math pi ()) #(math exp (1))))
(3.141592653589793 2.718281828459045)
```
```lfe
(repl1@cahwsx01)> (defun ackermann
                    ((0 n) (+ n 1))
                    ((m 0) (ackermann (- m 1) 1))
                    ((m n) (ackermann (- m 1) (ackermann m (- n 1)))))
ackermann
(repl1@cahwsx01)> (defun ack-server ()
                    (register 'acksvr (self))
                    (ack-loop))
ack-serverack-loop
(repl1@cahwsx01)> (defun ack-loop ()
                    (receive
                      (`#(,pid terminate)
                        (! pid #(ok server-stopped)))
                      (`#(,pid ,m ,n)
                        (! pid (ackermann m n))
                        (ack-loop))))
ack-loop
```
```lfe
(repl1@cahwsx01)> (set rem-pid (spawn_link 'repl2@cahwsx01 #'ack-server/0))
<5881.91.0>
(repl1@cahwsx01)> (! #(acksvr repl2@cahwsx01) `#(,(self) 0 3))
#(<0.145.0> 0 3)
(repl1@cahwsx01)> (! #(acksvr repl2@cahwsx01) `#(,(self) 3 0))
#(<0.145.0> 3 0)
(repl1@cahwsx01)> (! #(acksvr repl2@cahwsx01) `#(,(self) 3 3))
#(<0.145.0> 3 3)
(repl1@cahwsx01)> (! #(acksvr repl2@cahwsx01) `#(,(self) terminate))
#(<0.145.0> terminate)
(repl1@cahwsx01)> (c:flush)
Shell got 4
Shell got 5
Shell got 61
Shell got {ok,'server-stopped'}
ok
```

### Two Nodes, Same LAN

```lfe
(repl1@10.0.4.64)> (net_adm:ping 'repl1@10.0.4.181)
pong
(repl1@10.0.4.64)>  (net_adm:names "10.0.4.181")
#(ok (#("repl1" 50123)))
```

```lfe
(repl1@10.0.4.181)> (net_adm:ping 'repl1@10.0.4.64)
pong
(repl1@10.0.4.181)> (net_adm:names "10.0.4.64")
#(ok (#("repl1" 54906)))
(repl1@10.0.4.181)> (rpc:multicall 'math 'pi '())
#((3.141592653589793 3.141592653589793) ())
(repl1@10.0.4.181)> (set rem-pid (spawn_link 'repl1@10.0.4.64 #'ack-server/0))
<6430.54.0>
(repl1@10.0.4.181)> (! #(acksvr repl1@10.0.4.64) `#(,(self) 0 3))
#(<0.35.0> 0 3)
(repl1@10.0.4.181)> (! #(acksvr repl1@10.0.4.64) `#(,(self) 3 0))
#(<0.35.0> 3 0)
(repl1@10.0.4.181)> (! #(acksvr repl1@10.0.4.64) `#(,(self) 3 3))
#(<0.35.0> 3 3)
(repl1@10.0.4.181)> (! #(acksvr repl1@10.0.4.64) `#(,(self) terminate))
#(<0.35.0> terminate)
(repl1@10.0.4.181)> (c:flush)
Shell got 4
Shell got 5
Shell got 61
Shell got {ok,'server-stopped'}
ok
```
### Two Nodes, Same Internet

Ditto. Seriously.

Please be sure to use encryption for all insecure connections (SSH tunnels, VPNs, etc.)

## Finite State Machines

TBD

## Events

TBD

## Supervisors

```bash
$ cd tut08
$ make repl
```

```lisp
> (tut08-sup:start)
#(ok <0.35.0>)
> (supervisor:which_children 'tut08-sup)
(#(child-3 <0.38.0> supervisor (tut08-child-3))
 #(child-2 <0.37.0> worker (tut08-child-2))
 #(child-1 <0.36.0> worker (tut08-child-1)))
> (whereis 'tut08-child-1)
<0.36.0>
> (tut08-child-1:stop)
ok
> (whereis 'tut08-child-1)
<0.49.0>
> (supervisor:which_children 'tut08-sup)
(#(child-3 <0.38.0> supervisor (tut08-child-3))
 #(child-2 <0.37.0> worker (tut08-child-2))
 #(child-1 <0.49.0> worker (tut08-child-1)))
> (supervisor:which_children 'tut08-child-3)
(#(tut08-grandchild-2 <0.40.0> worker (tut08-grandchild-2))
 #(tut08-grandchild-1 <0.39.0> worker (tut08-grandchild-1)))
> (exit (whereis 'tut08-grandchild-2) 'kill)
true
> (supervisor:which_children 'tut08-child-3)
(#(tut08-grandchild-2 <0.54.0> worker (tut08-grandchild-2))
 #(tut08-grandchild-1 <0.39.0> worker (tut08-grandchild-1)))
>
```

## Applications

```lisp
> (application:start 'tut09)
ok
> (supervisor:which_children 'tut09-sup)
(#(child-3 <0.41.0> supervisor (tut09-child-3))
 #(child-2 <0.40.0> worker (tut09-child-2))
 #(child-1 <0.39.0> worker (tut09-child-1)))
```

## Start Phases

```lisp
> (application:start 'tut10)
Start phase 1 ...
Start phase 2 ...
Start phase 3 ...
ok
> (supervisor:which_children 'tut10-sup)
(#(child-3 <0.41.0> supervisor (tut10-child-3))
 #(child-2 <0.40.0> worker (tut10-child-2))
 #(child-1 <0.39.0> worker (tut10-child-1)))
```

## Nested Applications

TBD

## Nested Applications with Start Phases

TBD

## Heartbeat Monitoring

TBD

## Hot-Swapping Code

TBD

## Releases

TBD

## A Complete Example

In the next set of tutorials we will be using what we've learned in all the
previous tutorials to build -- from the ground up -- a complete distributed
application. Each decision -- from architecture to behaviour options -- will be
discussed with nothing left out or gloseed over. By the end of this next series
you will have everything you need to build your own distributed applications in
LFE.

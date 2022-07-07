# port4me - Get the Same, Personal, Free TCP Port over and over

There exist many tools to identify a free TCP port, where most of them return a random port.  Although it works technically, it might add a fair bit of friction if a new random port number has to be entered by the user each time they need to use a specific tool.

To address this, **port4me** is language-agnostic algorithm that attempts, with high probability, to provide the user with the same port each time, even when used at different days.  It achieves this by scanning the same pseudo-random sequence of ports, where the random seed is a function of the user's name (`USER`) on the system, and other optional input parameters, e.g. the name of the software that will use the port.  The algorithm for generating the sequence of random ports can be perfectly reproduced in most known programming languages.  For example, in Bash, we can do:

```sh
$ port4me
10561
$ port4me
10561
```

However, if port 10561 happens to be already occupied, the next port in the pseudo-random sequence will be attempted, e.g.

```sh
$ port4me
61003
$ port4me
61003
```

To see the first five ports that are always scanned in order, do:

```sh
$ port4me --count=5 --all
10561
61003
43007
4047
31212
```

Thus, the user is likely to get assigned the same small set of ports, and often the same one, over and over, even on multi-tenant systems.  This increases the chances for them to be able to reuse the same command-line commands already available in their command-line history without having to edit port numbers.

This random sequence is initiated by a random seed that can be set via the hashcode of a seed string.  For example, we can base it of the name of the current user:

```sh
$ echo "USER=$USER"
USER=alice
$ port4me --seed="$USER"
30439
```

or a combination of a user and software name, e.g.

```sh
$ port4me --seed="$USER,rstudio"
3134
$ port4me --seed="$USER,jupyter"
45004
```

or, shortly, 

```sh
$ port4me --tool=rstudio
3134
$ port4me --tool=jupyter
45004
```

because `--seed="$USER"` is the default and any specified "tool" is appended with a comma as a separator.


This can be used to specify the port a specific software will use, e.g.

```sh
$jupyter notebook --port "$(port4me --seed="$USER,jupyter")"
```

and

```sh
$ bundle exec jekyll serve --port "$(port4me --tool=jupyter)"
```


# The port4me Algorithm

## Requirements

* It should be possible to implement the algorithm using 32-bit _signed_ integer arithmetics.  One must not assume that the largest represented integer can exceed 2^31-1.

* At a minimum, it should be possible to implement the algorithm in vanilla Sh\*, Csh, Bash, C, C++, Fortran, Lua, Python, R, and Ruby, _without_ the need for add-on packages beyond what is available from their core distribution. (*) Shells that do not support integer arithmetics, may use tools such as `expr`, `dc`, `bc`, and `awk` for these calculations.

* The implementations should be written such that they work also when sourced, or copy'and'pasted into source code elsewhere, e.g. in R and Python scripts.


## Design

* A [Linear congruential generator (LCG)](https://en.wikipedia.org/wiki/Linear_congruential_generator) will be used to generate the psuedo-random port sequence

* A [Cyclic redundancy check (CRC) sum](https://en.wikipedia.org/wiki/Cyclic_redundancy_check) will be used to generate a valid random seed from an ASCII character string of any length

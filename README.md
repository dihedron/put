# put - Yet another stream editor

```put``` is a stream editor with a limited subset of ```sed```'s functionalities exposed through a modern CLI; it is designed to be used as a shell filter, with readability in mind.

## Installation

To install ```put``` you need to compile it from sources; follow the [Golang](https.//www.golang.org) installation instructions, then clone this repository under your ```$GOPATH/src``` and compile it, as follows:

```bash
$ > cd $GOPATH/src
$ > mkdir -p github.com/dihedron
$ > cd github.com/dihedron
$ > git clone https://github.com/dihedron/put.git
$ > go install github.com/dihedron/put
```

If you added ```$GOPATH/bin``` to your ```$PATH```, the command will be readily available.

The application can be compiled on all Golang-supported OSs, including most flavours of *nix and Windows (the latter I have not tested, though); it does not require configuration or INI files, libraries, DLLs or Registry keys and can be placed anywhere on your filesystem or on portable USB sticks.

If you want to compress it to save space, you can safely use UPX to do it, like this:

```bash
$ > upx --brute $GOPATH/bin/put
```

## Usage

```put``` is meant to be used like this:

```bash
$ > cat infile.txt | 
        put "some replacement text" where "^a\s*line$" | 
        put --once "{1} but not least" before "^and this is the (last)$" |
        put "some text" after "^yet\s+another\s*line$" |         
        put - where "^a\s+line\s*to drop$"  
        > out.txt 
```

It provides 4 types of operations:
1. replace lines matching a pattern with other text (```where``` clause with replacement text);
2. add a line before each line matching a pattern (```before``` clause);
3. add a line after each line matching a pattern (```after``` clause);
4. drop lines matching a pattern (```where``` clause with ```-``` as replacement text).

The ```--once``` flags indicates that the operaton should be performed only against the first occurrence of a matching line; if omitted, each matching line is "edited" as instructed.

Replacement text can include substitution anchors, such as the ```{1}``` in the example above; it will be substituted with the value of the first capturing group in the pattern; if you write a regular expression matching the whole line (e.g. ending with ```.*$```) the zero-th anchor (```{0}```) represents the whole expression. To check how ```put``` interprets your regular expression, see [Debugging](#debugging) below.

## Example

As an example let's take an ```/etc/hosts```; say you want to add the host name on the ```localhost``` line (the one starting with ```127.0.0.1```) to prevent complaints by your ```sudo``` commands. The following sequence copies the matching line to a comment (first invocation of ```put```), then replaces whatever is after the ```localhost``` word with the current hostname:

```bash
$ > cat hosts | 
        put "# {0} (changed on $(date +%Y/%m%d))" before "^(127\.0\.0\.1\s+localhost).*" | 
        put "{1} $(hostname)" where "^(127\.0\.0\.1\s+localhost).*" 
        > hosts2
```

so that the following original file:

```
127.0.0.1	localhost
127.0.1.1	myhost.example.com	myhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
``` 

is turned into the following:

```
# 127.0.0.1	localhost (changed on 2018/0618)
127.0.0.1	localhost myhost
127.0.1.1	myhost.example.com	myhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Please note the original line is saved as a comment __before__ the changed line.

## <a name="debugging"></a>Debugging

If you want to see what the command is doing internally, simply run it with the ```PUT_DEBUG``` environment variable set to one of ```debug```, ```info```, ```warning``` or ```error```, e.g. as follows:

```bash
$ > PUT_DEBUG=debug put [args] < /etc/hosts
```

This can be hepful if you need to see what are the available bindings for your regular expression (```{0}```, ```{1}```...) so you can debug it. 

## Suggestions and contributions

... are very welcome: please open an Issue to give your feedback or to request a pull.

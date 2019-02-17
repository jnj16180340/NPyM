# NPM: Python packaging for humans
My colleagues that work with NodeJS seem constantly frustrated by bullshit that just shouldn't ever happen. I don't envy them, unless I need to coach someone through the installation of a(ny) Python project.

NPM is slow and insecure. NPM is full of terrible code whose use costs you more time than it saves. NPM's branding is slathered in glurge that evokes the essence of 21st century capitalism. And NPM is just nicer to use than anything in the Python ecosystem. One outsider, unfamiliar with the subtlties at play, has unkindly described Python packaging as `a circus run by madmen`. On bad days, I don't disagree.

Greater minds than my own have thought hard about this issue, which can be irreverently reduced to the question of "making pip more like npm". This often hints at more serious matters, like "making PyPA more like NPM Inc" and "making python more like node"; matters too dirty to think about clearly. My own question is this: Why imitate, when the real thing is right in front of you?

So, without further ado:
- NO `venv`
- NO `requirements.txt`
- NO `setup.py` and *especially* no `setuptools`
- NO `toml`, no `txt`, and no `Pipfile`
- NO `site-packages` or `dist-packages`
- NO `poetry`, nor `pipenv`
- NO `pip`


`npm install @pipper/lolwut` and then...
```
PYTHONPATH='node_modules/@pipper' python3 -c 'import sys; print(sys.path); import lolwut; import lolwut.mod; lolwut.func(); lolwut.mod.func()' 
```

it seems to work.
```
nate@mimi:~/Projects/NPyMToo$ PYTHONPATH='node_modules/@pipper' python3 -c 'import sys; print(sys.path); import lolwut; import lolwut.mod; lolwut.func(); lolwut.mod.func()'
['', '/home/nate/Projects/NPyMToo/node_modules/@pipper', '/usr/lib/python36.zip', '/usr/lib/python3.6', '/usr/lib/python3.6/lib-dynload', '/home/nate/.local/lib/python3.6/site-packages', '/usr/local/lib/python3.6/dist-packages', '/usr/lib/python3/dist-packages']
Importing from lolwut @ lolwut [/home/nate/Projects/NPyMToo/node_modules/@pipper/lolwut/__init__.py]
Importing from lolwut.mod @ lolwut [/home/nate/Projects/NPyMToo/node_modules/@pipper/lolwut/mod.py]
Calling from lolwut @ lolwut [/home/nate/Projects/NPyMToo/node_modules/@pipper/lolwut/__init__.py]
Calling from lolwut.mod @ lolwut [/home/nate/Projects/NPyMToo/node_modules/@pipper/lolwut/mod.py]
```

How to handle:
- Don't have to set env vars?
- Keeping `$PYTHONPATH` of manageable size
- Flattening $PYTHONPATH if necessary...


On importing:
- script hooks are `preinstall`,`postinstall`
- Naively flatten node_mods
- patch every `__init__` with sys.path.append?
- recursively add `PYTHONPATH`s postinstall?
- postinstall move to `$npm_config_*` whatever it is
- or append to $PYTHONPATH string in root level `.env`

preinstall/postinstall
```
PWD=/home/nate/Projects/NPyMToo/node_modules/@pipper/lolwut
INIT_CWD=/home/nate/Projects/NPyMToo
```

### Python: Ignoring site, dist packages AKA fake venv
1. Setting `$PYTHONPATH=node_modules` works
2. With `site.addsitedir(sitedir='./node_modules')`

```bash
python3 -ES -c 'import site; site.addsitedir(sitedir="./node_modules"); import sys; print(sys.path); import lolwut; import lol
wut.mod; lolwut.func(); lolwut.mod.func()'
```

#### Ignoring `dist-packages`
- Just use `python -ES`
- Trick it to think its in a venv:
  - `$PATH` must contain *exactly* `$PWD/bin`
  - PATH="${PWD}/bin:${PATH}" python3 -EI  -c 'import sys; print(sys.path)'
  - ...*and* the directory structure must be valid venv? this sux just use -ES

### On NPM
#### From npm packaging guidelines:
> name§

> The name is what your thing is called.

> Some tips:

>    Don’t put “js” or “node” in the name. It’s assumed that it’s js, since you’re writing a package.json file, and you can specify the engine using the “engines” field. (See below.)
Clearly, assuming makes an ass of u and me
>   The name will probably be passed as an argument to require(), so it should be something short, but also reasonably descriptive.
...probably it will, but that's no reason to encourage it!

### Closing remarks
It's a bit more effort to get binary modules, but that doesn't even matter if you're into `pypy` :)

The paths / version compatibility thing is a bit, but hey move fast and break things, it'll fit right in

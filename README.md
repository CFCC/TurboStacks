TurboStacks
===========

`docker-compose` infrastructure application stacks for [Camp Fitch Tech Focus](http://campcomputer.com). 

Organization
------------
The top level consists of a list of various directories representing the logical function of the stack.
For example: `dns`, `lancache`. Within each stack directory is a `docker-compose.yml` which contains
the directives needed to compose that stack.

Administration
--------------
To anyone familiar with `docker-compose` this should smell familiar:

### Startup
```
cd ${stack_dir}
docker-compose up -d
```

### Shudown
If you want to delete the leftover containers, add the `-d` flag.

```
cd ${stack_dir}
docker-compose down [-d]
```

### Restart
You can restart a particular service (as named in the `docker-compose.yml` or all of them by omission.
```
cd ${stack_dir}
docker-compose restart [service]
```

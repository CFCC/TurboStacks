TurboStacks
===========

`docker-compose` infrastructure application stacks for [Camp Fitch Tech Focus](http://campcomputer.com). 

Organization
------------
The top level consists of a list of various directories representing the logical function of the stack.
For example: `resolver`, `lancache`. Within each stack directory is a `docker-compose.yaml` which contains
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
```
cd ${stack_dir}
docker-compose down
```

### Restart
You can restart a particular service (as named in the `docker-compose.yaml` or all of them by omission.
```
cd ${stack_dir}
docker-compose restart [service]
```

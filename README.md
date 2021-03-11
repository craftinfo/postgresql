# postgresql

How to add postgresql (libpq) to a mulle-sde project

## mulle-sde dependency

### add

```
mulle-sde dependency add \
                      --c \
                      --address postgres \
                      --tag 11.2 \
                      'https://ftp.postgresql.org/pub/source/v${MULLE_TAG}/postgresql-${MULLE_TAG}.tar.bz2' 
```

As postgresql is not hosted on github, `mulle-sde domain` can't parse the URL.
So we set the address manually to "postgres".

We substitute the version information in the URL with `${MULLE_TAG}` and set
the default version with `--tag 11.2`. Later on we could override the version
with `mulle-sde environment set POSTGRES_TAG 11.3`. 

If we had set the address to "postgresql" the environment variable would be `POSTGRESQL_TAG`.


## set (tweak)
The client library we are interested in, is not called postgreql and neither is the header, so we note this.
It is important for **cmake** find the library, otherwise **cmake** will be unhappy.

```
mulle-sde dependency set postgres aliases pq
mulle-sde dependency set postgres include libpq-fe.h
```

### mark

As postgresql uses `configure` and make it doesn't partake in the multi-phase compilation assumed
by `mulle-sde` so we note this.

```
mulle-sde dependency mark postgres singlephase
```
We may not want to expose the postgres headers to consumers of our library, so we make
it private (public is the default):

```
mulle-sde dependency unmark postgres public
```

We could remove the header from automatic inclusion with

```
mulle-sde dependency unmark postgres header
```

We could remove the library from automatic linking with

```
mulle-sde dependency unmark postgres link
```



## Edit CMakeLists.txt

The postgresql headers are installed all over the place. To expand the include paths, it's easiest
to add this to your `CMakeLists.txt` file (some place after `include( Environment.cmake)`):

```
...
include( Environment)

include( Files)

#
# Add this
#
include_directories( "${DEPENDENCY_DIR}/${CMAKE_BUILD_TYPE}/include/postgresql/internal")
include_directories( "${DEPENDENCY_DIR}/include/postgresql/internal")
include_directories( "${DEPENDENCY_DIR}/${CMAKE_BUILD_TYPE}/include/postgresql/server")
include_directories( "${DEPENDENCY_DIR}/include/postgresql/server")
...
```



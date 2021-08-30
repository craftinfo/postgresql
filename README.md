# postgresql

#### How to add postgresql (libpq) to a mulle-sde project.

This repository just contains instructions how to add [postgresql](https://www.postgresql.org/). It does not
need a *craftinfo* and therefore has none.

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

The client library we are interested in, is called "libpq.a" instead of "libpostgres.a". 
And the header is  `<libpq-fe.h>` and not `<postgres/postgres.h>` as the default name would
be assumed from the address. So we tweak this.

```
mulle-sde dependency set postgres aliases pq
mulle-sde dependency set postgres include libpq-fe.h
```

### mark

As postgresql uses `configure` and `make`, it doesn't partake in the multi-phase compilation 
assumed by `mulle-sde` so we note this.

```
mulle-sde dependency mark postgres singlephase
```
We may not want to expose the postgres headers to consumers of our library, so we make
it private (public is the default):

```
mulle-sde dependency unmark postgres public
```

> #### Note
> 
> We could remove the header from automatic inclusion with
> 
> ```
> mulle-sde dependency unmark postgres header
> ```
> 
> We could remove the library from automatic linking with
> 
> ```
> mulle-sde dependency unmark postgres link
> ```

#### Changing configure options

OpenSSL is needed for login. The readline library we can live without.

```
mulle-sde dependency craftinfo set postgres CONFIGUREFLAGS "--with-openssl --without-readline"
```

#### If we want to build `fat` for macOS

Usually one would set this in the environment CFLAGS of the top most project: 

```
mulle-sde environment --os darwin set CFLAGS "-arch x86_64 -arch arm64"
```

But one can also change this on the dependency itself

```
mulle-sde dependency craftinfo --os darwin set postgres CFLAGS "-arch x86_64 -arch arm64"
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
include_directories( SYSTEM "${DEPENDENCY_DIR}/${CMAKE_BUILD_TYPE}/include/postgresql/internal")
include_directories( SYSTEM "${DEPENDENCY_DIR}/include/postgresql/internal")
include_directories( SYSTEM "${DEPENDENCY_DIR}/${CMAKE_BUILD_TYPE}/include/postgresql/server")
include_directories( SYSTEM "${DEPENDENCY_DIR}/include/postgresql/server")
...
```



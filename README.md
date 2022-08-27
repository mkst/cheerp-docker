# cheerp-docker

Docker files to build the Cheerp compiler, utilities and libraries for various environments.

NOTE: Currently only Ubuntu 22.04 is supported.

**Build**
```sh
docker build . \
    -f ubuntu/22.04/Dockerfile \
    -t cheerp-compiler:2.7
```

**Run**
```sh
docker run --rm -ti -v $(pwd):/work cheerp-compiler:2.7
# /opt/cheerp/bin/clang++ /work/tests/example.cpp -o /work/tests/cheerp_example.js -O3
```

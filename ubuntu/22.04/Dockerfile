FROM ubuntu:22.04 as build

RUN apt-get update && apt-get install -y \
    cmake \
    gcc \
    g++ \
    git \
    lld \
    ninja-build \
    python3 \
    python3-distutils

RUN mkdir -p /cheerp

WORKDIR /cheerp

ENV CHEERP_SRC=/cheerp

# cheerp-compiler

RUN git clone --branch cheerp-2.7 --depth 1 https://github.com/leaningtech/cheerp-compiler /cheerp/cheerp-compiler
WORKDIR /cheerp/cheerp-compiler

RUN cmake -S llvm -B build -C llvm/CheerpCmakeConf.cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS=clang -G Ninja
RUN ninja -C build -j4
RUN ninja -C build install

RUN git clone --branch cheerp-2.7 --depth 1 https://github.com/leaningtech/cheerp-utils /cheerp/cheerp-utils
WORKDIR /cheerp/cheerp-utils

RUN cmake -B build -DCMAKE_INSTALL_PREFIX=/opt/cheerp .
RUN make -C build install

# cheerp-newlib

RUN git clone --branch cheerp-2.7 --depth 1 https://github.com/leaningtech/cheerp-newlib /cheerp/cheerp-newlib

RUN mkdir -p /cheerp/cheerp-newlib/newlib/build_genericjs
WORKDIR /cheerp/cheerp-newlib/newlib/build_genericjs

RUN ../configure --host=cheerp-genericjs --with-cxx-headers=$PWD/../../cheerp-compiler/libcxx/include --prefix=/opt/cheerp --enable-newlib-io-long-long --enable-newlib-iconv --enable-newlib-iconv-encodings=utf-16,utf-8,ucs_2 --enable-newlib-mb --enable-newlib-nano-formatted-io AR=${CHEERP_SRC}/cheerp-compiler/build/bin/llvm-ar RANLIB="${CHEERP_SRC}/cheerp-compiler/build/bin/llvm-ar s"
RUN make
RUN make install
RUN ../build-bc-libs.sh genericjs

RUN mkdir -p /cheerp/cheerp-newlib/newlib/build_asmjs
WORKDIR /cheerp/cheerp-newlib/newlib/build_asmjs

RUN ../configure --host=cheerp-asmjs --with-cxx-headers=$PWD/../../cheerp-compiler/libcxx/include --prefix=/opt/cheerp --enable-newlib-io-long-long --enable-newlib-iconv --enable-newlib-iconv-encodings=utf-16,utf-8,ucs_2 --enable-newlib-mb --enable-newlib-nano-formatted-io AR=${CHEERP_SRC}/cheerp-compiler/build/bin/llvm-ar RANLIB="${CHEERP_SRC}/cheerp-compiler/build/bin/llvm-ar s"
RUN make
RUN make install
RUN ../build-bc-libs.sh asmjs

# cheerp-compiler part 2

WORKDIR /cheerp/cheerp-compiler
RUN cmake -S runtimes -B build_runtimes_genericjs -GNinja -C runtimes/CheerpCmakeConf.cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE="/opt/cheerp/share/cmake/Modules/CheerpToolchain.cmake"
RUN ninja -C build_runtimes_genericjs
RUN ninja -C build_runtimes_genericjs install

RUN cmake -S runtimes -B build_runtimes_wasm -GNinja -C runtimes/CheerpCmakeConf.cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE="/opt/cheerp/share/cmake/Modules/CheerpWasmToolchain.cmake"
RUN ninja -C build_runtimes_wasm
RUN ninja -C build_runtimes_wasm install

# cheerp-libs

RUN git clone --branch cheerp-2.7 --depth 1 https://github.com/leaningtech/cheerp-libs /cheerp/cheerp-libs
WORKDIR /cheerp/cheerp-libs

RUN make -C webgles install INSTALL_PREFIX=/opt/cheerp CHEERP_PREFIX=/opt/cheerp
RUN make -C wasm install INSTALL_PREFIX=/opt/cheerp CHEERP_PREFIX=/opt/cheerp
RUN make -C stdlibs install INSTALL_PREFIX=/opt/cheerp CHEERP_PREFIX=/opt/cheerp

COPY ./tests/example.cpp /root/

RUN /opt/cheerp/bin/clang++ /root/example.cpp -o /root/cheerp_example.js -O3


FROM node:16 as test

COPY --from=build /root/cheerp_example.js /root/cheerp_example.wasm /root/

RUN node /root/cheerp_example.js


FROM ubuntu:22.04 as base

COPY --from=build /opt/cheerp /opt/cheerp

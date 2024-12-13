ARG ARCH
ARG VERSION
ARG VARIANT

FROM --platform=linux/amd64 flecspublic.azurecr.io/flecs-build AS builder

ARG ARCH
ARG VERSION
ARG VARIANT

RUN apt-get update && \
    apt-get --yes --no-install-recommends install \
    dos2unix m4 && \
    apt-get --yes clean && \
    rm -rf /var/lib/apt/lists/*

ADD app/flecs-tb.tar.xz /usr/src/app/
WORKDIR /usr/src/app

RUN git clone --depth 1 --recurse-submodules https://github.com/FLECS-Technologies/flecs-build.git

RUN cmake -G Ninja -DCMAKE_BUILD_TYPE=Release -DARCH=${ARCH} -DTB_WITH_PERF=No -DTB_WITH_CDSPLUGIN=Yes -B build .
RUN cmake --build build/ --target CmpTowerBridge
RUN cmake --install build/

FROM flecs.azurecr.io/com.codesys.control:${VERSION}

ARG ARCH
ARG VERSION
ARG VARIANT

RUN [ ! -z "${ARCH}" ]
RUN [ ! -z "${VERSION}" ]

COPY --from=builder /usr/local/CmpTowerBridge/libCmpTowerBridge.so /opt/codesys/lib
COPY --from=builder [ \
    "/usr/local/lib/libflunder.so", \
    "/usr/local/lib/libflunder.so.3", \
    "/usr/local/lib/libzenohc.so", \
    "/usr/local/lib/x86_64-linux-gnu/"]

RUN ldconfig

# Add Component to .cfg
# LINE and CMPNR are coming from awk, parsing all lines starting with Component.X=
# - LINE is the line where the last Component.X entry is in CODESYSControl_User.cfg
# - CMPNR is the number X of the last Component.X entry
# sed will inser a new line after the last Component.X entry with Component.(X+1)=CmpTowerBridge
RUN  ["/bin/bash", "-c", \
    "awk '/^Component\\.[0-9]+/ {last=NR; match($0, /\\.[0-9]+=/); cmp=substr($0, RSTART+1, RLENGTH-2)} END {print last, cmp}' /etc/CODESYSControl_User.cfg |\
    {\n\
        read -r LINE CMPNR \n\
        sed -i \"$((LINE+1))i Component.$((CMPNR+1))=CmpTowerBridge\" /etc/CODESYSControl_User.cfg \n\
    }"]

FROM arm32v7/golang:1.20 as chisel

RUN apt-get update \
    && apt-get install -y \
        ca-certificates \
        file

RUN git clone --no-checkout https://github.com/canonical/chisel /opt/chisel \
    && cd /opt/chisel \
    && git checkout 00f796f17323422704241517aa0386ded5e0fba1 \
    \
    && git clone --no-checkout https://github.com/canonical/rocks-toolbox /opt/rocks-toolbox \
    && cd /opt/rocks-toolbox \
    && git checkout e92d18b733647e77b6829968c91f8a16d25c1f2d

WORKDIR /opt/chisel

RUN go generate internal/deb/version.go \
    && go build ./cmd/chisel \
    && cp /opt/chisel/chisel /usr/bin/ \
    && cp /opt/rocks-toolbox/chisel-wrapper /usr/bin/

RUN groupadd \
        --gid=1654 \
        app \
    && useradd -l \
        --uid=1654 \
        --gid=1654 \
        --shell /bin/false \
        app \
    && install -d -m 0755 -o 1654 -g 1654 "/rootfs/home/app" \
    && mkdir -p "/rootfs/etc" \
    && rootOrAppRegex='^\(root\|app\):' \
    && cat /etc/passwd | grep $rootOrAppRegex > "/rootfs/etc/passwd" \
    && cat /etc/group | grep $rootOrAppRegex > "/rootfs/etc/group"

RUN mkdir -p /rootfs/var/lib/dpkg/ \
    && chisel-wrapper --generate-dpkg-status /rootfs/var/lib/dpkg/status -- \
        --release "ubuntu-22.04" --root /rootfs \
            base-files_base \
            base-files_release-info \
            ca-certificates_data \
            libc6_libs \
            libgcc-s1_libs \
            libssl3_libs \
            libstdc++6_libs \
            zlib1g_libs


FROM scratch

COPY --from=chisel /rootfs /

ENV \
    # UID of the non-root user 'app'
    APP_UID=1654 \
    # Configure web servers to bind to port 8080 when present
    ASPNETCORE_HTTP_PORTS=8080 \
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true \
    # Set the invariant mode since ICU package isn't included (see https://github.com/dotnet/announcements/issues/20)
    DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=true

# Workaround for https://github.com/moby/moby/issues/38710
COPY --from=chisel --chown=$APP_UID:$APP_UID /rootfs/home/app /home/app

USER $APP_UID

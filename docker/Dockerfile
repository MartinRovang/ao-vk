# The cross-compile toolchain the README's "Building" section asks for, in one image:
# clang-cl + lld-link, cmake, ninja, the MSVC CRT and Windows SDK (xwin) and the Vulkan SDK.
FROM ubuntu:24.04
# Clang 19: the MSVC standard library that xwin fetches refuses anything older. Ubuntu's package has no
# clang-cl; clang itself is clang-cl when started under that name.
RUN apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates curl xz-utils cmake ninja-build python3 clang-19 lld-19 llvm-19 \
    && rm -rf /var/lib/apt/lists/* \
    && ln -s /usr/lib/llvm-19/bin/clang /usr/local/bin/clang-cl \
    && for t in lld-link llvm-rc llvm-mt llvm-lib; do ln -s /usr/lib/llvm-19/bin/$t /usr/local/bin/$t; done
# xwin downloads Microsoft's CRT and SDK; --accept-license accepts their licence terms for them.
ARG XWIN=0.6.5
RUN curl -fsSL https://github.com/Jake-Shadle/xwin/releases/download/$XWIN/xwin-$XWIN-x86_64-unknown-linux-musl.tar.gz \
        | tar xz -C /usr/local/bin --strip-components=1 xwin-$XWIN-x86_64-unknown-linux-musl/xwin \
    && xwin --accept-license --arch x86 --cache-dir /tmp/xwin-cache splat --output /opt/xwin \
    && rm -rf /tmp/xwin-cache
ARG VULKAN=1.4.321.1
RUN curl -fsSL https://sdk.lunarg.com/sdk/download/$VULKAN/linux/vulkansdk-linux-x86_64-$VULKAN.tar.xz \
        | tar xJ -C /opt $VULKAN/x86_64/bin/glslc $VULKAN/x86_64/include \
    && mv /opt/$VULKAN /opt/vulkan

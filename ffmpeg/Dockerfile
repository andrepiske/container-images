FROM ubuntu:noble

RUN set -eux ; \
    apt-get update -y && apt-get -y install \
      autoconf \
      automake \
      build-essential \
      cmake \
      git-core \
      libass-dev \
      libfreetype6-dev \
      libgnutls28-dev \
      libmp3lame-dev \
      libsdl2-dev \
      libtool \
      libva-dev \
      libvdpau-dev \
      libvorbis-dev \
      libxcb1-dev \
      libxcb-shm0-dev \
      libxcb-xfixes0-dev \
      meson \
      ninja-build \
      pkg-config \
      texinfo \
      wget \
      yasm \
      zlib1g-dev

RUN set -eux ; \
    apt-get -y install nasm libnuma-dev

# libopus, libfdk-aac, libdav1d-dev
RUN set -eux ; \
    apt-get -y install libfdk-aac-dev libopus-dev libdav1d-dev

WORKDIR /opt/videobuild

# Build libx264
RUN set -eux ; \
    git clone --depth 1 https://code.videolan.org/videolan/x264.git x264-src && \
    cd x264-src ; \
    export PATH="/opt/videobuild/ffmpeg/bin:$PATH" ; \
    export PKG_CONFIG_PATH="/opt/videobuild/ffmpeg/lib/pkgconfig" ; \
    ./configure --prefix="/opt/videobuild/ffmpeg" --bindir="/opt/videobuild/ffmpeg/bin" --enable-static --enable-pic && \
    make -j$(nproc) && \
    make install

RUN set -eux ; \
    git clone --depth 1 https://bitbucket.org/multicoreware/x265_git.git x265-src && \
    cd x265-src/build/linux ; \
    export PATH="/opt/videobuild/ffmpeg/bin:$PATH" ; \
    cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="/opt/videobuild/ffmpeg" -DENABLE_SHARED=off ../../source && \
    make -j$(nproc) && \
    make install

RUN <<EOF
cat > /opt/videobuild/ffmpeg/lib/pkgconfig/x265.pc <<'END'
prefix=/opt/videobuild/ffmpeg
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir=${prefix}/include

Name: x265
Description: H.265 encoder library
Version: 4.0
Libs: -L${exec_prefix}/lib -lx265 -lpthread -lm -ldl
Libs.private:
Cflags: -I${prefix}/include
END
EOF

# Build SVT-AV1
RUN set -eux ; \
    git clone https://gitlab.com/AOMediaCodec/SVT-AV1.git svtav1-src ; \
    mkdir -p svtav1-src/build && cd svtav1-src/build ; \
    export PATH="/opt/videobuild/ffmpeg/bin:$PATH" ; \
    cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="/opt/videobuild/ffmpeg" -DCMAKE_BUILD_TYPE=Release -DBUILD_DEC=OFF -DBUILD_SHARED_LIBS=OFF .. && \
    make -j$(nproc) && \
    make install

# libvpx
RUN set -eux ; \
   git clone --depth 1 https://chromium.googlesource.com/webm/libvpx.git libvpx-src && \
   cd libvpx-src ; \
   export PATH="/opt/videobuild/ffmpeg/bin:$PATH" ; \
   ./configure --prefix="/opt/videobuild/ffmpeg" --disable-examples --disable-unit-tests --enable-vp9-highbitdepth --as=yasm && \
   make -j$(nproc) && \
   make install

# NVIDIA things
RUN set -eux ; \
    apt-get install -y \
      nvidia-cuda-toolkit libvulkan-dev libjxl-dev libmp3lame-dev

RUN set -eux ; \
    git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git nv-codec-headers ; \
    cd nv-codec-headers ; \
    make install

RUN set -eux ; apt-get install -y libvulkan-dev

# # build ffmpeg itself
RUN set -eux ; \
    git clone --depth 1 https://github.com/FFmpeg/FFmpeg.git ffmpeg-src && \
    cd ffmpeg-src ; \
    export PATH="/opt/videobuild/ffmpeg/bin:$PATH" ; \
    export PKG_CONFIG_PATH="/opt/videobuild/ffmpeg/lib/pkgconfig" ; \
    ./configure \
      --prefix="/opt/videobuild/ffmpeg" \
      --pkg-config-flags="--static" \
      --extra-cflags="-I/opt/videobuild/ffmpeg/include -I/usr/local/cuda/include" \
      --extra-ldflags="-L/opt/videobuild/ffmpeg/lib -L/usr/local/cuda/lib64" \
      --extra-libs="-lpthread -lm -lnuma" \
      --ld="g++" \
      --bindir="/opt/videobuild/ffmpeg/bin" \
      --enable-gpl \
      --enable-libass \
      --enable-libfdk-aac \
      --enable-libfreetype \
      --enable-libmp3lame \
      --enable-libopus \
      --enable-libdav1d \
      --enable-libvorbis \
      --enable-libx264 \
      --enable-libx265 \
      --enable-nonfree \
      --enable-libjxl \
      --enable-cuda-nvcc \
      --enable-libnpp \
      --enable-libsvtav1 \
      --enable-libvpx && \
    make -j$(nproc) && \
    make install && \
    hash -r

# --enable-vulkan \

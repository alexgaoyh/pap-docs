# 1、全部清理（如果目录下有内容一定要做）
```shell
rm -rf /tmp/musl-cross-make-build
sudo rm -rf /usr/local/aarch64-linux-musl
```

# 2、clone & 构建（不要乱 cd）
```shell
sudo apt update
sudo apt upgrade
sudo apt install -y build-essential autoconf automake libtool pkg-config git libglib2.0-dev libjpeg-dev libpng-dev libtiff-dev zlib1g-dev musl musl-tools cmake unzip gawk curl zip unzip dos2unix meson ninja-build

cd /tmp
git clone https://github.com/richfelker/musl-cross-make.git
cd musl-cross-make
```

# 3、写 config.mak（关键）
```shell
cat > config.mak <<EOF
TARGET = aarch64-linux-musl
OUTPUT = /usr/local/aarch64-linux-musl

GCC_CONFIG += --enable-languages=c,c++
COMMON_CONFIG += CFLAGS="-O2"

# 使用 musl-cross-make 内置 musl（最稳）
MUSL_CROSS_STATIC = yes
EOF
```

# 4、构建 & 安装（正确方式）
```shell
make -j$(nproc)
sudo make install
```

# 5、配置 PATH
```shell
export PATH=/usr/local/aarch64-linux-musl/bin:$PATH
```

# 6、验证（你现在这一步会全部成功）
```shell
aarch64-linux-musl-gcc --version
aarch64-linux-musl-g++ --version
aarch64-linux-musl-ar --version
aarch64-linux-musl-ranlib --version
```

# 7、转 sh 文件后执行。

```shell
#!/bin/bash
set -e
set -o pipefail

# ===============================================================
# 环境配置 (请根据你的实际路径调整)
# ===============================================================
PREFIX_BASE=/usr/local
BUILD_ROOT=/tmp/build-vips-static
MAKE_JOBS=$(nproc)
PKG_DIR=/home/ubuntu/packages   # 你的源码包存放目录

# 交叉编译器路径
MUSL_PREFIX=/usr/local/aarch64-linux-musl
export CC=${MUSL_PREFIX}/bin/aarch64-linux-musl-gcc
export CXX=${MUSL_PREFIX}/bin/aarch64-linux-musl-g++
export AR=${MUSL_PREFIX}/bin/aarch64-linux-musl-ar
export RANLIB=${MUSL_PREFIX}/bin/aarch64-linux-musl-ranlib
export STRIP=${MUSL_PREFIX}/bin/aarch64-linux-musl-strip
export PATH=${MUSL_PREFIX}/bin:$PATH

# 编译参数
export CFLAGS="-O2 -static -fno-pie -no-pie -march=armv8-a"
export CXXFLAGS="-O2 -static -fno-pie -no-pie -march=armv8-a -static-libgcc -static-libstdc++"
export LDFLAGS="-static -static-libgcc -static-libstdc++ -Wl,-static -Wl,--no-dynamic-linker"

# 版本定义
ZLIB_VER=1.3.1
PNG_VER=1.6.44
JPEG_VER=3.0.1
TIFF_VER=4.7.0
PCRE2_VER=10.42
GLIB_VER=2.78.0
VIPS_VER=8.18.0

check_file() { [ ! -f "$1" ] && (echo "❌ 缺少文件：$1"; exit 1); }

sudo rm -rf $BUILD_ROOT && mkdir -p $BUILD_ROOT && cd $BUILD_ROOT

# ===============================================================
# 1. 基础图像库编译 (zlib, png, jpeg, tiff)
# ===============================================================

echo "--- Building Image Dependencies ---"
# zlib
tar -xf ${PKG_DIR}/zlib-${ZLIB_VER}.tar.gz && cd zlib-${ZLIB_VER}
./configure --static --prefix=${PREFIX_BASE}/deps-static
make -j$MAKE_JOBS && sudo make install
cd ..

# libpng
tar -xf ${PKG_DIR}/libpng-${PNG_VER}.tar.gz && cd libpng-${PNG_VER}
CPPFLAGS="-I${PREFIX_BASE}/deps-static/include" LDFLAGS="-L${PREFIX_BASE}/deps-static/lib" \
./configure --host=aarch64-linux-musl --enable-static --disable-shared --prefix=${PREFIX_BASE}/deps-static
make -j$MAKE_JOBS && sudo make install
cd ..

# libjpeg-turbo
tar -xf ${PKG_DIR}/libjpeg-turbo-${JPEG_VER}.tar.gz && cd libjpeg-turbo-${JPEG_VER}
mkdir build && cd build
cmake .. -DCMAKE_C_COMPILER=$CC -DCMAKE_INSTALL_PREFIX=${PREFIX_BASE}/deps-static -DENABLE_SHARED=OFF -DENABLE_STATIC=ON
make -j$MAKE_JOBS && sudo make install
cd ../..

# libtiff
tar -xf ${PKG_DIR}/tiff-${TIFF_VER}.tar.gz && cd tiff-${TIFF_VER}
CPPFLAGS="-I${PREFIX_BASE}/deps-static/include" LDFLAGS="-L${PREFIX_BASE}/deps-static/lib" \
./configure --host=aarch64-linux-musl --enable-static --disable-shared --prefix=${PREFIX_BASE}/deps-static \
--disable-webp --disable-zstd --disable-lzma
make -j$MAKE_JOBS && sudo make install
cd ..

# ===============================================================
# 2. 编译 GLib 及其依赖 PCRE2 (VIPS 核心)
# ===============================================================

echo "--- Building GLib Stack ---"
# PCRE2
tar -xf ${PKG_DIR}/pcre2-${PCRE2_VER}.tar.gz && cd pcre2-${PCRE2_VER}
./configure --host=aarch64-linux-musl --enable-static --disable-shared --prefix=${PREFIX_BASE}/deps-static
make -j$MAKE_JOBS && sudo make install
cd ..

# GLib (使用 Meson)
tar -xf ${PKG_DIR}/glib-${GLIB_VER}.tar.xz && cd glib-${GLIB_VER}
cat > musl_cross.txt <<EOF
[host_machine]
system = 'linux'
cpu_family = 'aarch64'
cpu = 'armv8-a'
endian = 'little'
[binaries]
c = '$CC'
cpp = '$CXX'
ar = '$AR'
strip = '$STRIP'
pkgconfig = 'pkg-config'
glib-mkenums = '/usr/bin/glib-mkenums'
glib-genmarshal = '/usr/bin/glib-genmarshal'
[built-in options]
c_args = ['-I${PREFIX_BASE}/deps-static/include']
c_link_args = ['-L${PREFIX_BASE}/deps-static/lib', '-static']
EOF

export PKG_CONFIG_PATH=${PREFIX_BASE}/deps-static/lib/pkgconfig
export PKG_CONFIG_LIBDIR=${PREFIX_BASE}/deps-static/lib/pkgconfig
export PKG_CONFIG_SYSROOT_DIR=/
meson setup build --prefix=${PREFIX_BASE}/deps-static --cross-file musl_cross.txt \
--default-library=static -Dlibmount=disabled -Dselinux=disabled -Dnls=disabled -Dtests=false
ninja -C build && sudo ninja -C build install
cd ..

# ===============================================================
# 编译 Expat (通配符自适应 + 手动强行搬运)
# ===============================================================
echo "--- Building Expat (Manual Delivery) ---"
DEPS_PREFIX=${PREFIX_BASE}/deps-static

tar -xf ${PKG_DIR}/expat-2.6.0.tar.gz && cd expat-2.6.0

# 2. 重新配置并编译
./configure --host=aarch64-linux-musl --enable-static --disable-shared --prefix=${DEPS_PREFIX}
make -j$MAKE_JOBS

# 3. 手动创建目标目录
sudo mkdir -p ${DEPS_PREFIX}/lib/pkgconfig
sudo mkdir -p ${DEPS_PREFIX}/include

# 4. 强行拷贝文件 (处理 libtool 生成的隐藏目录 .libs)
echo "--- 正在强行搬运 Expat 文件到 ${DEPS_PREFIX} ---"
sudo cp -v ./lib/.libs/libexpat.a ${DEPS_PREFIX}/lib/
sudo cp -v ./expat.pc ${DEPS_PREFIX}/lib/pkgconfig/
sudo cp -v ./lib/expat.h ./lib/expat_external.h ${DEPS_PREFIX}/include/

# 5. 修正 .pc 文件中的路径（非常重要：防止 Meson 去找根目录）
sudo sed -i "s|^prefix=.*|prefix=${DEPS_PREFIX}|" ${DEPS_PREFIX}/lib/pkgconfig/expat.pc

echo "✅ Expat 手动搬运完成"
cd ..

# ===============================================================
# 3. 编译 libvips（aarch64 + musl · 全静态修复版）
# ===============================================================

echo "--- Preparing libvips 8.18.0 build ---"

PREFIX_BASE="/usr/local"
DEPS_PREFIX="${PREFIX_BASE}/deps-static"
DEPS_LIB="${DEPS_PREFIX}/lib"
DEPS_INCLUDE="${DEPS_PREFIX}/include"

# 清理旧目录
rm -rf vips-8.18.0 build
tar -xf ${PKG_DIR}/vips-8.18.0.tar.xz
cd vips-8.18.0

# ---------------------------------------------------------------
# pkg-config 环境（只指向静态依赖）
# ---------------------------------------------------------------
export PKG_CONFIG_PATH="${DEPS_LIB}/pkgconfig"
export PKG_CONFIG_LIBDIR="${PKG_CONFIG_PATH}"
export PKG_CONFIG_SYSROOT_DIR="/"

# ---------------------------------------------------------------
# Meson 配置
# ---------------------------------------------------------------
meson setup build \
  --cross-file=../glib-2.78.0/musl_cross.txt \
  -Dprefix=/usr/local/vips-static \
  -Ddefault_library=static \
  -Dprefer_static=true \
  -Dintrospection=disabled \
  -Dmodules=disabled \
  -Dexamples=false \
  -Djpeg=enabled \
  -Dpng=enabled \
  -Dtiff=enabled \
  -Dlcms=disabled \
  -Dexif=disabled \
  -Dorc=disabled \
  -Dc_args="-I${DEPS_INCLUDE} -DHAVE_POSIX_MEMALIGN=1" \
  -Dcpp_args="-I${DEPS_INCLUDE} -DHAVE_POSIX_MEMALIGN=1"

# ---------------------------------------------------------------
# 【关键修复】musl 下 Meson 无法探测 posix_memalign
# ---------------------------------------------------------------
echo "--- Patching config.h for musl ---"
sed -i 's|.*undef HAVE_POSIX_MEMALIGN.*|#define HAVE_POSIX_MEMALIGN 1|' build/config.h

# ---------------------------------------------------------------
# 编译 & 安装 libvips（只生成静态库，不做最终链接）
# ---------------------------------------------------------------
echo "--- Building libvips ---"
ninja -C build
echo "--- Installing libvips ---"
sudo ninja -C build install

echo "✅ libvips 静态库构建完成（未链接）"
cd ..
```
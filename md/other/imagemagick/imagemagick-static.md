# [杂谈]从源码静态编译 ImageMagick

## 准备环境

```shell
#!/bin/bash
set -e
set -o pipefail

# ---------------------------------------------------------------
# Ubuntu 24 安装 musl-cross-make 并构建 x86_64-linux-musl 静态工具链
# ---------------------------------------------------------------

# 1. 安装构建依赖
echo "=== 安装构建依赖 ==="
sudo apt update
sudo apt upgrade
sudo apt install -y build-essential autoconf automake libtool pkg-config git libjpeg-dev libpng-dev libtiff-dev zlib1g-dev musl musl-tools cmake unzip gawk curl

# 2. 准备源码目录
BUILD_DIR=/tmp/musl-cross-make-build
INSTALL_PREFIX=/usr/local/x86_64-linux-musl

echo "=== 下载 musl-cross-make ==="
rm -rf "$BUILD_DIR"
mkdir -p "$BUILD_DIR"
cd "$BUILD_DIR"

git clone https://github.com/richfelker/musl-cross-make.git
cd musl-cross-make

清理之前构建缓存：
cd /home/ubuntu/packages/musl-cross-make-master
make clean
rm -rf build/local/x86_64-linux-musl
确保 Linux headers 安装：
sudo apt install linux-headers-$(uname -r)

# 3. 创建 config.mak 根据官方说明
echo "=== 写入 config.mak ==="
cat > config.mak <<EOF
TARGET = x86_64-linux-musl
OUTPUT = /usr/local/x86_64-linux-musl
COMMON_CONFIG += CFLAGS="-O2"
GCC_CONFIG += --enable-languages=c,c++
MUSL_SRC = /home/ubuntu/packages/musl-1.2.5
MUSL_CONFIG += --disable-shared
MUSL_CROSS_STATIC = yes
EOF

# 4. 构建工具链
echo "=== 构建工具链 ==="
make -j$(nproc)

cd build/local/x86_64-linux-musl/
sudo make install

# 6. 配置环境变量
echo "=== 配置环境变量 ==="
echo "export PATH=/usr/local/x86_64-linux-musl/x86_64-linux-musl/bin:\$PATH" >> ~/.bashrc
export PATH=/usr/local/x86_64-linux-musl/x86_64-linux-musl/bin:$PATH

# 7. 验证安装
echo "=== 验证工具链 ==="
x86_64-linux-musl-gcc --version
x86_64-linux-musl-g++ --version
x86_64-linux-musl-ar --version
x86_64-linux-musl-ranlib --version
```

```shell

#!/bin/bash
set -e
set -o pipefail

# ===============================================================
# Ubuntu24 + musl 全静态编译 ImageMagick
# ===============================================================

PREFIX_BASE=/usr/local
BUILD_ROOT=/tmp/build-imagemagick-static
MAKE_JOBS=$(nproc)
PKG_DIR=/home/ubuntu/packages   # 离线源码包目录

ZLIB_VERSION=1.3.1
LIBPNG_VERSION=1.6.44
LIBJPEG_TURBO_VERSION=3.0.1
LIBTIFF_VERSION=4.7.0
IMAGEMAGICK_VERSION=7.1.2-7

MUSL_PREFIX=/usr/local/x86_64-linux-musl
export CC=${MUSL_PREFIX}/bin/x86_64-linux-musl-gcc
export CXX=${MUSL_PREFIX}/bin/x86_64-linux-musl-g++
export AR=${MUSL_PREFIX}/bin/x86_64-linux-musl-ar
export RANLIB=${MUSL_PREFIX}/bin/x86_64-linux-musl-ranlib
export STRIP=${MUSL_PREFIX}/bin/x86_64-linux-musl-strip
export PATH=${MUSL_PREFIX}/bin:$PATH

export CFLAGS="-O2 -static"
export CXXFLAGS="-O2 -static -static-libgcc -static-libstdc++"
export LDFLAGS="-static -static-libgcc -static-libstdc++"

# 禁止 pkg-config 查找系统库
export PKG_CONFIG_PATH=/tmp/empty
export PKG_CONFIG_LIBDIR=/tmp/empty

check_file() {
if [ ! -f "$1" ]; then
echo "❌ 缺少文件：$1"
exit 1
fi
}

# 清理构建目录
sudo rm -rf $BUILD_ROOT
mkdir -p $BUILD_ROOT
cd $BUILD_ROOT

# ===============================================================
# 1. zlib
# ===============================================================
echo "=== 构建 zlib ${ZLIB_VERSION} ==="
check_file ${PKG_DIR}/zlib-${ZLIB_VERSION}.tar.gz
tar -xf ${PKG_DIR}/zlib-${ZLIB_VERSION}.tar.gz
cd zlib-${ZLIB_VERSION}
CC=$CC ./configure --static --prefix=${PREFIX_BASE}/zlib-static
make -j$MAKE_JOBS
sudo make install
cd ..

# ===============================================================
# 2. libpng
# ===============================================================
echo "=== 构建 libpng ${LIBPNG_VERSION} ==="
check_file ${PKG_DIR}/libpng-${LIBPNG_VERSION}.tar.gz
tar -xf ${PKG_DIR}/libpng-${LIBPNG_VERSION}.tar.gz
cd libpng-${LIBPNG_VERSION}
CPPFLAGS="-I${PREFIX_BASE}/zlib-static/include" \
LDFLAGS="-L${PREFIX_BASE}/zlib-static/lib -static" \
./configure --host=x86_64-linux-musl --enable-static --disable-shared \
--prefix=${PREFIX_BASE}/libpng-static
make -j$MAKE_JOBS
sudo make install
cd ..

# ===============================================================
# 3. libjpeg-turbo
# ===============================================================
echo "=== 构建 libjpeg-turbo ${LIBJPEG_TURBO_VERSION} ==="
check_file ${PKG_DIR}/libjpeg-turbo-${LIBJPEG_TURBO_VERSION}.tar.gz
tar -xf ${PKG_DIR}/libjpeg-turbo-${LIBJPEG_TURBO_VERSION}.tar.gz
cd libjpeg-turbo-${LIBJPEG_TURBO_VERSION}
mkdir -p build && cd build
cmake .. \
-G"Unix Makefiles" \
-DCMAKE_C_COMPILER=$CC \
-DCMAKE_CXX_COMPILER=$CXX \
-DCMAKE_BUILD_TYPE=Release \
-DCMAKE_INSTALL_PREFIX=${PREFIX_BASE}/jpeg-static \
-DENABLE_SHARED=OFF \
-DENABLE_STATIC=ON \
-DWITH_SIMD=OFF
make -j$MAKE_JOBS
sudo make install
cd ../..

# ===============================================================
# 4. libtiff
# ===============================================================
echo "=== 构建 libtiff ${LIBTIFF_VERSION} ==="
check_file ${PKG_DIR}/tiff-${LIBTIFF_VERSION}.tar.gz
tar -xf ${PKG_DIR}/tiff-${LIBTIFF_VERSION}.tar.gz
cd tiff-${LIBTIFF_VERSION}
CPPFLAGS="-I${PREFIX_BASE}/zlib-static/include -I${PREFIX_BASE}/jpeg-static/include" \
LDFLAGS="-L${PREFIX_BASE}/zlib-static/lib -L${PREFIX_BASE}/jpeg-static/lib -static" \
./configure --host=x86_64-linux-musl --enable-static --disable-shared --disable-cxx \
--prefix=${PREFIX_BASE}/tiff-static
make -j$MAKE_JOBS
sudo make install
cd ..

# ===============================================================
# 如上 imagemagick 构建的时候，有可能还是依赖到例如 libc.so 的地方，所以可以将 rm -rf /usr/local/x86_64-linux-musl/x86_64-linux-musl/lib/libc.so 删除。


# imagemagick 的 configure 检测逻辑对 libpng / libtiff 非常固执 —— 即使你写了 --with-png=yes，它仍会执行： checking for libpng >= 1.0.0 
# 并通过 pkg-config 或 libpng-config 来确认可用性。 如果找不到这些辅助脚本，它就直接认为 “no”，不会用你手动提供的 LIBS。 所以可以增加如下的文件，便于找到。
#  /usr/local/libpng-static/lib/pkgconfig/libpng.pc   需要有如下内容
#         prefix=/usr/local/libpng-static
#         exec_prefix=${prefix}
#         libdir=${exec_prefix}/lib
#         includedir=${prefix}/include/libpng16

#         Name: libpng
#         Description: Loads and saves PNG files
#         Version: 1.6.44
#         Requires.private: zlib
#         Libs: -L${libdir} -lpng16
#         Libs.private: -lz
#         Cflags: -I${includedir}

# 验证方式是： 
# pkg-config --exists libtiff-4 && echo "libtiff detected OK"
# pkg-config --exists libjpeg && echo "libjpeg detected OK"
# pkg-config --exists zlib && echo "zlib detected OK"

# ===============================================================


# ===============================================================
# 5. ImageMagick
# ===============================================================
echo "=== 构建 ImageMagick ${IMAGEMAGICK_VERSION} ==="
check_file ${PKG_DIR}/ImageMagick-${IMAGEMAGICK_VERSION}.zip
unzip ${PKG_DIR}/ImageMagick-${IMAGEMAGICK_VERSION}.zip
cd ImageMagick-${IMAGEMAGICK_VERSION}

# 修复 pkg-config 路径，确保能检测到 libpng/libtiff
export PKG_CONFIG_PATH=${PREFIX_BASE}/zlib-static/lib/pkgconfig:${PREFIX_BASE}/libpng-static/lib/pkgconfig:${PREFIX_BASE}/jpeg-static/lib/pkgconfig:${PREFIX_BASE}/tiff-static/lib/pkgconfig:$PKG_CONFIG_PATH
export PKG_CONFIG_LIBDIR="$PKG_CONFIG_PATH"

# Configure 阶段全静态链接
./configure CC=$CC CXX=$CXX \
--host=x86_64-linux-musl \
--enable-static \
--disable-shared \
--disable-openmp \
--without-modules \
--without-perl \
--without-magick-plus-plus \
--without-x \
--with-zlib=yes \
--with-png=yes \
--with-jpeg=yes \
--with-tiff=yes \
CPPFLAGS="-I${PREFIX_BASE}/zlib-static/include \
-I${PREFIX_BASE}/libpng-static/include \
-I${PREFIX_BASE}/jpeg-static/include \
-I${PREFIX_BASE}/tiff-static/include" \
LDFLAGS="-L${PREFIX_BASE}/zlib-static/lib \
-L${PREFIX_BASE}/libpng-static/lib \
-L${PREFIX_BASE}/jpeg-static/lib \
-L${PREFIX_BASE}/tiff-static/lib \
-static -static-libgcc -static-libstdc++ -Wl,-Bstatic" \
LIBPNG_CFLAGS="-I${PREFIX_BASE}/libpng-static/include" \
LIBPNG_LIBS="-L${PREFIX_BASE}/libpng-static/lib -lpng16 -lz" \
LIBTIFF_CFLAGS="-I${PREFIX_BASE}/tiff-static/include" \
LIBTIFF_LIBS="-L${PREFIX_BASE}/tiff-static/lib -ltiff -lz" \
LIBS="-lpng16 -lz -ljpeg -ltiff" \
--prefix=${PREFIX_BASE}/imagemagick-static

make -j$MAKE_JOBS
sudo make install

# ===============================================================
# 验证全静态
# ===============================================================
echo "=== 验证 ImageMagick 是否全静态 ==="
file ${PREFIX_BASE}/imagemagick-static/bin/magick
readelf -d ${PREFIX_BASE}/imagemagick-static/bin/magick | grep INTERP || echo "✅ 无 INTERP 段 -> 全静态"
readelf -d ${PREFIX_BASE}/imagemagick-static/bin/magick | grep NEEDED || echo "✅ 没有 NEEDED -> 全静态"

echo "=== 输出路径: ${PREFIX_BASE}/imagemagick-static/bin/magick ==="

```

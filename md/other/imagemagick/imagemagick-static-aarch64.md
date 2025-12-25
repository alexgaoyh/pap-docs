# 1、全部清理（如果目录下有内容一定要做）
```shell
rm -rf /tmp/musl-cross-make-build
sudo rm -rf /usr/local/aarch64-linux-musl
```

# 2、clone & 构建（不要乱 cd）
```shell
sudo apt update
sudo apt upgrade
sudo apt install -y build-essential autoconf automake libtool pkg-config git libjpeg-dev libpng-dev libtiff-dev zlib1g-dev musl musl-tools cmake unzip gawk curl

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
# Ubuntu24 + musl 全静态编译 ImageMagick（ARM / aarch64）
# ===============================================================

PREFIX_BASE=/usr/local
BUILD_ROOT=/tmp/build-imagemagick-static
MAKE_JOBS=$(nproc)
PKG_DIR=/home/ubuntu/packages   # 离线源码包目录

ZLIB_VERSION=1.3.1
LIBPNG_VERSION=1.6.44
LIBJPEG_TURBO_VERSION=3.0.1
LIBTIFF_VERSION=4.7.0
FREETYPE_VERSION=2.13.2
IMAGEMAGICK_VERSION=7.1.2-7

MUSL_PREFIX=/usr/local/aarch64-linux-musl
export CC=${MUSL_PREFIX}/bin/aarch64-linux-musl-gcc
export CXX=${MUSL_PREFIX}/bin/aarch64-linux-musl-g++
export AR=${MUSL_PREFIX}/bin/aarch64-linux-musl-ar
export RANLIB=${MUSL_PREFIX}/bin/aarch64-linux-musl-ranlib
export STRIP=${MUSL_PREFIX}/bin/aarch64-linux-musl-strip
export PATH=${MUSL_PREFIX}/bin:$PATH

export CFLAGS="-O2 -static -fno-pie -no-pie -march=armv8-a"
export CXXFLAGS="-O2 -static -fno-pie -no-pie -march=armv8-a -static-libgcc -static-libstdc++"
export LDFLAGS="-static -static-libgcc -static-libstdc++ -Wl,-static -Wl,--no-dynamic-linker"

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
LDFLAGS="-L${PREFIX_BASE}/zlib-static/lib -static -static-libgcc -static-libstdc++ -Wl,-static -Wl,--no-dynamic-linker" \
./configure --host=aarch64-linux-musl --enable-static --disable-shared \
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
-DWITH_SIMD=OFF \
-DCMAKE_EXE_LINKER_FLAGS="-static -static-libgcc -static-libstdc++ -Wl,--no-dynamic-linker" \
-DCMAKE_SHARED_LINKER_FLAGS="-static -static-libgcc -static-libstdc++ -Wl,--no-dynamic-linker"
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
LDFLAGS="-L${PREFIX_BASE}/zlib-static/lib -L${PREFIX_BASE}/jpeg-static/lib -static -static-libgcc -static-libstdc++ -Wl,-static -Wl,--no-dynamic-linker" \
./configure --host=aarch64-linux-musl --enable-static --disable-shared --disable-cxx \
--prefix=${PREFIX_BASE}/tiff-static
make -j$MAKE_JOBS
sudo make install
cd ..

# ===============================================================
# 5. freetype
# ===============================================================
echo "=== 构建 freetype ${FREETYPE_VERSION} ==="
check_file ${PKG_DIR}/freetype-${FREETYPE_VERSION}.tar.gz
tar -xf ${PKG_DIR}/freetype-${FREETYPE_VERSION}.tar.gz
cd freetype-${FREETYPE_VERSION}

CPPFLAGS="-I${PREFIX_BASE}/freetype-static/include" \
LDFLAGS="-L${PREFIX_BASE}/freetype-static/lib -static -static-libgcc -static-libstdc++ -Wl,-static -Wl,--no-dynamic-linker" \
./configure \
  --host=aarch64-linux-musl \
  --enable-static \
  --disable-shared \
  --without-bzip2 \
  --without-png \
  --without-harfbuzz \
  --without-brotli \
  --prefix=${PREFIX_BASE}/freetype-static \
  CC=${MUSL_PREFIX}/bin/aarch64-linux-musl-gcc

make -j${MAKE_JOBS}
sudo make install
cd ..


# ===============================================================
# ImageMagick
# ===============================================================
echo "=== 构建 ImageMagick ${IMAGEMAGICK_VERSION} ==="
check_file ${PKG_DIR}/ImageMagick-${IMAGEMAGICK_VERSION}.zip
unzip ${PKG_DIR}/ImageMagick-${IMAGEMAGICK_VERSION}.zip
cd ImageMagick-${IMAGEMAGICK_VERSION}

export PKG_CONFIG_PATH=${PREFIX_BASE}/zlib-static/lib/pkgconfig:${PREFIX_BASE}/libpng-static/lib/pkgconfig:${PREFIX_BASE}/jpeg-static/lib/pkgconfig:${PREFIX_BASE}/tiff-static/lib/pkgconfig:${PREFIX_BASE}/freetype-static/lib/pkgconfig
export PKG_CONFIG_LIBDIR="$PKG_CONFIG_PATH"

./configure CC=$CC CXX=$CXX \
--host=aarch64-linux-musl \
--enable-static \
--disable-shared \
--disable-openmp \
--disable-hdri \
--without-modules \
--without-perl \
--without-magick-plus-plus \
--without-x \
--without-opencl \
--with-zlib=yes \
--with-png=yes \
--with-jpeg=yes \
--with-tiff=yes \
--with-freetype=yes \
--with-freetype-includes=${PREFIX_BASE}/freetype-static/include/freetype2 \
--with-freetype-lib=${PREFIX_BASE}/freetype-static/lib \
CPPFLAGS="-I${PREFIX_BASE}/zlib-static/include \
-I${PREFIX_BASE}/libpng-static/include \
-I${PREFIX_BASE}/jpeg-static/include \
-I${PREFIX_BASE}/tiff-static/include \
-I${PREFIX_BASE}/freetype-static/include/freetype2" \
LDFLAGS="-L${PREFIX_BASE}/zlib-static/lib \
-L${PREFIX_BASE}/libpng-static/lib \
-L${PREFIX_BASE}/jpeg-static/lib \
-L${PREFIX_BASE}/tiff-static/lib \
-L${PREFIX_BASE}/freetype-static/lib \
-static -static-libgcc -static-libstdc++ -Wl,-static -Wl,--no-dynamic-linker" \
LIBS="-ltiff -lpng16 -ljpeg -lz -lfreetype -lm" \
--prefix=${PREFIX_BASE}/imagemagick-static

make -j$MAKE_JOBS
sudo make install

# ===============================================================
# 验证全静态
# ===============================================================
file ${PREFIX_BASE}/imagemagick-static/bin/magick
readelf -l ${PREFIX_BASE}/imagemagick-static/bin/magick | grep INTERP || echo "✅ 无 INTERP"
readelf -d ${PREFIX_BASE}/imagemagick-static/bin/magick | grep NEEDED || echo "✅ 无 NEEDED"
```

```markdown
在执行如上命令的时候，有时候会出现依赖  libc.so 这个文件，执行一下如下命令，之后找不到 libc.so 的话，就会去找 .a 文件，处理完之后可以再回退。
mv /usr/local/aarch64-linux-musl/aarch64-linux-musl/lib/libc.so /usr/local/aarch64-linux-musl/aarch64-linux-musl/lib/libc.so.bak
```

相信目前大部分APP都已经支持了WebP格式的图片,下面我们通过这种图片简单介绍下WebP格式图片的优点。

### 简介

WebP 的优势体现在它具有更优的图像数据压缩算法，能带来更小的图片体积，而且拥有肉眼识别无差异的图像质量；同时具备了无损和有损的压缩模式、Alpha 透明以及动画的特性，在 JPEG 和 PNG 上的转化效果都相当优秀、稳定和统一。

下面我们看下官网介绍的对于JPEG格式的图片和Gif图片压缩对比！

![](https://tva1.sinaimg.cn/large/006tNbRwly1gbc6ha4mv7j31p80dkgwp.jpg)

![](https://tva1.sinaimg.cn/large/006tNbRwly1gbc6hzbo21j30nw0oydkv.jpg)

从上面的图中我们看到webP的压缩效果还是很明显的！

### 现有方案

熟悉iOS的开发者都知道图片下载和展示的主要框架有`SDWebImage`和`YYWebImage`。当然这两个库也都支持了WebP的图片展示,下面我们先介绍下这两个现有的方案。

#### SDWebImage

在github的简介上我们看到
![](https://tva1.sinaimg.cn/large/006tNbRwly1gbc6mz3lpxj31em0fqdkj.jpg)

SD实际上支持多种图片格式的扩展!

如果项目中我们使用了SD那么我们想增加对WebP格式的图片支持,只需要增加

```
pod 'SDWebImage/WebP'
```
`注意`： 因为libwebp(0.5.1)是谷歌的库，下载需要翻墙。SD中webP的库默认是0.5.1版本的

#### YYWebImage

导入方式与SD类似直接通过pod的方式导入

```objc
pod 'YYWebImage'
pod 'YYImage/WebP`
```

一般情况下我们直接使用上面两种方法就可以解决webP的集成使用,但是还存在两个问题：
* 1、webP库的版本控制依赖三方
* 2、webP库为谷歌的库直接pod集成需要翻墙 成本较高

### 自己制作WebP

首先我们可以登录谷歌的[WebP官网](https://developers.google.com/speed/webp/docs/using)
 
![](https://tva1.sinaimg.cn/large/006tNbRwgy1gbca27ik2fj319s0tyqha.jpg)

通过上面的图我们可以看到WebP的最新版本已经到了 1.1.0(SDWebImage还是0.5.1)

我们直接去下载并解压

![](https://tva1.sinaimg.cn/large/006tNbRwgy1gbca48vq3yj30mk0za7dx.jpg)

在文件夹中我们可以看到一个`iosbuild.sh`文件,我们在终端执行这个shell脚本

```objc
sh iosbuild.sh
```
执行完成后你会发现文件夹中多了下面几个文件

![](https://tva1.sinaimg.cn/large/006tNbRwgy1gbcabyxc1dj30by04cdgg.jpg)

这就是我们需要继承的webP相关的framework

#### 多个framework合成

我们在查看YYWebImage的时候,YY实际上重写了我们在官网下载的demo中的iosbuild.sh

![](https://tva1.sinaimg.cn/large/006tNbRwgy1gbcagkifxhj31qk0cugou.jpg)

```shell
#!/bin/bash
#
# This script generates 'WebP.framework' (static library).
# An iOS app can decode WebP images by including 'WebP.framework'.
#
# 1. Download the latest libwebp source code from
#    http://downloads.webmproject.org/releases/webp/index.html
# 2. Use this script instead of the original 'iosbuild.sh' to build the WebP.framework.
#    It will build all modules, include mux, demux, coder and decoder.
#
# Notice: You should use Xcode 7 (or above) to support bitcode.

set -e

# Extract the latest SDK version from the final field of the form: iphoneosX.Y
readonly SDK=$(xcodebuild -showsdks \
  | grep iphoneos | sort | tail -n 1 | awk '{print substr($NF, 9)}'
)
# Extract Xcode version.
readonly XCODE=$(xcodebuild -version | grep Xcode | cut -d " " -f2)
if [[ -z "${XCODE}" ]]; then
  echo "Xcode not available"
  exit 1
fi

readonly OLDPATH=${PATH}

# Add iPhoneOS-V6 to the list of platforms below if you need armv6 support.
# Note that iPhoneOS-V6 support is not available with the iOS6 SDK.
PLATFORMS="iPhoneSimulator iPhoneSimulator64"
PLATFORMS+=" iPhoneOS-V7 iPhoneOS-V7s iPhoneOS-V7-arm64"
readonly PLATFORMS
readonly SRCDIR=$(dirname $0)
readonly TOPDIR=$(pwd)
readonly BUILDDIR="${TOPDIR}/iosbuild"
readonly TARGETDIR="${TOPDIR}/WebP.framework"
readonly DEVELOPER=$(xcode-select --print-path)
readonly PLATFORMSROOT="${DEVELOPER}/Platforms"
readonly LIPO=$(xcrun -sdk iphoneos${SDK} -find lipo)
LIBLIST=''

if [[ -z "${SDK}" ]]; then
  echo "iOS SDK not available"
  exit 1
elif [[ ${SDK} < 6.0 ]]; then
  echo "You need iOS SDK version 6.0 or above"
  exit 1
else
  echo "iOS SDK Version ${SDK}"
fi

rm -rf ${BUILDDIR}
rm -rf ${TARGETDIR}
mkdir -p ${BUILDDIR}
mkdir -p ${TARGETDIR}/Headers/

if [[ ! -e ${SRCDIR}/configure ]]; then
  if ! (cd ${SRCDIR} && sh autogen.sh); then
    cat <<EOT
Error creating configure script!
This script requires the autoconf/automake and libtool to build. MacPorts can
be used to obtain these:
http://www.macports.org/install.php
EOT
    exit 1
  fi
fi

for PLATFORM in ${PLATFORMS}; do
  ARCH2=""
  if [[ "${PLATFORM}" == "iPhoneOS-V7-arm64" ]]; then
    PLATFORM="iPhoneOS"
    ARCH="aarch64"
    ARCH2="arm64"
  elif [[ "${PLATFORM}" == "iPhoneOS-V7s" ]]; then
    PLATFORM="iPhoneOS"
    ARCH="armv7s"
  elif [[ "${PLATFORM}" == "iPhoneOS-V7" ]]; then
    PLATFORM="iPhoneOS"
    ARCH="armv7"
  elif [[ "${PLATFORM}" == "iPhoneOS-V6" ]]; then
    PLATFORM="iPhoneOS"
    ARCH="armv6"
  elif [[ "${PLATFORM}" == "iPhoneSimulator64" ]]; then
    PLATFORM="iPhoneSimulator"
    ARCH="x86_64"
  else
    ARCH="i386"
  fi

  ROOTDIR="${BUILDDIR}/${PLATFORM}-${SDK}-${ARCH}"
  mkdir -p "${ROOTDIR}"

  DEVROOT="${DEVELOPER}/Toolchains/XcodeDefault.xctoolchain"
  SDKROOT="${PLATFORMSROOT}/"
  SDKROOT+="${PLATFORM}.platform/Developer/SDKs/${PLATFORM}${SDK}.sdk/"
  CFLAGS="-arch ${ARCH2:-${ARCH}} -pipe -isysroot ${SDKROOT} -O3 -DNDEBUG"
  CFLAGS+=" -miphoneos-version-min=6.0 -fembed-bitcode"

  set -x
  export PATH="${DEVROOT}/usr/bin:${OLDPATH}"
  ${SRCDIR}/configure --host=${ARCH}-apple-darwin --prefix=${ROOTDIR} \
    --build=$(${SRCDIR}/config.guess) \
    --disable-shared --enable-static \
    --enable-libwebpmux \
    --enable-libwebpdemux \
    --enable-swap-16bit-csp \
    CFLAGS="${CFLAGS}"
  set +x

  # run make only in the src/ directory to create libwebpdecoder.a
  cd src/
  make V=0
  make install

  MAKEPATH=$(pwd)
  cd ${ROOTDIR}/lib/
  ar x libwebp.a
  ar x libwebpmux.a
  ar x libwebpdemux.a
  ar q webp.a *.o

  LIBLIST+=" ${ROOTDIR}/lib/webp.a"
  cd ${MAKEPATH}

  make clean
  cd ..

  export PATH=${OLDPATH}
done

cp -a ${SRCDIR}/src/webp/*.h ${TARGETDIR}/Headers/
${LIPO} -create ${LIBLIST} -output ${TARGETDIR}/WebP

```


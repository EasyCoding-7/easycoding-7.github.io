---
layout: post
title:  "(OpenSource : Telegram) Build"
summary: ""
author: OpenSource
date: '2021-05-25 0:00:00 +0000'
category: ['OpenSource', 'Telegram']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/opensource-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/opensource/telegram/build/
---

* [빌드 참고사이트](https://github.com/telegramdesktop/tdesktop/blob/dev/docs/building-msvc-x64.md)

---

## 1. 폴더생성

* 텔레그램을 빌드할 BuildPath(가칭)을 만든다.
* BuildPath내부에 ThirdParty, Libraries 폴더를 생성한다.

![](/assets/img/posts/opensource/telegram/build-1.png){:class="img-fluid"}

## 2. ThirdParty설치

* 만약 아래중 설치된게 있다면 생략해도 좋다
    * jom이 다운로드 되지 않는 경우가 있는데 qt를 설치하면 jom.exe가 자동으로설치되니 그 경로를 PATH에 넣어도 된다.

* [Qt5](https://www.qt.io/download) : Download OpenSource User로 다운
* [Visual Studio Qt확장]() : VisualStudio에서 설치
* [Strawberry Perl](http://strawberryperl.com/) : `BuildPath\ThirdParty\Strawberry`
* [NASM](http://www.nasm.us) : `BuildPath\ThirdParty\NASM`
* [Yasm](http://yasm.tortall.net/Download.html) : rename to yasm.exe and put to `BuildPath\ThirdParty\yasm`
* [MSYS2](http://www.msys2.org/) : `BuildPath\ThirdParty\msys64`
* [jom](http://download.qt.io/official_releases/jom/jom.zip) : unpack to `BuildPath\ThirdParty\jom`
* [Python 2.7](https://www.python.org/downloads/) : `BuildPath\ThirdParty\Python27`
* [CMake](https://cmake.org/download/) : `BuildPath\ThirdParty\cmake`
* [Ninja](https://github.com/ninja-build/ninja/releases/download/v1.7.2/ninja-win.zip) : unpack to `BuildPath\ThirdParty\Ninja`
* [NuGet](https://dist.nuget.org/win-x86-commandline/latest/nuget.exe) : `BuildPath\ThirdParty\NuGet`
* Git : 설명생략

## 3. BuildPath에 Make수행전 배치파일 생성

`BuildPath\PrepareMake.bat`

```
cd ThirdParty
git clone https://github.com/desktop-app/patches.git
cd patches
git checkout 41ead72
cd ../
git clone https://chromium.googlesource.com/external/gyp
cd gyp
git checkout 9f2a7bb1
git apply ../patches/gyp.diff
cd ..\..
```

```
# 실행
$ PrepareMake.bat
```

## 4. 환경변수 선언

시스템환경변수에 gyp, ninga, NuGet의 경로를 넣는다.

![](/assets/img/posts/opensource/telegram/build-2.png){:class="img-fluid"}

## 5. BuildPath에 Make 배치파일 생성

`BuildPath\Make.bat`

첫 줄

```
SET PATH=%cd%\ThirdParty\Strawberry\perl\bin;%cd%\ThirdParty\Python27;%cd%\ThirdParty\NASM;%cd%\ThirdParty\jom;%cd%\ThirdParty\cmake\bin;%cd%\ThirdParty\yasm;%PATH%
```

마지막에 자신의 msbuild.exe의 PATH를 넣는다

```
SET PATH=%cd%\ThirdParty\Strawberry\perl\bin;%cd%\ThirdParty\Python27;%cd%\ThirdParty\NASM;%cd%\ThirdParty\jom;%cd%\ThirdParty\cmake\bin;%cd%\ThirdParty\yasm;%PATH%

git clone --recursive https://github.com/telegramdesktop/tdesktop.git

if not exist Libraries\win64 mkdir Libraries\win64
cd Libraries\win64

git clone https://github.com/desktop-app/patches.git
cd patches
git checkout 41ead72
cd ..

git clone https://github.com/desktop-app/lzma.git
cd lzma\C\Util\LzmaLib
msbuild LzmaLib.sln /property:Configuration=Debug /property:Platform="x64"
msbuild LzmaLib.sln /property:Configuration=Release /property:Platform="x64"
cd ..\..\..\..

git clone https://github.com/openssl/openssl.git openssl_1_1_1
cd openssl_1_1_1
git checkout OpenSSL_1_1_1-stable
perl Configure no-shared no-tests debug-VC-WIN64A
nmake
mkdir out64.dbg
move libcrypto.lib out64.dbg
move libssl.lib out64.dbg
move ossl_static.pdb out64.dbg\ossl_static
nmake clean
move out64.dbg\ossl_static out64.dbg\ossl_static.pdb
perl Configure no-shared no-tests VC-WIN64A
nmake
mkdir out64
move libcrypto.lib out64
move libssl.lib out64
move ossl_static.pdb out64
cd ..

git clone https://github.com/desktop-app/zlib.git
cd zlib\contrib\vstudio\vc14
msbuild zlibstat.vcxproj /property:Configuration=Debug /property:Platform="x64"
msbuild zlibstat.vcxproj /property:Configuration=ReleaseWithoutAsm /property:Platform="x64"
cd ..\..\..\..

git clone -b v4.0.1-rc2 https://github.com/mozilla/mozjpeg.git
cd mozjpeg
cmake . ^
    -G "Visual Studio 16 2019" ^
    -A x64 ^
    -DWITH_JPEG8=ON ^
    -DPNG_SUPPORTED=OFF
cmake --build . --config Debug
cmake --build . --config Release
cd ..

git clone https://github.com/kcat/openal-soft.git
cd openal-soft
git checkout openal-soft-1.21.0
cd build
cmake .. ^
    -G "Visual Studio 16 2019" ^
    -A x64 ^
    -D LIBTYPE:STRING=STATIC ^
    -D FORCE_STATIC_VCRT=ON
msbuild OpenAL.vcxproj /property:Configuration=Debug /property:Platform="x64"
msbuild OpenAL.vcxproj /property:Configuration=RelWithDebInfo /property:Platform="x64"
cd ..\..

git clone https://github.com/google/breakpad
cd breakpad
git checkout a1dbcdcb43
git apply ../patches/breakpad.diff
cd src
git clone https://github.com/google/googletest testing
cd client\windows
gyp --no-circular-check breakpad_client.gyp --format=ninja
cd ..\..
ninja -C out/Debug_x64 common crash_generation_client exception_handler
ninja -C out/Release_x64 common crash_generation_client exception_handler
cd tools\windows\dump_syms
gyp dump_syms.gyp
msbuild dump_syms.vcxproj /property:Configuration=Release /property:Platform="x64"
cd ..\..\..\..\..

git clone https://github.com/telegramdesktop/opus.git
cd opus
git checkout tdesktop
cd win32\VS2015
msbuild opus.sln /property:Configuration=Debug /property:Platform="x64"
msbuild opus.sln /property:Configuration=Release /property:Platform="x64"

cd ..\..\..\..\..
SET PATH_BACKUP_=%PATH%
SET PATH=%cd%\ThirdParty\msys64\usr\bin;%PATH%
cd Libraries\win64

git clone https://github.com/FFmpeg/FFmpeg.git ffmpeg
cd ffmpeg
git checkout release/4.2

set CHERE_INVOKING=enabled_from_arguments
set MSYS2_PATH_TYPE=inherit
bash --login ../patches/build_ffmpeg_win.sh

SET PATH=%PATH_BACKUP_%
cd ..

SET LibrariesPath=%cd%
git clone git://code.qt.io/qt/qt5.git qt_5_15_2
cd qt_5_15_2
perl init-repository --module-subset=qtbase,qtimageformats
git checkout v5.15.2
git submodule update qtbase qtimageformats
cd qtbase
for /r %i in (..\..\patches\qtbase_5_15_2\*) do git apply %i
cd ..

configure ^
    -prefix "%LibrariesPath%\Qt-5.15.2" ^
    -debug-and-release ^
    -force-debug-info ^
    -opensource ^
    -confirm-license ^
    -static ^
    -static-runtime ^
    -no-opengl ^
    -openssl-linked ^
    -I "%LibrariesPath%\openssl_1_1_1\include" ^
    OPENSSL_LIBS_DEBUG="%LibrariesPath%\openssl_1_1_1\out64.dbg\libssl.lib %LibrariesPath%\openssl_1_1_1\out64.dbg\libcrypto.lib Ws2_32.lib Gdi32.lib Advapi32.lib Crypt32.lib User32.lib" ^
    OPENSSL_LIBS_RELEASE="%LibrariesPath%\openssl_1_1_1\out64\libssl.lib %LibrariesPath%\openssl_1_1_1\out64\libcrypto.lib Ws2_32.lib Gdi32.lib Advapi32.lib Crypt32.lib User32.lib" ^
    -I "%LibrariesPath%\mozjpeg" ^
    LIBJPEG_LIBS_DEBUG="%LibrariesPath%\mozjpeg\Debug\jpeg-static.lib" ^
    LIBJPEG_LIBS_RELEASE="%LibrariesPath%\mozjpeg\Release\jpeg-static.lib" ^
    -mp ^
    -nomake examples ^
    -nomake tests ^
    -platform win32-msvc

jom -j8
jom -j8 install
cd ..

git clone --recursive https://github.com/desktop-app/tg_owt.git
cd tg_owt
mkdir out
cd out
mkdir Debug
cd Debug
cmake -G Ninja ^
    -DCMAKE_BUILD_TYPE=Debug ^
    -DTG_OWT_SPECIAL_TARGET=win64 ^
    -DTG_OWT_LIBJPEG_INCLUDE_PATH=%cd%/../../../mozjpeg ^
    -DTG_OWT_OPENSSL_INCLUDE_PATH=%cd%/../../../openssl_1_1_1/include ^
    -DTG_OWT_OPUS_INCLUDE_PATH=%cd%/../../../opus/include ^
    -DTG_OWT_FFMPEG_INCLUDE_PATH=%cd%/../../../ffmpeg ../..
ninja
cd ..
mkdir Release
cd Release
cmake -G Ninja ^
    -DCMAKE_BUILD_TYPE=Release ^
    -DTG_OWT_SPECIAL_TARGET=win64 ^
    -DTG_OWT_LIBJPEG_INCLUDE_PATH=%cd%/../../../mozjpeg ^
    -DTG_OWT_OPENSSL_INCLUDE_PATH=%cd%/../../../openssl_1_1_1/include ^
    -DTG_OWT_OPUS_INCLUDE_PATH=%cd%/../../../opus/include ^
    -DTG_OWT_FFMPEG_INCLUDE_PATH=%cd%/../../../ffmpeg ../..
ninja
cd ..\..\..
```

## 6. Build

* cmake, pyhton27 시스템 path에 넣어야함.
* 환경변수 `QT5_DIR` : `C:\Qt\5.12.11\msvc2017_64\lib\cmake\Qt5`

`BuildPath\tdesktop\Telegram` 에서 아래를 실행

```
configure.bat -D TDESKTOP_API_ID=YOUR_API_ID -D TDESKTOP_API_HASH=YOUR_API_HASH -D DESKTOP_APP_USE_PACKAGED=OFF -D DESKTOP_APP_DISABLE_CRASH_REPORTS=OFF
```
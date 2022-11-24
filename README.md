# Proot-android

############3
Download maintained termux apk version from f-droid store (free and open source) :

https://f-droid.org/en/

Disable phantom process monitor : 

https://github.com/agnostic-apollo/Android-Docs/blob/master/en/docs/apps/processes/phantom-cached-and-empty-processes.md#commands-to-disable-phantom-process-killing-and-tldr

Install Udroid Ubuntu & Termux-x11 : 

https://github.com/RandomCoderOrg/ubuntu-on-android/discussions/152

Compile Box86 & Box64 with dynarec in ubuntu: 

https://github.com/ptitSeb/box86

https://github.com/ptitSeb/box64

https://github.com/fengxue-jrql/box86-and-box64-for-arm64/blob/main/How%20to%20set%20the%20environment

PlayOnLinux (wine) : 

https://www.playonlinux.com/wine/binaries/phoenicis/upstream-linux-x86/PlayOnLinux-wine-7.0-rc4-upstream-linux-x86.tar.gz

https://www.playonlinux.com/wine/binaries/phoenicis/upstream-linux-amd64/PlayOnLinux-wine-6.14-upstream-linux-amd64.tar.gz

############3

Install dependencies in termux (not inside proot-distro)

mkdir ~/tmp

echo Y | pkg upgrade -y

pkg install -y x11-repo; 

pkg install -y clang lld binutils cmake autoconf automake libtool '*ndk*' make python git libandroid-shmem-static 'vulkan*' ninja llvm bison flex libx11 xorgproto libdrm libpixman libxfixes libjpeg-turbo xtrans libxxf86vm xorg-xrandr xorg-font-util xorg-util-macros libxfont2 libxkbfile libpciaccess xcb-util-renderutil xcb-util-image xcb-util-keysyms xcb-util-wm xorg-xkbcomp xkeyboard-config libxdamage libxinerama

pip install meson==0.60.0

pip install mako

cd ~/tmp

LD_PRELOAD='' git clone --depth 1 https://gitlab.freedesktop.org/xorg/proto/xorgproto.git

LD_PRELOAD='' git clone --depth 1 https://gitlab.freedesktop.org/wayland/wayland.git

LD_PRELOAD='' git clone --depth 1 https://gitlab.freedesktop.org/wayland/wayland-protocols.git

LD_PRELOAD='' git clone --depth 1 -b libxshmfence-1.3 https://gitlab.freedesktop.org/xorg/lib/libxshmfence.git

LD_PRELOAD='' git clone --depth 1 -b mesa-22.0.5 https://gitlab.freedesktop.org/mesa/mesa.git

LD_PRELOAD='' git clone --depth 1 -b 1.5.10 https://github.com/anholt/libepoxy.git

LD_PRELOAD='' git clone --shallow-since 2022-06-27 https://gitlab.freedesktop.org/virgl/virglrenderer.git

LD_PRELOAD='' git clone --depth 1 https://github.com/dottedmag/libsha1.git

LD_PRELOAD='' git clone --depth 1 https://gitlab.freedesktop.org/xorg/xserver.git -b xorg-server-1.20.14 xorg-server-1.20.14

LD_PRELOAD='' git clone --depth 1 https://github.com/glmark2/glmark2.git

curl -LO https://dri.freedesktop.org/libdrm/libdrm-2.4.109.tar.xz

#################
building packages
#################

cd ~/dir/xorgproto

./autogen.sh --prefix=$PREFIX --with-xmlto=no --with-fop=no --with-xsltproc=no

make -j3 install

cd ~/dir/wayland

mkdir build

cd build

meson -Dprefix=$PREFIX -Ddocumentation=false ..

ninja -j3 install

cd ~/dir/wayland-protocols

mkdir build

cd build

meson -Dprefix=$PREFIX ..

ninja -j3 install

rm $PREFIX/lib/pkgconfig/wayland-protocols.pc

cp $PREFIX/share/pkgconfig/wayland-protocols.pc $PREFIX/lib/pkgconfig

cd ~/tmp/libxshmfence

./autogen.sh --prefix=$PREFIX --with-shared-memory-dir=$TMPDIR

sed -i s/values.h/limits.h/ ./src/xshmfence_futex.h

make -j3 install CPPFLAGS=-DMAXINT=INT_MAX

cd ~/dir

tar -xf libdrm-2.4.109.tar.xz

cd libdrm-2.4.109

mkdir build

cd build

meson -Dprefix=$PREFIX -Dintel=false -Dradeon=false -Damdgpu=false -Dnouveau=false -Dvmwgfx=false -Dvc4=false ..
ninja -j3 install

cd ~/tmp/mesa

(may not be necessary for some) applied patch https://pastebin.com/mi22qWN8

sed -i '40s+^$+#include "X11/Xlib.h"+' src/egl/main/egldisplay.h

sed -i 's/^import os$/import os, shutil\ndef link(src, dest):\n shutil.copyfile(src, dest)\ndef unlink(src):\n os.remove(src)\nos.link = link\nos.unlink = unlink/' bin/install_megadrivers.py

mkdir build

cd build

LDFLAGS='-l:libandroid-shmem.a -llog' meson .. -Dprefix=$PREFIX -Dplatforms=x11 -Dgbm=enabled -Ddri-drivers='' -Dgallium-drivers=zink,swrast -Dllvm=disabled -Dvulkan-drivers='' -Dcpp_rtti=false -Dc_args=-Wno-error=incompatible-function-pointer-types -Dbuildtype=release

rm $PREFIX/lib/libglapi.so*

rm $PREFIX/lib/libGL.so*

ninja install

cd ~/dir/libsha1

./autogen.sh --prefix=$PREFIX

make -j3 install

cd ~/tmp/libepoxy

applied patch from patches in "instructions-v2.tar.gz" : https://github.com/suhan-paradkar/tewmux-disabled/re>

in the file ~/tmp/libepoxy/src/dispatch_common.c find #elif defined(__ANDROID__) and replace it with #elif fa>

cd ~/tmp/libepoxy

mkdir build

cd build

meson -Dprefix=$PREFIX -Dbuildtype=release -Dglx=yes -Degl=yes -Dtests=false ..

rm $PREFIX/lib/libepoxy.so*

ninja install

cd ~/dir/xorg-server-1.20.14

git apply ~/instructions-v2/patches/xorg-server/xorg-server-1.20.14.patch

./autogen.sh --enable-mitshm --enable-xcsecurity --enable-xf86bigfont --enable-xwayland --enable-xorg --enable-xnest --enable-xvfb --disable-xwin --enable-xephyr --enable-kdrive --disable-devel-docs --disable-config-hal --disable-config-udev --disable-unit-tests --disable-selective-werror --disable-static --without-dtrace --disable-glamor --enable-dri --enable-dri2 --enable-dri3 --enable-glx --with-sha1=libsha1 --with-pic --prefix=$PREFIX

make install -j3 LDFLAGS=' -fuse-ld=lld /data/data/com.termux/files/usr/lib/libandroid-shmem.a -llog'

cd ~/dir/glmark2

mkdir build

cd build

meson -Dprefix=$PREFIX -Dflavors=x11-gl ..

ninja -j3 install

######33

login to proot-distro x11 with script created earlier

#############3######

run command glmark2

enjoy

# HOWTO:RPI-NOX-VC4
---
---

*Building DRM,MESA,KMSCUBE and QT5 with VC4,  without X11 for RPi3/4 32bit or 64bit.*

**This guide was written with for building as a regular 'user'. For "OPTIONAL" system installation using 'sudo', you will need your root pwd unless your admin has authorized your sudo account otherwise.**


*This guide was made for Debian aarch64 / arm64, adjust to your OS and ARCH/Bitness as needed.* 

*For Example, for Raspbian, change:*
```
  --libdir=/usr/lib/aarch64-linux-gnu 
```
 *to*
```
 --libdir=/usr/lib/arm-linux-gnueabihf    
```

---
### SETENV 
*Make a new workdir called VC4-NOX-KIT, this will be our DESTDIR rootdir, which will*
*hold everything for packaging instead of installing it directly after a build.*
*After a directed DESTDIR install you can install it locally too if desired.*



```

set -ev
mkdir -pv ~/VC4-NOX-KIT && cd ~/VC4-NOX-KIT

```


---
## DRM

```
mkdir -pv DRM && cd DRM

git clone https://gitlab.freedesktop.org/mesa/drm

cd drm

meson build \
--prefix=/usr \
--libdir=/usr/lib/aarch64-linux-gnu \
-Dlibkms=true \
-Dvmwgfx=false \
-Damdgpu=false \
-Dradeon=false \
-Dnouveau=false \
-Dfreedreno=false \
-Dvc4=true \
-Dinstall-test-programs=true

```

*Select keyboard key "enter"*

*Look for this in your output*
```
    Message: libdrm will be compiled with:

        Message:   libkms         true
        Message:   VC4 API        true
```

*Build drm* 

```
ninja -C build
```

*Install drm to ~/VC4-NOX-KIT/DRM/$git_describe_tags_DIR-rpi-vc4*

```
export DESTDIR=~/VC4-NOX-KIT/DRM/`git describe --tags --always`-rpi-vc4 
echo Destination Directory = $DESTDIR
ninja -C build install
```

*select keyboard key "enter" if needed*

*make a tar.xz*


```
cd ~/VC4-NOX-KIT/DRM 
tar cJvf `basename $DESTDIR`.tar.xz `basename $DESTDIR`
unset DESTDIR
```

*Select keyboard key "enter"*

*At this point kms and other testing tools have been installed such as:*
   * /usr/bin/kmstest         #that should show: 'vc4' ...done, then main: All ok!
   * /usr/bin/vbltest          #will show freq tests on vc4
   * /usr/bin/modetest          #will show test modes for encoders/connectors/etc
   * plus more i didnt try*


*You can inspect the installed dirs or tar.xz* 
*and/or optionally install it local too*

```
cd drm
sudo -E ninja -C build install
```


*change dir

```
cd ~/VC4-NOX-KIT 
```

*DRM section complete

---

## MESA


*Not sure which HEADER EXCLUSION METHOD is needed, so all for now*
```
CPPFLAGS=-DMESA_EGL_NO_X11_HEADERS 
CFLAGS=-DMESA_EGL_NO_X11_HEADERS
MESA_EGL_NO_X11_HEADERS=1

mkdir -pv MESA && cd MESA

git clone https://gitlab.freedesktop.org/mesa/mesa
cd mesa

meson build \
--prefix=/usr \
--libdir=/usr/lib/aarch64-linux-gnu \
-Dbuildtype=release \
-Ddri-drivers=swrast \
-Dplatforms=drm,surfaceless \
-Ddri3=true \
-Dgles1=true \
-Dgles2=true \
-Dopengl=true \
-Dgbm=true \
-Dglx=disabled \
-Dllvm=false \
-Degl=true \
-Dglx-direct=true \
-Dgallium-drivers=vc4,v3d,kmsro

```

*Observe output*

*Select "enter" key if needed.*

*Example mesa build configuration output*
```
Message: Configuration summary:
                prefix:          /usr
        libdir:          lib/aarch64-linux-gnu
        includedir:      include
        
        OpenGL:          yes (ES1: yes ES2: yes)
        OSMesa:          no
        
        DRI platform:    drm
        DRI drivers:     swrast
        DRI driver dir:  /usr/lib/aarch64-linux-gnu/dri
        
        EGL:             yes
        EGL drivers:     builtin:egl_dri2 builtin:egl_dri3
        GBM:             yes
        EGL/Vulkan/VL platforms:   drm surfaceless
        
        Vulkan drivers:  no
        
        llvm:            no
        
        Gallium drivers: vc4 v3d kmsro
        Gallium st:      mesa
        HUD lmsensors:   no
        
        Shared-glapi:    yes
```

*Build mesa

```
ninja -C build
```
*Install Mesa to ~/VC4-NOX-KIT/MESA/mesa-$git_describe_tags_DIR-rpi-vc4*
```
export DESTDIR=~/VC4-NOX-KIT/MESA/mesa-`git describe --tags --always`-rpi-vc4 
echo Destination Directory = $DESTDIR
ninja -C build install
```

*select keyboard key "enter"*

*make a tar.xz*

```
cd ~/VC4-NOX-KIT/MESA 
tar cJvf `basename $DESTDIR`.tar.xz `basename $DESTDIR`
unset DESTDIR
```


*select keyboard key "enter" if needed*
  *You can inspect the installed dirs or tar.xz* 
    *and/or optionally install it local too*

```
cd  mesa
sudo -E ninja -C build install
```
*select keyboard key "enter"*

*Go back to main build dir*

```
cd ~/VC4-NOX-KIT 
```

*MESA section complete*

---


## KMSCUBE

```
mkdir -v ~/VC4-NOX-KIT/KMSCUBE
cd ~/VC4-NOX-KIT/KMSCUBE

git clone https://gitlab.freedesktop.org/mesa/kmscube
cd kmscube

meson build \
--prefix=/usr \
--libdir=/usr/lib/aarch64-linux-gnu
```

*Select keyboard key "enter" if needed*

*Build and install it to ~/VC4-NOX-KIT/KMSCUBE/kmscube-$git_describe_tags_DIR-rpi-vc4*

```
ninja -C build
export DESTDIR=~/VC4-NOX-KIT/KMSCUBE/kmscube-`git describe --tags --always`-rpi-vc4 
echo Destination Directory = $DESTDIR
ninja -C build install
```

*select keyboard key "enter" if needed.*

*make a tar.xz*


```
cd ~/VC4-NOX-KIT/KMSCUBE
tar cJvf `basename $DESTDIR`.tar.xz `basename $DESTDIR`
unset DESTDIR
```

*Select keyboard key "enter" if needed*
  *You can inspect the installed dirs or tar.xz* 
    *and/or optionally install it local too*

~~~
cd  kmscube
sudo -E ninja -C build install
~~~

*Select keyboard key "enter" if needed*

*Test kmscube by running it and note the second section: OpenGl ES 2.X...*
*This is my latest output*
```
    OpenGL ES 2.x information:
      version: "OpenGL ES 3.0 Mesa 19.2.0-devel (git-861c2b8d31)"
      shading language version: "OpenGL ES GLSL ES 3.00"
      vendor: "Broadcom"
      renderer: "V3D 4.2"
      extensions: "GL_EXT_blend_minmax GL_EXT_multi_draw_arrays GL_EXT_t...
```


*Go back to main build dir*

```
cd ~/VC4-NOX-KIT 
```

*KMS section complete*

---

## QT-5.9.8
*Change your PREFIX/LIBDIR and/or CROSS COMPILE OPTIONS as needed, as this is setup for*
*native compiles.*

```
mkdir -v ~/VC4-NOX-KIT/QT
cd ~/VC4-NOX-KIT/QT
```

```

git clone -b 5.9.8 --single-branch https://github.com/qt/qtbase.git

```

### SET ENV
```

PREFIX=/opt/qt-5.9.8-rpi3-vc4
MESA_EGL_NO_X11_HEADERS=1
QT_EGL_NO_X11=1
PKG_CONFIG_LIBDIR=/usr/lib/aarch64-linux-gnu/pkgconfig
PKG_CONFIG_SYSROOT_DIR=/


```

*Setup a shadow build dir called **"BUILDQT5"** 
  *which makes it easy to restart.*
     **If a configuration OPTION from QT is missing,* 
        **LIFE IS SHORT, Do Not re-run it and/or risk it not picking up changes.**
           *Install your deps/ldconfig*
              **remove the **BUILDQT5** dir**
                  *recreate it then re-run qt configure again* 
 
*For qt configure help see: "qtbase/config_help.txt"*


```

mkdir BUILDQT5
cd BUILDQT5

../qtbase/configure \
-prefix ${PREFIX} \
-libdir $PREFIX/lib \
-bindir $PREFIX/bin \
-sysconfdir $PREFIX/etc/xdg \
-headerdir $PREFIX/include \
-datadir $PREFIX/share \
-archdatadir $PREFIX/lib \
-docdir ${PREFIX}/doc \
-verbose \
-no-feature-testlib \
-opensource \
-confirm-license \
-release \
-device linux-rasp-pi3-vc4-g++ \
-device-option CROSS_COMPILE=/usr/bin/ \
-reduce-exports \
-no-pch \
-no-use-gold-linker \
-sysroot / \
-make libs \
-nomake examples \
-nomake tests \
-no-compile-examples \
-dbus-linked \
-qt-doubleconversion \
-qt-pcre \
-qt-zlib \
-ssl \
-fontconfig \
-qt-freetype \
-qt-harfbuzz \
-no-gtk \
-opengl es2 \
-qpa kms \
-no-xcb-xlib \
-eglfs \
-gbm \
-kms \
-no-xcb \
-evdev \
-libinput \
-mtdev \
-tslib \
-no-xkbcommon-x11 \
-xkbcommon-evdev \
-qt-libpng \
-qt-libjpeg \
-plugin-sql-mysql \
-qt-sqlite

```


*Carefully observe output summary and make changes or fix errors/deps as needed.*
*eg, perhaps you dont like all the input support, so you remove the stanzas*

```
-libinput \

-mtdev \
```

*Clean the builddir and re-run*
*Any changes you make should make you reset the configure and re-run fresh*
**If I'm being redundant its because I want to save you un-needed time/trouble.**
*Make any adjustments needed, cd .. , remove BUILDQT5 dir, recreate BUILDQT5 *
*re-run configure statement again, corrected as needed, eg,:*
    **QUICK CLEAN cmds**

```
cd .. && rm -rfv BUILDQT5 && mkdir -v BUILDQT5 && cd BUILDQT5

```

**The above cmd will blow out your old BUILDQT5 dir and reset you up for a new config run.**

*Then, run configure again* and then *build* or *fix* *reset* *repeat* as-needed.*

**Once the configuration is perfect for you continue on and build it**

*Better start a screen to build this in, so it doesnt die over a link on you.*
*Change -jX below, to -j(NCPUS) eg, -j2 for dual core or j4 for quad core cpus, etc.*

```
screen -aX make -jX
```

### QT DEBIAN AARCH 64BIT BUG
  ### SKIP DOWN IF NOT COMPILING ON 64BIT
                                                                                                                                                                                                            

*You might get an error like so or similiar:*

```
Project ERROR: Cannot run target compiler '/usr/bin/g++'. Output:
===================
Using built-in specs.
COLLECT_GCC=/usr/bin/g++
g++: error: unrecognized command line option '-mfpu=crypto-neon-fp-armv8'
g++: error: unrecognized command line option '-mfloat-abi=hard'
Target: aarch64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Debian 8.3.0-6' --with-bugurl=file:///usr/share/doc/gcc-8/README.Bugs --enable-languages=c,ada,c++,go,d,fortran,objc,obj-c++ --prefix=/usr --with-gcc-major-version-only --program-suffix=-8 --program-prefix=aarch64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-bootstrap --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-libquadmath --disable-libquadmath-support --enable-plugin --enable-default-pie --with-system-zlib --disable-libphobos --enable-multiarch --enable-fix-cortex-a53-843419 --disable-werror --enable-checking=release --build=aarch64-linux-gnu --host=aarch64-linux-gnu --target=aarch64-linux-gnu
Thread model: posix
gcc version 8.3.0 (Debian 8.3.0-6) 
===================
Maybe you forgot to setup the environment?
```

To fix this, change the raspi3-vc4 device spec and remove the default
options designed for the arm8 in 32bit mode, along with removing the 


*Debian 64bit QT-5.9.8 "linux-rasp-pi3-vc4-g++/qmake.conf" example setup/patch* 

```
cd ../qtbase/mkspecs/devices/linux-rasp-pi3-vc4-g++/
cp -av qmake.conf qmake.conf.original
```
*inline edit qmake.conf*

```
sed -i 's/-mfpu=crypto-neon-fp-armv8//g' qmake.conf
sed -i 's/hard-float//g' qmake.conf
```

*make a patch example*
```
diff -Nuar qmake.conf.original qmake.conf > ~/VC4-NOX-KIT/QT/qt5-debian-aarch64_linux-rasp-pi3-vc4-g++_qmake.conf.patch
```

*patch contents example*

```
--- qmake.conf.original	2019-08-16 09:51:19.544346156 -0400
+++ qmake.conf	2019-08-16 11:30:51.396342596 -0400
@@ -31,10 +31,10 @@
 QMAKE_LIBS_EGL         += -lEGL
 QMAKE_LIBS_OPENGL_ES2  += -lGLESv2 -lEGL
 
-QMAKE_CFLAGS            = -march=armv8-a -mtune=cortex-a53 -mfpu=crypto-neon-fp-armv8
+QMAKE_CFLAGS            = -march=armv8-a -mtune=cortex-a53 
 QMAKE_CXXFLAGS          = $$QMAKE_CFLAGS
 
-DISTRO_OPTS            += hard-float
+DISTRO_OPTS            += 
 DISTRO_OPTS            += deb-multi-arch
 
 EGLFS_DEVICE_INTEGRATION = eglfs_kms

```



*Abstract Summary*
*When building 64bit qt-5.9.8 on debian buster and more likely, a smallish bug
in qt occurs with the default compiler options for the rasp-pi3-vc4 device
configuration. This is because it was designed for 32 bit and has not been 
updated for 64bit, this will patch it until its updated properly upstream.

*Basically its that the compiler is confused by all the mfpu/float options
that are already implied in gcc aarch64 standard specs elsewhere.
Qt had anticpated this with the trap to catch an entry for aarch64 when found
in, but its incomplete for 64bit. 

*The problem is, when softfp or hardfp is found, on aarch64; That is A NO GO
for compiler, cause it deduces already that specifying a SOFTFP option for aarch64 
is an ERR and I would agree with GCC that it should be an error or kickback at least. 
*If someone really wants to enable softfp on a modern HARDFP HW enabled arm8
they can just update all GCC specs themselves, and redefine it as needed. 
In that weird case of where some person wants to run ancient code on 
new arm processors, likely because they dont have access to the source code
to recompile. Else, they should just recompile on new hardware with hardfp, duh.

*In summary: we needed the float options removed for arm64
and for identcation of arm/aarch64/arm64 better.
*Along with NEVER specifying FP info with CFLAGS for aarch64/arm64 that intended
for mass production "cpu shared" builds or whatever its called.

"fp Enable floating-point instructions. 
This is on by default for all possible values for options -march and -mcpu."
https://gcc.gnu.org/onlinedocs/gcc-8.1.0/gcc/AArch64-Options.html


**Use armv8a because, using armv8.1-a, would work for RPi3/4 
*except that it likely would *NOT WORK on the RPi3A+*, because that is a
first gen like 'lite' arm8 64bit processor and does not include the CRC extension
 in its set. Furthermore, leaving the tune for cortex-a53 is perfect for all pi3's
and will work compatibly for the a72 rpi4 as well.
**  See: RPi3A+ details @ https://en.wikipedia.org/wiki/Raspberry_Pi



#### 64BIT BUGS END SECTION

## Continuing QT5 install section.




*Install QT5 to ~/VC4-NOX-KIT/QT/qt-$git_describe_tags_DIR-rpi-vc4*

```

export DESTDIR=~/VC4-NOX-KIT/QT/qt-`git describe --tags --always`-rpi-vc4 
echo Destination Directory = $DESTDIR
make install -INSTALL_ROOT=$DESTDIR

```

*select keyboard key "enter"*

*make a tar.xz*


```

cd ~/VC4-NOX-KIT/QT
tar cJvf `basename $DESTDIR`.tar.xz `basename $DESTDIR`
unset DESTDIR


```

* select keyboard key "enter" if needed

* You can inspect the installed dirs or tar.xz 
* and/or optionally install it local too, after DESTDIR is unset.

```

cd  qtbase
make install


```


*Go back to main qt build dir

```

cd ~/VC4-NOX-KIT/QT 

```



```

mkdir -pv QTTOOLS
cd QTTOOLS

```

#build qttools 







#Setup Env/Test and run qtdiags
#to debug set this
export QT_DEBUG_PLUGINS=1

#after building qt , if you added more options, you might need to specify your
#default qpa

export QT_QPA_EGLFS_INTEGRATION=kms-eglfs

```

/opt/qt-5.9.8-rpi-vc4/bin/qtdiags 

```
*Check to make sure  your output looks good

```
LibGLES Vendor: Broadcom
Renderer: V3D 4.2
Version: OpenGL ES 3.0 Mesa 19.2.0-devel (git-861c2b8d31)
Shading language: OpenGL ES GLSL ES 3.00
Format: Version: 3.0 Profile: 0 Swap behavior: 0 Buffer size (RGB): 8,8,8
```



## End QT5 Section





Referenced Sources:

<https://github.com/anholt/mesa/wiki>
<https://github.com/anholt/mesa/wiki/Building-Mesa-for-VC4>
<https://github.com/anholt/mesa/wiki/VC4-complete-Raspbian-upgrade>
<https://mesonbuild.com/Quick-guide.html>
<https://gcc.gnu.org/onlinedocs/gcc-8.1.0/gcc/AArch64-Options.html>
<https://wiki.gentoo.org/wiki/Safe_CFLAGS#ARM>
<https://en.wikipedia.org/wiki/Raspberry_Pi>
<https://wiki.qt.io/RaspberryPi2EGLFS>
<https://www.markdownguide.org/cheat-sheet/>

*for future use building a new kernel too
<https://github.com/sakaki-/bcm2711-kernel-bis>

*potential example uses, something like
<https://olivierschonken.github.io/first/test/update/2017/10/11/64-bit-pi-with-gallium-3D/>

*my own intentions
<https://sourceforge.net/projects/mythtv-nox/files/>

---
Tux.
                          ![Tux, the Linux mascot](/Tux-shaded.png)


---






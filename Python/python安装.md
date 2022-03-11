# Linux 下安装 python

## 1、首先要有一些 linux 下的 rpm 包

```linux
sudo yum install @development zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel xz xz-devel libffi-devel findutils expat-devel mpdecimal-devel
```

意思安装 development 这个组里面所有的包，以及后面跟着的包

```linux
yum group list就能看到这个分组
```

apt 用这个命令：

```linux
apt-get install build-essential
```

等同于@development

```linux
apt-get install build-essential zlib1g-dev bzip2 libbz2-dev libreadline6-dev libsqlite3-dev libssl-dev xz-utills libffi-dev findutils libexpat1-dev libmpdec-dev
```

## 2、编译参数

```shell
./configure --prefix=/opt/python  --disable-shared '--with-threads' '--with-computed-gotos' '--enable-optimizations'  '--enable-ipv6' '--with-system-expat' '--with-dbmliborder=gdbm:ndbm' '--with-system-ffi' '--with-system-libmpdec' '--enable-loadable-sqlite-extensions' '--without-ensurepip' 'CFLAGS=-march=x86-64 -mtune=generic -O3 -pipe' 'LDFLAGS=-Wl,-O1,--sort-common,--as-needed,-z,relro,-z,now' 'CPPFLAGS=-D_FORTIFY_SOURCE=2'
```

readline-devel 是自动补全

全的：
Configuration:
|参数|说明|
|-|-|
|–prefix=PREFIX|install architecture-independent files in PREFIX [/usr/local]|
|–exec-prefix=EPREFIX|install architecture-dependent files in EPREFIX [PREFIX]|
|–bindir=DIR|user executables [EPREFIX/bin]|
|–sbindir=DIR|system admin executables [EPREFIX/sbin]|
|–libexecdir=DIR|program executables [EPREFIX/libexec]|
|–sysconfdir=DIR|read-only single-machine data [PREFIX/etc]|
|–sharedstatedir=DIR|modifiable architecture-independent data [PREFIX/com]|
|–localstatedir=DIR|modifiable single-machine data [PREFIX/var]|
|–libdir=DIR|object code libraries [EPREFIX/lib]|
|–includedir=DIR| header files [PREFIX/include]|
|–oldincludedir=DIR|C header files for non-gcc [/usr/include]|
|–datarootdir=DIR|read-only arch.-independent data root [PREFIX/share]|
|–datadir=DIR|read-only architecture-independent data [DATAROOTDIR]|
|–infodir=DIR|info documentation [DATAROOTDIR/info]|
|–localedir=DIR|locale-dependent data [DATAROOTDIR/locale]|
|–mandir=DIR|man documentation [DATAROOTDIR/man]|
|–docdir=DIR|documentation root [DATAROOTDIR/doc/python]|
|–htmldir=DIR|html documentation [DOCDIR]|
|–dvidir=DIR|dvi documentation [DOCDIR]|
|–pdfdir=DIR|pdf documentation [DOCDIR]|
|–psdir=DIR|ps documentation [DOCDIR]|
|–build=BUILD|configure for building on BUILD [guessed]|
|–host=HOST|cross-compile to build programs to run on HOST [BUILD]|
|–target=TARGET|configure for building compilers for TARGET [HOST]|
|–disable-option-checking|ignore unrecognized --enable/–with options|
|–disable-FEATURE|do not include FEATURE (same as --enable-FEATURE=no)|
|–enable-FEATURE[=ARG]|include FEATURE [ARG=yes]|
|–enable-universalsdk[=SDKDIR]|Build fat binary against Mac OS X SDK|
|–enable-framework[=INSTALLDIR]|Build (MacOSX|
|–enable-shared|disable/enable building shared python library|
|–enable-profiling|enable C-level code profiling|
|–enable-optimizations|Enable expensive, stable optimizations (PGO, etc).Disabled by default.|
|–enable-loadable-sqlite-extensions|support loadable extensions in \_sqlite module|
|–enable-ipv6|Enable ipv6 (with ipv4) support|
|–disable-ipv6|Disable ipv6 support|
|–enable-big-digits[=BITS]|use big digits for Python longs [[BITS=30]]|
|–with-PACKAGE[=ARG]|use PACKAGE [ARG=yes]|
|–without-PACKAGE|do not use PACKAGE (same as --with-PACKAGE=no)|
|–with-universal-archs=ARCH|select architectures for universal build (“32-bit”,“64-bit”, “3-way”, “intel”, “intel-32”, “intel-64”, or “all”)|
|–with-framework-name=FRAMEWORK|specify an alternate name of the framework built with --enable-framework|
|–without-gcc|never use gcc|
|–with-icc|build with icc|
|–with-cxx-main=|compile main() and link python executable with C++ compiler|
|–with-suffix=.exe|set executable suffix|
|–with-pydebug|build with Py_DEBUG defined|
|–with-assertions|build with C assertions enabled|
|–with-ltoEnable|Link Time Optimization in any build. Disabled by default.–with-hash-algorithm=[fnvsiphash24]–with-address-sanitizerenable AddressSanitizer (asan)–with-memory-sanitizerenable MemorySanitizer (msan)–with-undefined-behavior-sanitizerenable UndefinedBehaviorSanitizer (ubsan)–with-libs=‘lib1 …’link against additional libs–with-system-expatbuild pyexpat module using an installed expat library–with-system-ffibuild \_ctypes module using an installed ffi library–with-system-libmpdecbuild \_decimal module using an installed libmpdec library–with-tcltk-includes=’-I…’override search for Tcl and Tk include files–with-tcltk-libs=’-L…’override search for Tcl and Tk libs–with-dbmliborder=db1:db2:…order to check db backends for dbm. Valid value is a colon separated string with the backend names ndbm',gdbm’ and `bdb’.–with(out)-doc-stringsdisable/enable documentation strings–with(out)-pymallocdisable/enable specialized mallocs–with(out)-c-locale-coerciondisable/enable C locale coercion to a UTF-8 based locale–with-valgrindEnable Valgrind support–with(out)-dtracedisable/enable DTrace support–with-libm=STRINGmath library–with-libc=STRINGC library–with(out)-computed-gotosUse computed gotos in evaluation loop (enabled by default on supported compilers)–with(out)-ensurepip=[=upgrade]“install” or “upgrade” using bundled pip–with-openssl=DIRroot of the OpenSSL directory–with-ssl-default-suites=[pythonopenssl|

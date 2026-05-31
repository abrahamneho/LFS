# Linux from scratch (LSF) commands in 2K26.
---

## Check whether your host system has all the appropriate versions
```bash
cat > version-check.sh << "EOF"
#!/bin/bash
# A script to list version numbers of critical development tools

# If you have tools installed in other directories, adjust PATH here AND
# in ~lfs/.bashrc (section 4.4) as well.

LC_ALL=C 
PATH=/usr/bin:/bin

bail() { echo "FATAL: $1"; exit 1; }
grep --version > /dev/null 2> /dev/null || bail "grep does not work"
sed '' /dev/null || bail "sed does not work"
sort   /dev/null || bail "sort does not work"

ver_check()
{
   if ! type -p $2 &>/dev/null
   then 
     echo "ERROR: Cannot find $2 ($1)"; return 1; 
   fi
   v=$($2 --version 2>&1 | grep -E -o '[0-9]+\.[0-9\.]+[a-z]*' | head -n1)
   if printf '%s\n' $3 $v | sort --version-sort --check &>/dev/null
   then 
     printf "OK:    %-9s %-6s >= $3\n" "$1" "$v"; return 0;
   else 
     printf "ERROR: %-9s is TOO OLD ($3 or later required)\n" "$1"; 
     return 1; 
   fi
}

ver_kernel()
{
   kver=$(uname -r | grep -E -o '^[0-9\.]+')
   if printf '%s\n' $1 $kver | sort --version-sort --check &>/dev/null
   then 
     printf "OK:    Linux Kernel $kver >= $1\n"; return 0;
   else 
     printf "ERROR: Linux Kernel ($kver) is TOO OLD ($1 or later required)\n" "$kver"; 
     return 1; 
   fi
}

# Coreutils first because --version-sort needs Coreutils >= 7.0
ver_check Coreutils      sort     8.1 || bail "Coreutils too old, stop"
ver_check Bash           bash     3.2
ver_check Binutils       ld       2.13.1
ver_check Bison          bison    2.7
ver_check Diffutils      diff     2.8.1
ver_check Findutils      find     4.2.31
ver_check Gawk           gawk     4.0.1
ver_check GCC            gcc      5.4
ver_check "GCC (C++)"    g++      5.4
ver_check Grep           grep     2.5.1a
ver_check Gzip           gzip     1.3.12
ver_check M4             m4       1.4.10
ver_check Make           make     4.0
ver_check Patch          patch    2.5.4
ver_check Perl           perl     5.8.8
ver_check Python         python3  3.4
ver_check Sed            sed      4.1.5
ver_check Tar            tar      1.22
ver_check Texinfo        texi2any 5.0
ver_check Xz             xz       5.0.0
ver_kernel 5.4

if mount | grep -q 'devpts on /dev/pts' && [ -e /dev/ptmx ]
then echo "OK:    Linux Kernel supports UNIX 98 PTY";
else echo "ERROR: Linux Kernel does NOT support UNIX 98 PTY"; fi

alias_check() {
   if $1 --version 2>&1 | grep -qi $2
   then printf "OK:    %-4s is $2\n" "$1";
   else printf "ERROR: %-4s is NOT $2\n" "$1"; fi
}
echo "Aliases:"
alias_check awk GNU
alias_check yacc Bison
alias_check sh Bash

echo "Compiler check:"
if printf "int main(){}" | g++ -x c++ -
then echo "OK:    g++ works";
else echo "ERROR: g++ does NOT work"; fi
rm -f a.out

if [ "$(nproc)" = "" ]; then
   echo "ERROR: nproc is not available or it produces empty output"
else
   echo "OK: nproc reports $(nproc) logical cores are available"
fi
EOF
```
```bash
bash version-check.sh
```
```bash
su -
```
### If any Error seems
```bash
sudo apt update
```
```bash
sudo apt install -y \
<packages_name> \
<next_packages_name>
```
---

## Start fdisk on your disk
```bash
lsblk
```
```bash
fdisk /dev/<xyz>
```
### See existing partitions
```bash
p
```
### Create a Root partition
### Create a Swap partition
### Create a Grub Bios partition
```bash
lsblk
```
---

## Create an file system
### ext4 file system
```bash
mkfs -v -t ext4 /dev/<xxx>
```
### Swap partition
```bash
mkswap /dev/<yyy>
```
---

## Setting the $LFS Variable and the Umask
### Choose a directory location
```bash
export LFS=/mnt/lfs
```
### Check that LFS
```bash
echo $LFS
```
### Set the file mode
```bash
umask 022
```
### Check umask
```bash
umask
```
---

## Create the mount point
### Create the ext4 mount point
```bash
mkdir -pv $LFS
```
```bash
mount -v -t ext4 /dev/<xxx> $LFS
```
### Set the owner and permission mode
```bash
chown root:root $LFS
```
```bash
chmod 755 $LFS
```
## Create the swap mount point
```bash
/sbin/swapon -v /dev/<zzz>
```
```bash
lsblk
```
---

## Create sources directory
```bash
mkdir -v $LFS/sources
```
### Make this directory writable and sticky
```bash
chmod -v a+wt $LFS/sources
```
---

### All packages
#### Download lists
```bash
cd $LFS/sources
```
```bash
wget https://www.linuxfromscratch.org/lfs/view/stable-systemd/wget-list-systemd
```
```bash
wget https://www.linuxfromscratch.org/lfs/view/stable-systemd/md5sums
```
```bash
ls
```
```bash
cd -
```
### Download the packages
```bash
wget --input-file=wget-list-systemd --continue --directory-prefix=$LFS/sources
```
### Verify packages
```bash
pushd $LFS/sources
  md5sum -c md5sums
popd
```
### Change the owners
```bash
chown root:root $LFS/sources/*
```
---

## Creating a Limited Directory Layout
```bash
mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}
```
```bash
for i in bin lib sbin; do
  ln -sv usr/$i $LFS/$i
done
```
```bash
case $(uname -m) in
  x86_64) mkdir -pv $LFS/lib64 ;;
esac
```
#### check x86_64 
```bash
uname -m
```
```bash
mkdir -pv $LFS/tools
```
---

## Adding the LFS User
```bash
groupadd lfs
```
```bash
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
```
### Create Password for lfs
```bash
passwd lfs
```
#### Enter new password
#### Retype new password
### Grant LFS User full access
```bash
chown -v lfs $LFS/{usr{,/*},var,etc,tools}
```
```bash
case $(uname -m) in
  x86_64) chown -v lfs $LFS/lib64 ;;
esac
```
### Login LFS User
```bash
su - lfs
```
---

## Creating two new startup files for the bash shell
```bash
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```
```bash
cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
EOF
```
### Set MAKEFLAGS
```bash
nproc
```
```bash
cat >> ~/.bashrc << "EOF"
export MAKEFLAGS=-j$(nproc)
EOF
```
```bash
source ~/.bash_profile
```
```bash
echo $MAKEFLAGS
```
---

## Compiling a Cross-Toolchain
```bash
echo $LFS
```
```bash
cd $LFS/sources
```
### Binutils-2.46.0 - Pass 1
```bash
tar -xvf binutils-2.46.0.tar.xz
```
```bash
cd binutils-2.46.0
```
```bash
mkdir -v build
cd       build
```
```bash
 time { ../configure --prefix=$LFS/tools \
             --with-sysroot=$LFS \
             --target=$LFS_TGT   \
             --disable-nls       \
             --enable-gprofng=no \
             --disable-werror    \
             --enable-new-dtags  \
             --enable-default-hash-style=gnu && make && make install; }
```
```bash
cd ../..
```
```bash
rm -rvf binutils-2.46.0
```
---

### GCC-15.2.0 - Pass 1 
```bash
tar -xvf gcc-15.2.0.tar.xz
```
```bash
cd gcc-15.2.0
```
#### Additional steps
```bash
tar -xvf ../mpfr-4.2.2.tar.xz
```
```bash
mv -v mpfr-4.2.2 mpfr
```
```bash
tar -xvf ../gmp-6.3.0.tar.xz
```
```bash
mv -v gmp-6.3.0 gmp
```
```bash
tar -xvf ../mpc-1.3.1.tar.gz
```
```bash
mv -v mpc-1.3.1 mpc
```
#### Set the default directory name for 64-bit libraries to “lib”
```bash
case $(uname -m) in
  x86_64)
    sed -e '/m64=/s/lib64/lib/' \
        -i.orig gcc/config/i386/t-linux64
 ;;
esac
```
#### steps
```bash
mkdir -v build
cd       build
```
```bash
time { ../configure                  \
    --target=$LFS_TGT         \
    --prefix=$LFS/tools       \
    --with-glibc-version=2.43 \
    --with-sysroot=$LFS       \
    --with-newlib             \
    --without-headers         \
    --enable-default-pie      \
    --enable-default-ssp      \
    --disable-nls             \
    --disable-shared          \
    --disable-multilib        \
    --disable-threads         \
    --disable-libatomic       \
    --disable-libgomp         \
    --disable-libquadmath     \
    --disable-libssp          \
    --disable-libvtv          \
    --disable-libstdcxx       \
    --enable-languages=c,c++ && make && make install; }
```
```bash
cd ..
cat gcc/limitx.h gcc/glimits.h gcc/limity.h > \
  `dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/include/limits.h
```
```bash
cd ..
```
```bash
rm -rvf gcc-15.2.0
```
### Linux-6.18.10 API Headers
```bash
tar -xvf linux-6.18.10.tar.xz
```
```bash
cd linux-6.18.10
```
```bash
make mrproper
```
```bash
make headers
```
```bash
find usr/include -type f ! -name '*.h' -delete
```
```bash
cp -rv usr/include $LFS/usr
```
```bash
cd ..
```
```bash
rm -rvf linux-6.18.10
```
### Glibc-2.43
```bash
tar -xvf glibc-2.43.tar.xz
```
```bash
cd glibc-2.43
```
```bash
case $(uname -m) in
    i?86)   ln -sfv ld-linux.so.2 $LFS/lib/ld-lsb.so.3
    ;;
    x86_64) ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64
            ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64/ld-lsb-x86-64.so.3
    ;;
esac
```
```bash
patch -Np1 -i ../glibc-fhs-1.patch
```
```bash
mkdir -v build
cd       build
```
```bash
echo "rootsbindir=/usr/sbin" > configparms
```
```bash
../configure                             \
      --prefix=/usr                      \
      --host=$LFS_TGT                    \
      --build=$(../scripts/config.guess) \
      --disable-nscd                     \
      libc_cv_slibdir=/usr/lib           \
      --enable-kernel=5.4
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
sed '/RTLDLIST=/s@/usr@@g' -i $LFS/usr/bin/ldd
```
```bash
echo 'int main(){}' | $LFS_TGT-gcc -x c - -v -Wl,--verbose &> dummy.log
readelf -l a.out | grep ': /lib'
```
`[Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]`

```bash
grep -E -o "$LFS/lib.*/S?crt[1in].*succeeded" dummy.log
```
`/mnt/lfs/lib/../lib/Scrt1.o succeeded`

`/mnt/lfs/lib/../lib/crti.o succeeded`

`/mnt/lfs/lib/../lib/crtn.o succeeded`

```bash
grep -B3 "^ $LFS/usr/include" dummy.log
```
`#include <...> search starts here:`

` /mnt/lfs/tools/lib/gcc/x86_64-lfs-linux-gnu/15.2.0/include`

` /mnt/lfs/tools/lib/gcc/x86_64-lfs-linux-gnu/15.2.0/include-fixed`

` /mnt/lfs/usr/include`
 
```bash
grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g'
```
`SEARCH_DIR("=/mnt/lfs/tools/x86_64-lfs-linux-gnu/lib64")`

`SEARCH_DIR("=/usr/local/lib64")`

`SEARCH_DIR("=/lib64")`

`SEARCH_DIR("=/usr/lib64")`

`SEARCH_DIR("=/mnt/lfs/tools/x86_64-lfs-linux-gnu/lib")`

`SEARCH_DIR("=/usr/local/lib")`

`SEARCH_DIR("=/lib")`

`SEARCH_DIR("=/usr/lib");`

```bash
grep "/lib.*/libc.so.6 " dummy.log
```
`attempt to open /mnt/lfs/usr/lib/libc.so.6 succeeded`

```bash
grep found dummy.log
```
`found ld-linux-x86-64.so.2 at /mnt/lfs/usr/lib/ld-linux-x86-64.so.2`

```bash
rm -v a.out dummy.log
```
```bash
cd ../..
```
```bash
rm -rvf glibc-2.43
```
### Libstdc++ from GCC-15.2.0
```bash
tar -xvf gcc-15.2.0.tar.xz
```
```bash
cd gcc-15.2.0
```
```bash
mkdir -v build
cd       build
```
```bash
../libstdc++-v3/configure      \
    --host=$LFS_TGT            \
    --build=$(../config.guess) \
    --prefix=/usr              \
    --disable-multilib         \
    --disable-nls              \
    --disable-libstdcxx-pch    \
    --with-gxx-include-dir=/tools/$LFS_TGT/include/c++/15.2.0
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
rm -v $LFS/usr/lib/lib{stdc++{,exp,fs},supc++}.la
```
```bash
cd ../..
```
```bash
rm -rvf gcc-15.2.0
```
---

## Cross Compiling Temporary Tools
### M4-1.4.21
```bash
tar -xvf m4-1.4.21.tar.xz
```
```bash
cd m4-1.4.21
```
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
cd ..
```
```bash
rm -rvf m4-1.4.21
```
### Ncurses-6.6
```bash
tar -xvf ncurses-6.6.tar.gz
```
```bash
cd ncurses-6.6
```
```bash
mkdir build
pushd build
  ../configure --prefix=$LFS/tools AWK=gawk
  make -C include
  make -C progs tic
  install progs/tic $LFS/tools/bin
popd
```
```bash
./configure --prefix=/usr                \
            --host=$LFS_TGT              \
            --build=$(./config.guess)    \
            --mandir=/usr/share/man      \
            --with-manpage-format=normal \
            --with-shared                \
            --without-normal             \
            --with-cxx-shared            \
            --without-debug              \
            --without-ada                \
            --disable-stripping          \
            AWK=gawk
```
```bash
make
```
```bash
make DESTDIR=$LFS install
ln -sv libncursesw.so $LFS/usr/lib/libncurses.so
sed -e 's/^#if.*XOPEN.*$/#if 1/' \
    -i $LFS/usr/include/curses.h
```
```bash
cd ..
```
```bash
rm -rvf ncurses-6.6
```
### Bash-5.3
```bash
tar -xvf bash-5.3.tar.gz
```
```bash
cd bash-5.3
```
```bash
./configure --prefix=/usr                      \
            --build=$(sh support/config.guess) \
            --host=$LFS_TGT                    \
            --without-bash-malloc
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
ln -sv bash $LFS/bin/sh
```
```bash
cd ..
```
```bash
rm -rvf bash-5.3
```
### Coreutils-9.10
```bash
tar -xvf coreutils-9.10.tar.xz
```
```bash
cd coreutils-9.10
```
```bash
./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --enable-install-program=hostname \
            --enable-no-install-program=kill,uptime
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
mv -v $LFS/usr/bin/chroot              $LFS/usr/sbin
```
```bash
mkdir -pv $LFS/usr/share/man/man8
```
```bash
mv -v $LFS/usr/share/man/man1/chroot.1 $LFS/usr/share/man/man8/chroot.8
```
```bash
sed -i 's/"1"/"8"/'                    $LFS/usr/share/man/man8/chroot.8
```
```bash
cd ..
```
```bash
rm -rvf coreutils-9.10
```
### Diffutils-3.12
```bash
tar -xvf diffutils-3.12.tar.xz 
```
```bash
cd diffutils-3.12
```
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            gl_cv_func_strcasecmp_works=y \
            --build=$(./build-aux/config.guess)
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
cd ..
```
```bash
rm -rvf diffutils-3.12
```
### File-5.46
```bash
tar -xvf file-5.46.tar.gz
```
```bash
cd file-5.46
```
```bash
mkdir build
pushd build
  ../configure --disable-bzlib      \
               --disable-libseccomp \
               --disable-xzlib      \
               --disable-zlib
  make
popd
```
```bash
./configure --prefix=/usr --host=$LFS_TGT --build=$(./config.guess)
```
```bash
make FILE_COMPILE=$(pwd)/build/src/file
```
```bash
make DESTDIR=$LFS install
```
```bash
rm -v $LFS/usr/lib/libmagic.la
```
```bash
cd ..
```
```bash
rm -rvf file-5.46
```
### Findutils-4.10.0
```bash
tar -xvf findutils-4.10.0.tar.xz
```
```bash
cd findutils-4.10.0
```
```bash
./configure --prefix=/usr                   \
            --localstatedir=/var/lib/locate \
            --host=$LFS_TGT                 \
            --build=$(build-aux/config.guess)
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
cd ..
```
```bash
rm -rvf findutils-4.10.0
```
### Gawk-5.3.2
```bash
tar -xvf gawk-5.3.2.tar.xz
```
```bash
cd gawk-5.3.2
```
```bash
sed -i 's/extras//' Makefile.in
```
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
cd ..
```
```bash
rm -rvf gawk-5.3.2
```
### Grep-3.12
```bash
tar -xvf grep-3.12.tar.xz
```
```bash
cd grep-3.12
```
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(./build-aux/config.guess)
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
cd ..
```
```bash
rm -rvf grep-3.12
```
### Gzip-1.14
```bash
tar -xvf gzip-1.14.tar.xz
```
```bash
cd gzip-1.14
```
```bash
./configure --prefix=/usr --host=$LFS_TGT
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
cd ..
```
```bash
rm -rvf gzip-1.14
```
### Make-4.4.1
```bash
tar -xvf make-4.4.1.tar.gz
```
```bash
cd make-4.4.1
```
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
cd ..
```
```bash
rm -rvf make-4.4.1
```
### Patch-2.8
```bash
tar -xvf patch-2.8.tar.xz
```
```bash
cd patch-2.8
```
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
cd ..
```
```bash
rm -rvf patch-2.8
```
### Sed-4.9
```bash
tar -xvf sed-4.9.tar.xz
```
```bash
cd sed-4.9
```
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(./build-aux/config.guess)
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
cd ..
```
```bash
rm -rvf sed-4.9
```
### Tar-1.35
```bash
tar -xvf tar-1.35.tar.xz
```
```bash
cd tar-1.35
```
```bash
./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
cd ..
```
```bash
rm -rvf tar-1.35
```
### Xz-5.8.2
```bash
tar -xvf xz-5.8.2.tar.xz
```
```bash
cd xz-5.8.2
```
```bash
./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --disable-static                  \
            --docdir=/usr/share/doc/xz-5.8.2
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
rm -v $LFS/usr/lib/liblzma.la
```
```bash
cd ..
```
```bash
rm -rvf xz-5.8.2
```
### Binutils-2.46.0 - Pass 2
```bash
tar -xvf binutils-2.46.0.tar.xz
```
```bash
cd binutils-2.46.0
```
```bash
sed '6031s/$add_dir//' -i ltmain.sh
```
```bash
mkdir -v build
cd       build
```
```bash
../configure                   \
    --prefix=/usr              \
    --build=$(../config.guess) \
    --host=$LFS_TGT            \
    --disable-nls              \
    --enable-shared            \
    --enable-gprofng=no        \
    --disable-werror           \
    --enable-64-bit-bfd        \
    --enable-new-dtags         \
    --enable-default-hash-style=gnu
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
rm -v $LFS/usr/lib/lib{bfd,ctf,ctf-nobfd,opcodes,sframe}.{a,la}
```
```bash
cd ../..
```
```bash
rm -rvf binutils-2.46.0
```
### GCC-15.2.0 - Pass 2
```bash
tar -xvf gcc-15.2.0.tar.xz
```
```bash
cd gcc-15.2.0
```
```bash
tar -xvf ../mpfr-4.2.2.tar.xz
```
```bash
mv -v mpfr-4.2.2 mpfr
```
```bash
tar -xvf ../gmp-6.3.0.tar.xz
```
```bash
mv -v gmp-6.3.0 gmp
```
```bash
tar -xvf ../mpc-1.3.1.tar.gz
```
```bash
mv -v mpc-1.3.1 mpc
```
```bash
case $(uname -m) in
  x86_64)
    sed -e '/m64=/s/lib64/lib/' \
        -i.orig gcc/config/i386/t-linux64
  ;;
esac
```
```bash
sed '/thread_header =/s/@.*@/gthr-posix.h/' \
    -i libgcc/Makefile.in libstdc++-v3/include/Makefile.in
```
```bash
mkdir -v build
cd       build
```
```bash
../configure                   \
    --build=$(../config.guess) \
    --host=$LFS_TGT            \
    --target=$LFS_TGT          \
    --prefix=/usr              \
    --with-build-sysroot=$LFS  \
    --enable-default-pie       \
    --enable-default-ssp       \
    --disable-nls              \
    --disable-multilib         \
    --disable-libatomic        \
    --disable-libgomp          \
    --disable-libquadmath      \
    --disable-libsanitizer     \
    --disable-libssp           \
    --disable-libvtv           \
    --enable-languages=c,c++   \
    LDFLAGS_FOR_TARGET=-L$PWD/$LFS_TGT/libgcc 
```
```bash
make
```
```bash
make DESTDIR=$LFS install
```
```bash
ln -sv gcc $LFS/usr/bin/cc
```
```bash
cd ../..
```
```bash
rm -rvf gcc-15.2.0
```
---

## Entering Chroot and Building Additional Temporary Tools
### Come back to root
```bash
exit
```
### Changing Ownership
```bash
echo $LFS
```
### change the ownership of the $LFS/*
```bash
chown --from lfs -R root:root $LFS/{usr,var,etc,tools}
```
```bash
case $(uname -m) in
  x86_64) chown --from lfs -R root:root $LFS/lib64 ;;
esac
```
### Creating the directories
```bash
mkdir -pv $LFS/{dev,proc,sys,run}
```
#### Mounting and Populating /dev
```bash
mount -v --bind /dev $LFS/dev
```
#### Mounting Virtual Kernel File Systems 
```bash
mount -vt devpts devpts -o gid=5,mode=0620 $LFS/dev/pts
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run
```
### Create /dev/shm & Mount a tmpfs
```bash
if [ -h $LFS/dev/shm ]; then
  install -v -d -m 1777 $LFS$(realpath /dev/shm)
else
  mount -vt tmpfs -o nosuid,nodev tmpfs $LFS/dev/shm
fi
```
### Entering the Chroot Environment
```bash
chroot "$LFS" /usr/bin/env -i   \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/usr/bin:/usr/sbin     \
    MAKEFLAGS="-j$(nproc)"      \
    TESTSUITEFLAGS="-j$(nproc)" \
    /bin/bash --login
```
### Creating Directories
```bash
mkdir -pv /{boot,home,mnt,opt,srv}
```
```bash
mkdir -pv /etc/{opt,sysconfig}
mkdir -pv /lib/firmware
mkdir -pv /media/{floppy,cdrom}
mkdir -pv /usr/{,local/}{include,src}
mkdir -pv /usr/lib/locale
mkdir -pv /usr/local/{bin,lib,sbin}
mkdir -pv /usr/{,local/}share/{color,dict,doc,info,locale,man}
mkdir -pv /usr/{,local/}share/{misc,terminfo,zoneinfo}
mkdir -pv /usr/{,local/}share/man/man{1..8}
mkdir -pv /var/{cache,local,log,mail,opt,spool}
mkdir -pv /var/lib/{color,misc,locate}
```
```bash
ln -sfv /run /var/run
ln -sfv /run/lock /var/lock
```
```bash
install -dv -m 0750 /root
install -dv -m 1777 /tmp /var/tmp
```
### Creating Essential Files and Symlinks
```bash
ln -sv /proc/self/mounts /etc/mtab
```
```bash
cat > /etc/hosts << EOF
127.0.0.1  localhost $(hostname)
::1        localhost
EOF
```
```bash
cat > /etc/passwd << "EOF"
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/dev/null:/usr/bin/false
daemon:x:6:6:Daemon User:/dev/null:/usr/bin/false
messagebus:x:18:18:D-Bus Message Daemon User:/run/dbus:/usr/bin/false
systemd-journal-gateway:x:73:73:systemd Journal Gateway:/:/usr/bin/false
systemd-journal-remote:x:74:74:systemd Journal Remote:/:/usr/bin/false
systemd-journal-upload:x:75:75:systemd Journal Upload:/:/usr/bin/false
systemd-network:x:76:76:systemd Network Management:/:/usr/bin/false
systemd-resolve:x:77:77:systemd Resolver:/:/usr/bin/false
systemd-timesync:x:78:78:systemd Time Synchronization:/:/usr/bin/false
systemd-coredump:x:79:79:systemd Core Dumper:/:/usr/bin/false
uuidd:x:80:80:UUID Generation Daemon User:/dev/null:/usr/bin/false
systemd-oom:x:81:81:systemd Out Of Memory Daemon:/:/usr/bin/false
nobody:x:65534:65534:Unprivileged User:/dev/null:/usr/bin/false
EOF
```
```bash
cat > /etc/group << "EOF"
root:x:0:
bin:x:1:daemon
sys:x:2:
kmem:x:3:
tape:x:4:
tty:x:5:
daemon:x:6:
floppy:x:7:
disk:x:8:
lp:x:9:
dialout:x:10:
audio:x:11:
video:x:12:
utmp:x:13:
clock:x:14:
cdrom:x:15:
adm:x:16:
messagebus:x:18:
systemd-journal:x:23:
input:x:24:
mail:x:34:
kvm:x:61:
systemd-journal-gateway:x:73:
systemd-journal-remote:x:74:
systemd-journal-upload:x:75:
systemd-network:x:76:
systemd-resolve:x:77:
systemd-timesync:x:78:
systemd-coredump:x:79:
uuidd:x:80:
systemd-oom:x:81:
wheel:x:97:
users:x:999:
nogroup:x:65534:
EOF
```
```bash
echo "tester:x:101:101::/home/tester:/bin/bash" >> /etc/passwd
echo "tester:x:101:" >> /etc/group
```
```bash
install -o tester -d /home/tester
```
```bash
exec /usr/bin/bash --login
```
```bash
touch /var/log/{btmp,lastlog,faillog,wtmp}
chgrp -v utmp /var/log/lastlog
chmod -v 664  /var/log/lastlog
chmod -v 600  /var/log/btmp
```
#### Move to sources
```bash
cd sources
```
### Gettext-1.0
```bash
tar -xvf gettext-1.0.tar.xz
```
```bash
cd gettext-1.0
```
```bash
./configure --disable-shared
```
```bash
make
```
```bash
cp -v gettext-tools/src/{msgfmt,msgmerge,xgettext} /usr/bin
```
```bash
cd ..
```
```bash
rm -rvf gettext-1.0
```
### Bison-3.8.2 
```bash
tar -xvf bison-3.8.2.tar.xz
```
```bash
cd bison-3.8.2
```
```bash
./configure --prefix=/usr \
            --docdir=/usr/share/doc/bison-3.8.2
```
```bash
make
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf bison-3.8.2
```
### Perl-5.42.0
```bash
tar -xvf perl-5.42.0.tar.xz
```
```bash
cd perl-5.42.0
```
```bash
sh Configure -des                                         \
             -D prefix=/usr                               \
             -D vendorprefix=/usr                         \
             -D useshrplib                                \
             -D privlib=/usr/lib/perl5/5.42/core_perl     \
             -D archlib=/usr/lib/perl5/5.42/core_perl     \
             -D sitelib=/usr/lib/perl5/5.42/site_perl     \
             -D sitearch=/usr/lib/perl5/5.42/site_perl    \
             -D vendorlib=/usr/lib/perl5/5.42/vendor_perl \
             -D vendorarch=/usr/lib/perl5/5.42/vendor_perl
```
```bash
make
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf perl-5.42.0
```
### Python-3.14.3
```bash
tar -xvf Python-3.14.3.tar.xz
```
```bash
cd Python-3.14.3
```
```bash
./configure --prefix=/usr       \
            --enable-shared     \
            --without-ensurepip \
            --without-static-libpython
```
```bash
make
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf Python-3.14.3
```
### Texinfo-7.2
```bash
tar -xvf texinfo-7.2.tar.xz
```
```bash
cd texinfo-7.2
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf texinfo-7.2
```
### Util-linux-2.41.3
```bash
tar -xvf util-linux-2.41.3.tar.xz
```
```bash
cd util-linux-2.41.3
```
```bash
mkdir -pv /var/lib/hwclock
```
```bash
./configure --libdir=/usr/lib     \
            --runstatedir=/run    \
            --disable-chfn-chsh   \
            --disable-login       \
            --disable-nologin     \
            --disable-su          \
            --disable-setpriv     \
            --disable-runuser     \
            --disable-pylibmount  \
            --disable-static      \
            --disable-liblastlog2 \
            --without-python      \
            ADJTIME_PATH=/var/lib/hwclock/adjtime \
            --docdir=/usr/share/doc/util-linux-2.41.3
```
```bash
make
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf util-linux-2.41.3
```
### Cleaning up and Saving the Temporary System
#### Cleaning
```bash
rm -rf /usr/share/{info,man,doc}/*
```
```bash
find /usr/{lib,libexec} -name \*.la -delete
```
```bash
rm -rf /tools
```
#### Backup
```bash

```
```bash

```
```bash

```
#### Restore
```bash

```
```bash

```
```bash

```
## Installing Basic System Software
### Man-pages-6.17
```bash
tar -xvf man-pages-6.17.tar.xz
```
```bash
cd man-pages-6.17
```
```bash
rm -v man3/crypt*
```
```bash
make -R GIT=false prefix=/usr install
```
```bash
cd ..
```
```bash
rm -rvf man-pages-6.17
```
### Iana-Etc-20260202
```bash
tar -xvf iana-etc-20260202.tar.gz
```
```bash
cd iana-etc-20260202
```
```bash
cp -v services protocols /etc
```
```bash
cd ..
```
```bash
rm -rvf iana-etc-20260202 
```
### Glibc-2.43
```bash
tar -xvf glibc-2.43.tar.xz
```
```bash
cd glibc-2.43
```
```bash
patch -Np1 -i ../glibc-fhs-1.patch
```
```bash
mkdir -v build
cd       build
```
```bash
echo "rootsbindir=/usr/sbin" > configparms
```
```bash
../configure --prefix=/usr                   \
             --disable-werror                \
             --disable-nscd                  \
             libc_cv_slibdir=/usr/lib        \
             --enable-stack-protector=strong \
             --enable-kernel=5.4
```
```bash
make
```
```bash
make check
```
`io/tst-lchmod is known to fail in the LFS chroot environment`

```bash
touch /etc/ld.so.conf
```
```bash
sed '/test-installation/s@$(PERL)@echo not running@' -i ../Makefile
```
```bash
make install
```
```bash
sed '/RTLDLIST=/s@/usr@@g' -i /usr/bin/ldd
```
```bash
localedef -i C -f UTF-8 C.UTF-8
localedef -i cs_CZ -f UTF-8 cs_CZ.UTF-8
localedef -i de_DE -f ISO-8859-1 de_DE
localedef -i de_DE@euro -f ISO-8859-15 de_DE@euro
localedef -i de_DE -f UTF-8 de_DE.UTF-8
localedef -i el_GR -f ISO-8859-7 el_GR
localedef -i en_GB -f ISO-8859-1 en_GB
localedef -i en_GB -f UTF-8 en_GB.UTF-8
localedef -i en_HK -f ISO-8859-1 en_HK
localedef -i en_PH -f ISO-8859-1 en_PH
localedef -i en_US -f ISO-8859-1 en_US
localedef -i en_US -f UTF-8 en_US.UTF-8
localedef -i es_ES -f ISO-8859-15 es_ES@euro
localedef -i es_MX -f ISO-8859-1 es_MX
localedef -i fa_IR -f UTF-8 fa_IR
localedef -i fr_FR -f ISO-8859-1 fr_FR
localedef -i fr_FR@euro -f ISO-8859-15 fr_FR@euro
localedef -i fr_FR -f UTF-8 fr_FR.UTF-8
localedef -i is_IS -f ISO-8859-1 is_IS
localedef -i is_IS -f UTF-8 is_IS.UTF-8
localedef -i it_IT -f ISO-8859-1 it_IT
localedef -i it_IT -f ISO-8859-15 it_IT@euro
localedef -i it_IT -f UTF-8 it_IT.UTF-8
localedef -i ja_JP -f EUC-JP ja_JP
localedef -i ja_JP -f UTF-8 ja_JP.UTF-8
localedef -i nl_NL@euro -f ISO-8859-15 nl_NL@euro
localedef -i ru_RU -f KOI8-R ru_RU.KOI8-R
localedef -i ru_RU -f UTF-8 ru_RU.UTF-8
localedef -i se_NO -f UTF-8 se_NO.UTF-8
localedef -i ta_IN -f UTF-8 ta_IN.UTF-8
localedef -i tr_TR -f UTF-8 tr_TR.UTF-8
localedef -i zh_CN -f GB18030 zh_CN.GB18030
localedef -i zh_HK -f BIG5-HKSCS zh_HK.BIG5-HKSCS
localedef -i zh_TW -f UTF-8 zh_TW.UTF-8
```
```bash
make localedata/install-locales
```
#### Configuring Glibc
```bash
cat > /etc/nsswitch.conf << "EOF"
# Begin /etc/nsswitch.conf

passwd: files systemd
group: files systemd
shadow: files systemd

hosts: mymachines resolve [!UNAVAIL=return] files myhostname dns
networks: files

protocols: files
services: files
ethers: files
rpc: files

# End /etc/nsswitch.conf
EOF
```
```bash
tar -xf ../../tzdata2025c.tar.gz
```
```bash
ZONEINFO=/usr/share/zoneinfo
mkdir -pv $ZONEINFO/{posix,right}
```
```bash
for tz in etcetera southamerica northamerica europe africa antarctica  \
          asia australasia backward; do
    zic -L /dev/null   -d $ZONEINFO       ${tz}
    zic -L /dev/null   -d $ZONEINFO/posix ${tz}
    zic -L leapseconds -d $ZONEINFO/right ${tz}
done
```
```bash
cp -v zone.tab zone1970.tab iso3166.tab $ZONEINFO
zic -d $ZONEINFO -p America/New_York
unset ZONEINFO tz
```
```bash
tzselect
```
### Select a continent
### Select a country
### Conform Yes or No
```bash
ln -sfv /usr/share/zoneinfo/<xxx> /etc/localtime
```
` Replace <xxx> with the name of the time zone selected (e.g., Asia/Kolkata)`

```bash
cat > /etc/ld.so.conf << "EOF"
# Begin /etc/ld.so.conf
/usr/local/lib
/opt/lib

EOF
```
```bash
cat >> /etc/ld.so.conf << "EOF"
# Add an include directory
include /etc/ld.so.conf.d/*.conf

EOF
```
```bash
mkdir -pv /etc/ld.so.conf.d
```
```bash
cd ../..
```
```bash
rm -rvf glibc-2.43
```
### Zlib-1.3.2
```bash
tar -xvf zlib-1.3.2.tar.gz
```
```bash
cd zlib-1.3.2
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
rm -fv /usr/lib/libz.a
```
```bash
cd ..
```
```bash
rm -rvf zlib-1.3.2
```
### Bzip2-1.0.8
```bash
tar -xvf bzip2-1.0.8.tar.gz
```
```bash
cd bzip2-1.0.8
```
```bash
patch -Np1 -i ../bzip2-1.0.8-install_docs-1.patch
```
```bash
sed -i 's@\(ln -s -f \)$(PREFIX)/bin/@\1@' Makefile
```
```bash
sed -i "s@(PREFIX)/man@(PREFIX)/share/man@g" Makefile
```
```bash
make -f Makefile-libbz2_so
```
```bash
make clean
```
```bash
make
```
```bash
make PREFIX=/usr install
```
```bash
cp -av libbz2.so.* /usr/lib
ln -sfv libbz2.so.1.0.8 /usr/lib/libbz2.so
```
```bash
ln -sfv libbz2.so.1.0.8 /usr/lib/libbz2.so.1
```
```bash
cp -v bzip2-shared /usr/bin/bzip2
for i in /usr/bin/{bzcat,bunzip2}; do
  ln -sfv bzip2 $i
done
```
```bash
rm -fv /usr/lib/libbz2.a
```
```bash
cd ..
```
```bash
rm -rvf bzip2-1.0.8
```
### Xz-5.8.2
```bash
tar -xvf xz-5.8.2.tar.xz
```
```bash
cd xz-5.8.2 
```
```bash
./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/xz-5.8.2
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf xz-5.8.2 
```
### Lz4-1.10.0
```bash
tar -xvf lz4-1.10.0.tar.gz
```
```bash
cd lz4-1.10.0
```
```bash
make BUILD_STATIC=no PREFIX=/usr
```
```bash
make -j1 check
```
```bash
make BUILD_STATIC=no PREFIX=/usr install
```
```bash
cd ..
```
```bash
rm -rvf lz4-1.10.0
```
### Zstd-1.5.7
```bash
tar -xvf zstd-1.5.7.tar.gz
```
```bash
cd zstd-1.5.7
```
```bash
make prefix=/usr
```
```bash
make check
```
```bash
make prefix=/usr install
```
```bash
rm -v /usr/lib/libzstd.a
```
```bash
cd ..
```
```bash
rm -rvf zstd-1.5.7
```
### File-5.46
```bash
tar -xvf file-5.46.tar.gz
```
```bash
cd file-5.46
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf file-5.46
```
### Readline-8.3
```bash
tar -xvf readline-8.3.tar.gz
```
```bash
cd readline-8.3
```
```bash
sed -i '/MV.*old/d' Makefile.in
sed -i '/{OLDSUFF}/c:' support/shlib-install
```
```bash
sed -i 's/-Wl,-rpath,[^ ]*//' support/shobj-conf
```
```bash
sed -e '270a\
     else\
       chars_avail = 1;'      \
    -e '288i\   result = -1;' \
    -i.orig input.c
```
```bash
./configure --prefix=/usr    \
            --disable-static \
            --with-curses    \
            --docdir=/usr/share/doc/readline-8.3
```
```bash
make SHLIB_LIBS="-lncursesw"
```
```bash
make install
```
```bash
install -v -m644 doc/*.{ps,pdf,html,dvi} /usr/share/doc/readline-8.3
```
```bash
cd ..
```
```bash
rm -rvf readline-8.3
```
### Pcre2-10.47
```bash
tar -xvf pcre2-10.47.tar.bz2
```
```bash
cd pcre2-10.47
```
```bash
./configure --prefix=/usr                       \
            --docdir=/usr/share/doc/pcre2-10.47 \
            --enable-unicode                    \
            --enable-jit                        \
            --enable-pcre2-16                   \
            --enable-pcre2-32                   \
            --enable-pcre2grep-libz             \
            --enable-pcre2grep-libbz2           \
            --enable-pcre2test-libreadline      \
            --disable-static
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf pcre2-10.47
```
### M4-1.4.21
```bash
tar -xvf m4-1.4.21.tar.xz
```
```bash
cd m4-1.4.21
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf m4-1.4.21
```
### Bc-7.0.3
```bash
tar -xvf bc-7.0.3.tar.xz
```
```bash
cd bc-7.0.3
```
```bash
CC='gcc -std=c99' ./configure --prefix=/usr -G -O3 -r
```
```bash
make
```
```bash
make test
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf bc-7.0.3
```
### Flex-2.6.4
```bash
tar -xvf flex-2.6.4.tar.gz
```
```bash
cd flex-2.6.4
```
```bash
./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/flex-2.6.4
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
ln -sv flex   /usr/bin/lex
ln -sv flex.1 /usr/share/man/man1/lex.1
```
```bash
cd ..
```
```bash
rm -rvf flex-2.6.4
```
### Tcl-8.6.17
```bash
tar -xvf tcl8.6.17-src.tar.gz
```
```bash
cd tcl8.6.17
```
```bash
SRCDIR=$(pwd)
```
```bash
cd unix
```
```bash
./configure --prefix=/usr           \
            --mandir=/usr/share/man \
            --disable-rpath
```
```bash
make
```
```bash
sed -e "s|$SRCDIR/unix|/usr/lib|" \
    -e "s|$SRCDIR|/usr/include|"  \
    -i tclConfig.sh
```
```bash
sed -e "s|$SRCDIR/unix/pkgs/tdbc1.1.12|/usr/lib/tdbc1.1.12|" \
    -e "s|$SRCDIR/pkgs/tdbc1.1.12/generic|/usr/include|"     \
    -e "s|$SRCDIR/pkgs/tdbc1.1.12/library|/usr/lib/tcl8.6|"  \
    -e "s|$SRCDIR/pkgs/tdbc1.1.12|/usr/include|"             \
    -i pkgs/tdbc1.1.12/tdbcConfig.sh
```
```bash
sed -e "s|$SRCDIR/unix/pkgs/itcl4.3.4|/usr/lib/itcl4.3.4|" \
    -e "s|$SRCDIR/pkgs/itcl4.3.4/generic|/usr/include|"    \
    -e "s|$SRCDIR/pkgs/itcl4.3.4|/usr/include|"            \
    -i pkgs/itcl4.3.4/itclConfig.sh
```
```bash
unset SRCDIR
```
```bash
LC_ALL=C.UTF-8 make test
```
```bash
make install 
chmod 644 /usr/lib/libtclstub8.6.a
```
```bash
chmod -v u+w /usr/lib/libtcl8.6.so
```
```bash
make install-private-headers
```
```bash
ln -sfv tclsh8.6 /usr/bin/tclsh
```
```bash
mv -v /usr/share/man/man3/{Thread,Tcl_Thread}.3
```
```bash
cd ..
```
```bash
tar -xf ../tcl8.6.17-html.tar.gz --strip-components=1
```
```bash
mkdir -v -p /usr/share/doc/tcl-8.6.17
```
```bash
cp -v -r  ./html/* /usr/share/doc/tcl-8.6.17
```
```bash
cd ..
```
```bash
rm -rvf tcl8.6.17
```
### Expect-5.45.4
```bash
tar -xvf expect5.45.4.tar.gz 
```
```bash
cd expect5.45.4 
```
```bash
python3 -c 'from pty import spawn; spawn(["echo", "ok"])'
```
`This command should output ok`

```bash
patch -Np1 -i ../expect-5.45.4-gcc15-1.patch
```
```bash
./configure --prefix=/usr           \
            --with-tcl=/usr/lib     \
            --enable-shared         \
            --disable-rpath         \
            --mandir=/usr/share/man \
            --with-tclinclude=/usr/include
```
```bash
make
```
```bash
make test
```
```bash
make install
```
```bash
ln -svf expect5.45.4/libexpect5.45.4.so /usr/lib
```
```bash
cd ..
```
```bash
rm -rvf expect5.45.4
```
### DejaGNU-1.6.3
```bash
tar -xvf dejagnu-1.6.3.tar.gz 
```
```bash
cd dejagnu-1.6.3 
```
```bash
mkdir -v build
cd       build
```
```bash
../configure --prefix=/usr
makeinfo --html --no-split -o doc/dejagnu.html ../doc/dejagnu.texi
makeinfo --plaintext       -o doc/dejagnu.txt  ../doc/dejagnu.texi
```
```bash
make check
```
```bash
make install
```
```bash
install -v -dm755  /usr/share/doc/dejagnu-1.6.3
install -v -m644   doc/dejagnu.{html,txt} /usr/share/doc/dejagnu-1.6.3
```
```bash
cd ../..
```
```bash
rm -rvf dejagnu-1.6.3
```
### Pkgconf-2.5.1
```bash
tar -xvf pkgconf-2.5.1.tar.xz
```
```bash
cd pkgconf-2.5.1
```
```bash
./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/pkgconf-2.5.1
```
```bash
make
```
```bash
make install
```
```bash
ln -sv pkgconf   /usr/bin/pkg-config
ln -sv pkgconf.1 /usr/share/man/man1/pkg-config.1
```
```bash
cd ..
```
```bash
rm -rvf pkgconf-2.5.1
```
### Binutils-2.46.0
```bash
tar -xvf binutils-2.46.0.tar.xz
```
```bash
cd binutils-2.46.0
```
```bash
mkdir -v build
cd       build
```
```bash
../configure --prefix=/usr       \
             --sysconfdir=/etc   \
             --enable-ld=default \
             --enable-plugins    \
             --enable-shared     \
             --disable-werror    \
             --enable-64-bit-bfd \
             --enable-new-dtags  \
             --with-system-zlib  \
             --enable-default-hash-style=gnu
```
```bash
make tooldir=/usr
```
```bash
make -k check
```
```bash
grep '^FAIL:' $(find -name '*.log')
```
`One test related to gprofng is known to fail.`

```bash
make tooldir=/usr install
```
```bash
rm -rfv /usr/lib/lib{bfd,ctf,ctf-nobfd,gprofng,opcodes,sframe}.a \
        /usr/share/doc/gprofng/
```
```bash
cd ../..
```
```bash
rm -rvf binutils-2.46.0
```
### GMP-6.3.0
```bash
tar -xvf gmp-6.3.0.tar.xz
```
```bash
cd gmp-6.3.0
```
```bash
sed -i '/long long t1;/,+1s/()/(...)/' configure
```
```bash
./configure --prefix=/usr    \
            --enable-cxx     \
            --disable-static \
            --docdir=/usr/share/doc/gmp-6.3.0
```
```bash
make
```
```bash
make html
```
```bash
make check 2>&1 | tee gmp-check-log
```
```bash
awk '/# PASS:/{total+=$3} ; END{print total}' gmp-check-log
```
```bash
make install
```
```bash
make install-html
```
```bash
cd ..
```
```bash
rm -rvf gmp-6.3.0
```
### MPFR-4.2.2
```bash
tar -xvf mpfr-4.2.2.tar.xz
```
```bash
cd mpfr-4.2.2
```
```bash
./configure --prefix=/usr        \
            --disable-static     \
            --enable-thread-safe \
            --docdir=/usr/share/doc/mpfr-4.2.2
```
```bash
make
```
```bash
make html
```
```bash
make check
```
```bash
make install
```
```bash
make install-html
```
```bash
cd ..
```
```bash
rm -rvf mpfr-4.2.2
```
### MPC-1.3.1
```bash
tar -xvf mpc-1.3.1.tar.gz
```
```bash
cd mpc-1.3.1
```
```bash
./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/mpc-1.3.1
```
```bash
make
```
```bash
make html
```
```bash
make check
```
```bash
make install
```
```bash
make install-html
```
```bash
cd ..
```
```bash
rm -rvf mpc-1.3.1
```
### Attr-2.5.2
```bash
tar -xvf attr-2.5.2.tar.gz
```
```bash
cd attr-2.5.2
```
```bash
./configure --prefix=/usr     \
            --disable-static  \
            --sysconfdir=/etc \
            --docdir=/usr/share/doc/attr-2.5.2
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf attr-2.5.2
```
### Acl-2.3.2
```bash
tar -xvf acl-2.3.2.tar.xz
```
```bash
cd acl-2.3.2
```
```bash
./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/acl-2.3.2
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf acl-2.3.2
```
### Libcap-2.77
```bash
tar -xvf libcap-2.77.tar.xz
```
```bash
cd libcap-2.77
```
```bash
sed -i '/install -m.*STA/d' libcap/Makefile
```
```bash
make prefix=/usr lib=lib
```
```bash
make test
```
```bash
make prefix=/usr lib=lib install
```
```bash
cd ..
```
```bash
rm -rvf libcap-2.77
```
### Libxcrypt-4.5.2
```bash
tar -xvf libxcrypt-4.5.2.tar.xz
```
```bash
cd libxcrypt-4.5.2
```
```bash
sed -i '/strchr/s/const//' lib/crypt-{sm3,gost}-yescrypt.c
```
```bash
./configure --prefix=/usr                \
            --enable-hashes=strong,glibc \
            --enable-obsolete-api=no     \
            --disable-static             \
            --disable-failure-tokens
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf libxcrypt-4.5.2
```
### Shadow-4.19.3
```bash
tar -xvf shadow-4.19.3.tar.xz
```
```bash
cd shadow-4.19.3
```
```bash
sed -i 's/groups$(EXEEXT) //' src/Makefile.in
find man -name Makefile.in -exec sed -i 's/groups\.1 / /'   {} \;
find man -name Makefile.in -exec sed -i 's/getspnam\.3 / /' {} \;
find man -name Makefile.in -exec sed -i 's/passwd\.5 / /'   {} \;
```
```bash
sed -e 's:#ENCRYPT_METHOD DES:ENCRYPT_METHOD YESCRYPT:' \
    -e 's:/var/spool/mail:/var/mail:'                   \
    -e '/PATH=/{s@/sbin:@@;s@/bin:@@}'                  \
    -i etc/login.defs
```
```bash
touch /usr/bin/passwd
```
```bash
./configure --sysconfdir=/etc   \
            --disable-static    \
            --with-{b,yes}crypt \
            --without-libbsd    \
            --disable-logind    \
            --with-group-name-max-length=32
```
```bash
make
```
```bash
make exec_prefix=/usr install
```
```bash
make -C man install-man
```
```bash
pwconv
```
```bash
grpconv
```
```bash
mkdir -p /etc/default
```
```bash
useradd -D --gid 999
```
```bash
sed -i '/MAIL/s/yes/no/' /etc/default/useradd
```
#### Set password
```bash
passwd root
```
#### New password
#### Re-enter new password
```bash
cd ..
```
```bash
rm -rvf shadow-4.19.3
```
### GCC-15.2.0
```bash
tar -xvf gcc-15.2.0.tar.xz
```
```bash
cd gcc-15.2.0
```
```bash
sed -i 's/char [*]q/const &/' libgomp/affinity-fmt.c
```
```bash
case $(uname -m) in
  x86_64)
    sed -e '/m64=/s/lib64/lib/' \
        -i.orig gcc/config/i386/t-linux64
  ;;
esac
```
```bash
mkdir -v build
cd       build
```
```bash
../configure --prefix=/usr            \
             LD=ld                    \
             --enable-languages=c,c++ \
             --enable-default-pie     \
             --enable-default-ssp     \
             --enable-host-pie        \
             --disable-multilib       \
             --disable-bootstrap      \
             --disable-fixincludes    \
             --with-system-zlib
```
```bash
make
```
```bash
ulimit -s -H unlimited
```
```bash
sed -e '/cpython/d' -i ../gcc/testsuite/gcc.dg/plugin/plugin.exp
```
```bash
chown -R tester .
```
```bash
su tester -c "PATH=$PATH make -k check"
```
```bash
../contrib/test_summary
```
```bash
make install
```
```bash
chown -v -R root:root \
    /usr/lib/gcc/$(gcc -dumpmachine)/15.2.0/include{,-fixed}
```
```bash
ln -svr /usr/bin/cpp /usr/lib
```
```bash
ln -sv gcc.1 /usr/share/man/man1/cc.1
```
```bash
ln -sfv ../../libexec/gcc/$(gcc -dumpmachine)/15.2.0/liblto_plugin.so \
        /usr/lib/bfd-plugins/
```
```bash
echo 'int main(){}' | cc -x c - -v -Wl,--verbose &> dummy.log
readelf -l a.out | grep ': /lib'
```
`[Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]`

```bash
grep -E -o '/usr/lib.*/S?crt[1in].*succeeded' dummy.log
```
`/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.0/../../../../lib/Scrt1.o succeeded`
`/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.0/../../../../lib/crti.o succeeded`
`/usr/lib/gcc/x86_64-pc-linux-gnu/15.2.0/../../../../lib/crtn.o succeeded`

```bash
grep -B4 '^ /usr/include' dummy.log
```
`#include <...> search starts here:`
` /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.0/include`
` /usr/local/include`
` /usr/lib/gcc/x86_64-pc-linux-gnu/15.2.0/include-fixed`
` /usr/include`
 
```bash
grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g'
```
`SEARCH_DIR("/usr/x86_64-pc-linux-gnu/lib64")`
`SEARCH_DIR("/usr/local/lib64")`
`SEARCH_DIR("/lib64")`
`SEARCH_DIR("/usr/lib64")`
`SEARCH_DIR("/usr/x86_64-pc-linux-gnu/lib")`
`SEARCH_DIR("/usr/local/lib")`
`SEARCH_DIR("/lib")`
`SEARCH_DIR("/usr/lib");`

```bash
grep "/lib.*/libc.so.6 " dummy.log
```
`attempt to open /usr/lib/libc.so.6 succeeded`

```bash
grep found dummy.log
```
`found ld-linux-x86-64.so.2 at /usr/lib/ld-linux-x86-64.so.2`

```bash
rm -v a.out dummy.log
```
```bash
mkdir -pv /usr/share/gdb/auto-load/usr/lib
```
```bash
mv -v /usr/lib/*gdb.py /usr/share/gdb/auto-load/usr/lib
```
```bash
cd ../..
```
```bash
rm -rvf gcc-15.2.0
```
### Ncurses-6.6
```bash
tar -xvf ncurses-6.6.tar.gz 
```
```bash
cd ncurses-6.6
```
```bash
./configure --prefix=/usr           \
            --mandir=/usr/share/man \
            --with-shared           \
            --without-debug         \
            --without-normal        \
            --with-cxx-shared       \
            --enable-pc-files       \
            --with-pkg-config-libdir=/usr/lib/pkgconfig
```
```bash
make
```
```bash
make DESTDIR=$PWD/dest install
```
```bash
sed -e 's/^#if.*XOPEN.*$/#if 1/' \
    -i dest/usr/include/curses.h
cp --remove-destination -av dest/* /
```
```bash
for lib in ncurses form panel menu ; do
    ln -sfv lib${lib}w.so /usr/lib/lib${lib}.so
    ln -sfv ${lib}w.pc    /usr/lib/pkgconfig/${lib}.pc
done
```
```bash
ln -sfv libncursesw.so /usr/lib/libcurses.so
```
```bash
cp -v -R doc -T /usr/share/doc/ncurses-6.6
```
```bash
cd ..
```
```bash
rm -rvf ncurses-6.6
```
### Sed-4.9
```bash
tar -xvf sed-4.9.tar.xz
```
```bash
cd sed-4.9
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make html
```
```bash
chown -R tester .
```
```bash
su tester -c "PATH=$PATH make check"
```
```bash
make install
```
```bash
install -d -m755           /usr/share/doc/sed-4.9
```
```bash
install -m644 doc/sed.html /usr/share/doc/sed-4.9
```
```bash
cd ..
```
```bash
rm -rvf sed-4.9
```
### Psmisc-23.7
```bash
tar -xvf psmisc-23.7.tar.xz
```
```bash
cd psmisc-23.7
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf psmisc-23.7
```
### Gettext-1.0
```bash
tar -xvf gettext-1.0.tar.xz
```
```bash
cd gettext-1.0 
```
```bash
./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/gettext-1.0
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
chmod -v 0755 /usr/lib/preloadable_libintl.so
```
```bash
cd ..
```
```bash
rm -rvf gettext-1.0
```
### Bison-3.8.2
```bash
tar -xvf bison-3.8.2.tar.xz
```
```bash
cd bison-3.8.2 
```
```bash
./configure --prefix=/usr --docdir=/usr/share/doc/bison-3.8.2
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf bison-3.8.2
```
### Grep-3.12
```bash
tar -xvf grep-3.12.tar.xz 
```
```bash
cd grep-3.12
```
```bash
sed -i "s/echo/#echo/" src/egrep.sh
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf grep-3.12
```
### Bash-5.3
```bash
tar -xvf bash-5.3.tar.gz
```
```bash
cd bash-5.3
```
```bash
./configure --prefix=/usr             \
            --without-bash-malloc     \
            --with-installed-readline \
            --docdir=/usr/share/doc/bash-5.3
```
```bash
make
```
```bash
chown -R tester .
```
```bash
LC_ALL=C.UTF-8 su -s /usr/bin/expect tester << "EOF"
set timeout -1
spawn make tests
expect eof
lassign [wait] _ _ _ value
exit $value
EOF
```
```bash
make install
```
```bash
exec /usr/bin/bash --login
```
```bash
cd ..
```
```bash
rm -rvf bash-5.3
```
### Libtool-2.5.4
```bash
tar -xvf libtool-2.5.4.tar.xz
```
```bash
cd libtool-2.5.4
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
rm -fv /usr/lib/libltdl.a
```
```bash
cd ..
```
```bash
rm -rvf libtool-2.5.4
```
### GDBM-1.26
```bash
tar -xvf gdbm-1.26.tar.gz
```
```bash
cd gdbm-1.26
```
```bash
./configure --prefix=/usr    \
            --disable-static \
            --enable-libgdbm-compat
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf gdbm-1.26
```
### Gperf-3.3
```bash
tar -xvf gperf-3.3.tar.gz
```
```bash
cd gperf-3.3 
```
```bash
./configure --prefix=/usr --docdir=/usr/share/doc/gperf-3.3
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf gperf-3.3
```
### Expat-2.7.4
```bash
tar -xvf expat-2.7.4.tar.xz
```
```bash
cd expat-2.7.4
```
```bash
./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/expat-2.7.4
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
install -v -m644 doc/*.{html,css} /usr/share/doc/expat-2.7.4
```
```bash
cd ..
```
```bash
rm -rvf expat-2.7.4
```
### Inetutils-2.7
```bash
tar -xvf inetutils-2.7.tar.gz
```
```bash
cd inetutils-2.7
```
```bash
sed -i 's/def HAVE_TERMCAP_TGETENT/ 1/' telnet/telnet.c
```
```bash
./configure --prefix=/usr        \
            --bindir=/usr/bin    \
            --localstatedir=/var \
            --disable-logger     \
            --disable-whois      \
            --disable-rcp        \
            --disable-rexec      \
            --disable-rlogin     \
            --disable-rsh        \
            --disable-servers
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
mv -v /usr/{,s}bin/ifconfig
```
```bash
cd ..
```
```bash
rm -rvf inetutils-2.7
```
### Less-692
```bash
tar -xvf less-692.tar.gz
```
```bash
cd less-692 
```
```bash
./configure --prefix=/usr --sysconfdir=/etc
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf less-692
```
### Perl-5.42.0
```bash
tar -xvf perl-5.42.0.tar.xz
```
```bash
cd perl-5.42.0 
```
```bash
export BUILD_ZLIB=False
```
```bash
export BUILD_BZIP2=0
```
```bash
sh Configure -des                                          \
             -D prefix=/usr                                \
             -D vendorprefix=/usr                          \
             -D privlib=/usr/lib/perl5/5.42/core_perl      \
             -D archlib=/usr/lib/perl5/5.42/core_perl      \
             -D sitelib=/usr/lib/perl5/5.42/site_perl      \
             -D sitearch=/usr/lib/perl5/5.42/site_perl     \
             -D vendorlib=/usr/lib/perl5/5.42/vendor_perl  \
             -D vendorarch=/usr/lib/perl5/5.42/vendor_perl \
             -D man1dir=/usr/share/man/man1                \
             -D man3dir=/usr/share/man/man3                \
             -D pager="/usr/bin/less -isR"                 \
             -D useshrplib                                 \
             -D usethreads
```
```bash
make
```
```bash
TEST_JOBS=$(nproc) make test_harness
```
```bash
make install
unset BUILD_ZLIB BUILD_BZIP2
```
```bash
cd ..
```
```bash
rm -rvf perl-5.42.0
```
### XML::Parser-2.47
```bash
tar -xvf XML-Parser-2.47.tar.gz
```
```bash
cd XML-Parser-2.47 
```
```bash
perl Makefile.PL
```
```bash
make
```
```bash
make test
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf XML-Parser-2.47 
```
### Intltool-0.51.0
```bash
tar -xvf intltool-0.51.0.tar.gz
```
```bash
cd intltool-0.51.0
```
```bash
sed -i 's:\\\${:\\\$\\{:' intltool-update.in
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
install -v -Dm644 doc/I18N-HOWTO /usr/share/doc/intltool-0.51.0/I18N-HOWTO
```
```bash
cd ..
```
```bash
rm -rvf intltool-0.51.0
```
### Autoconf-2.72
```bash
tar -xvf autoconf-2.72.tar.xz
```
```bash
cd autoconf-2.72
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf autoconf-2.72
```
### Automake-1.18.1
```bash
tar -xvf automake-1.18.1.tar.xz
```
```bash
cd automake-1.18.1
```
```bash
./configure --prefix=/usr --docdir=/usr/share/doc/automake-1.18.1
```
```bash
make
```
```bash
make -j$(($(nproc)>4?$(nproc):4)) check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf automake-1.18.1
```
### OpenSSL-3.6.1
```bash
tar -xvf openssl-3.6.1.tar.gz
```
```bash
cd openssl-3.6.1
```
```bash
./config --prefix=/usr         \
         --openssldir=/etc/ssl \
         --libdir=lib          \
         shared                \
         zlib-dynamic
```
```bash
make
```
```bash
HARNESS_JOBS=$(nproc) make test
```
```bash
sed -i '/INSTALL_LIBS/s/libcrypto.a libssl.a//' Makefile
make MANSUFFIX=ssl install
```
```bash
mv -v /usr/share/doc/openssl /usr/share/doc/openssl-3.6.1
```
```bash
cp -vfr doc/* /usr/share/doc/openssl-3.6.1
```
```bash
cd ..
```
```bash
rm -rvf openssl-3.6.1
```
### Libelf from Elfutils-0.194
```bash
tar -xvf elfutils-0.194.tar.bz2
```
```bash
cd elfutils-0.194
```
```bash
./configure --prefix=/usr        \
            --disable-debuginfod \
            --enable-libdebuginfod=dummy
```
```bash
make -C lib
make -C libelf
```
```bash
make -C libelf install
install -vm644 config/libelf.pc /usr/lib/pkgconfig
rm /usr/lib/libelf.a
```
```bash
cd ..
```
```bash
rm -rvf elfutils-0.194
```
### Libffi-3.5.2
```bash
tar -xvf libffi-3.5.2.tar.gz
```
```bash
cd libffi-3.5.2
```
```bash
./configure --prefix=/usr    \
            --disable-static \
            --with-gcc-arch=native
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf libffi-3.5.2
```
### Sqlite-3510200
```bash
tar -xvf sqlite-autoconf-3510200.tar.gz
```
```bash
cd sqlite-autoconf-3510200 
```
```bash
tar -xf ../sqlite-doc-3510200.tar.xz
```
```bash
./configure --prefix=/usr     \
            --disable-static  \
            --enable-fts{4,5} \
            CPPFLAGS="-D SQLITE_ENABLE_COLUMN_METADATA=1 \
                      -D SQLITE_ENABLE_UNLOCK_NOTIFY=1   \
                      -D SQLITE_ENABLE_DBSTAT_VTAB=1     \
                      -D SQLITE_SECURE_DELETE=1"
```
```bash
make LDFLAGS.rpath=""
```
```bash
make install
```
```bash
install -v -m755 -d /usr/share/doc/sqlite-3.51.2
cp -v -R sqlite-doc-3510200/* /usr/share/doc/sqlite-3.51.2
```
```bash
cd ..
```
```bash
rm -rvf sqlite-autoconf-3510200
```
### Python-3.14.3
```bash
tar -xvf Python-3.14.3.tar.xz 
```
```bash
cd Python-3.14.3 
```
```bash
./configure --prefix=/usr          \
            --enable-shared        \
            --with-system-expat    \
            --enable-optimizations \
            --without-static-libpython
```
```bash
make
```
```bash
make test TESTOPTS="--timeout 120"
```
```bash
make install
```
```bash
cat > /etc/pip.conf << EOF
[global]
root-user-action = ignore
disable-pip-version-check = true
EOF
```
```bash
install -v -dm755 /usr/share/doc/python-3.14.3/html
```
```bash
tar --strip-components=1  \
    --no-same-owner       \
    --no-same-permissions \
    -C /usr/share/doc/python-3.14.3/html \
    -xvf ../python-3.14.3-docs-html.tar.bz2
```
```bash
cd ..
```
```bash
rm -rvf Python-3.14.3
```
### Flit-Core-3.12.0
```bash
tar -xvf flit_core-3.12.0.tar.gz
```
```bash
cd flit_core-3.12.0 
```
```bash
pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD
```
```bash
pip3 install --no-index --find-links dist flit_core
```
```bash
cd ..
```
```bash
rm -rvf flit_core-3.12.0
```
### Packaging-26.0
```bash
tar -xvf packaging-26.0.tar.gz
```
```bash
cd packaging-26.0
```
```bash
pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD
```
```bash
pip3 install --no-index --find-links dist packaging
```
```bash
cd ..
```
```bash
rm -rvf packaging-26.0 
```
### Wheel-0.46.3
```bash
tar -xvf wheel-0.46.3.tar.gz
```
```bash
cd wheel-0.46.3 
```
```bash
pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD
```
```bash
pip3 install --no-index --find-links dist wheel
```
```bash
cd ..
```
```bash
rm -rvf wheel-0.46.3
```
### Setuptools-82.0.0
```bash
tar -xvf setuptools-82.0.0.tar.gz
```
```bash
cd setuptools-82.0.0 
```
```bash
pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD
```
```bash
pip3 install --no-index --find-links dist setuptools
```
```bash
cd ..
```
```bash
rm -rvf setuptools-82.0.0
```
### Ninja-1.13.2
```bash
tar -xvf ninja-1.13.2.tar.gz
```
```bash
cd ninja-1.13.2
```
```bash
export NINJAJOBS=10
```
```bash
sed -i '/int Guess/a \
  int   j = 0;\
  char* jobs = getenv( "NINJAJOBS" );\
  if ( jobs != NULL ) j = atoi( jobs );\
  if ( j > 0 ) return j;\
' src/ninja.cc
```
```bash
python3 configure.py --bootstrap --verbose
```
```bash
install -vm755 ninja /usr/bin/
install -vDm644 misc/bash-completion /usr/share/bash-completion/completions/ninja
install -vDm644 misc/zsh-completion  /usr/share/zsh/site-functions/_ninja
```
```bash
cd ..
```
```bash
rm -rvf ninja-1.13.2
```
### Meson-1.10.1
```bash
tar -xvf meson-1.10.1.tar.gz
```
```bash
cd meson-1.10.1
```
```bash
pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD
```
```bash
pip3 install --no-index --find-links dist meson
install -vDm644 data/shell-completions/bash/meson /usr/share/bash-completion/completions/meson
install -vDm644 data/shell-completions/zsh/_meson /usr/share/zsh/site-functions/_meson
```
```bash
cd ..
```
```bash
rm -rvf meson-1.10.1
```
### Kmod-34.2
```bash
tar -xvf kmod-34.2.tar.xz
```
```bash
cd kmod-34.2
```
```bash
mkdir -p build
cd       build
```
```bash
meson setup --prefix=/usr ..    \
            --buildtype=release \
            -D manpages=false
```
```bash
ninja
```
```bash
ninja install
```
```bash
cd ../..
```
```bash
rm -rvf kmod-34.2
```
### Coreutils-9.10
```bash
tar -xvf coreutils-9.10.tar.xz
```
```bash
cd coreutils-9.10
```
```bash
patch -Np1 -i ../coreutils-9.10-i18n-1.patch
```
```bash
autoreconf -fv
automake -af
FORCE_UNSAFE_CONFIGURE=1 ./configure \
            --prefix=/usr
```
```bash
make
```
```bash
make NON_ROOT_USERNAME=tester check-root
```
```bash
groupadd -g 102 dummy -U tester
```
```bash
chown -R tester . 
```
```bash
su tester -c "PATH=$PATH make -k RUN_EXPENSIVE_TESTS=yes check" \
   < /dev/null
```
```bash
groupdel dummy
```
```bash
make install
```
```bash
mv -v /usr/bin/chroot /usr/sbin
mv -v /usr/share/man/man1/chroot.1 /usr/share/man/man8/chroot.8
sed -i 's/"1"/"8"/' /usr/share/man/man8/chroot.8
```
```bash
cd ..
```
```bash
rm -rvf coreutils-9.10
```
### Diffutils-3.12
```bash
tar -xvf diffutils-3.12.tar.xz
```
```bash
cd diffutils-3.12
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf diffutils-3.12
```
### Gawk-5.3.2
```bash
tar -xvf gawk-5.3.2.tar.xz
```
```bash
cd gawk-5.3.2
```
```bash
sed -i 's/extras//' Makefile.in
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
chown -R tester .
su tester -c "PATH=$PATH make check"
```
```bash
rm -f /usr/bin/gawk-5.3.2
make install
```
```bash
ln -sv gawk.1 /usr/share/man/man1/awk.1
```
```bash
install -vDm644 doc/{awkforai.txt,*.{eps,pdf,jpg}} -t /usr/share/doc/gawk-5.3.2
```
```bash
cd ..
```
```bash
rm -rvf gawk-5.3.2
```
### Findutils-4.10.0
```bash
tar -xvf findutils-4.10.0.tar.xz
```
```bash
cd findutils-4.10.0
```
```bash
./configure --prefix=/usr --localstatedir=/var/lib/locate
```
```bash
make
```
```bash
chown -R tester .
su tester -c "PATH=$PATH make check"
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf findutils-4.10.0
```
### Groff-1.23.0
```bash
tar -xvf groff-1.23.0.tar.gz
```
```bash
cd groff-1.23.0
```
```bash
PAGE=A4 ./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf groff-1.23.0
```
### GRUB-2.14
```bash
mkdir BLFS
```
```bash
cd BLFS
```
#### Open NEW terminal tab
```bash
su -
```
```bash
cd ..
```
```bash
cd mnt/lfs/sources/BLFS
```
```bash
wget https://github.com/rhboot/efibootmgr/archive/18/efibootmgr-18.tar.gz
```
```bash
wget https://github.com/rhboot/efivar/archive/39/efivar-39.tar.gz
```
```bash
wget https://www.linuxfromscratch.org/patches/blfs/13.0/efivar-39-upstream_fixes-1.patch
```
```bash
wget https://ftp.osuosl.org/pub/rpm/popt/releases/popt-1.x/popt-1.19.tar.gz
```
```bash
wget https://downloads.sourceforge.net/freetype/freetype-2.14.1.tar.xz
```
```bash
wget https://downloads.sourceforge.net/freetype/freetype-doc-2.14.1.tar.xz
```
```bash
wget https://unifoundry.com/pub/unifont/unifont-17.0.03/font-builds/unifont-17.0.03.pcf.gz
```
#### Check MD5 sum
```bash
md5sum *
```
`e170147da25e1d5f72721ffc46fe4e06  efibootmgr-18.tar.gz`
`a8fc3e79336cd6e738ab44f9bc96a5aa  efivar-39.tar.gz`
`231bf2f0502aa256de91cae7d921c2a5  efivar-39-upstream_fixes-1.patch`
`78c7d7450fb7d0999ccd029f84094340  freetype-2.14.1.tar.xz`
`6e08cb8bcd30802a4e8e65c2eb5071cc  freetype-doc-2.14.1.tar.xz`
`eaa2135fddb6eb03f2c87ee1823e5a78  popt-1.19.tar.gz`
`b926294d5bd663027223e424a7f557e3  unifont-17.0.03.pcf.gz`

#### Back to CHROOT terminal tab
#### check list
```bash
ls
```
### efivar-39
```bash
tar -xvf efivar-39.tar.gz
```
```bash
cd efivar-39
```
```bash
patch -Np1 -i ../efivar-39-upstream_fixes-1.patch
```
```bash
make ENABLE_DOCS=0
```
```bash
make install ENABLE_DOCS=0 LIBDIR=/usr/lib
```
```bash
install -vm644 docs/efivar.1 /usr/share/man/man1 &&
install -vm644 docs/*.3      /usr/share/man/man3
```
```bash
cd ..
```
```bash
rm -rvf efivar-39
```
### Popt-1.19
```bash
tar -xvf popt-1.19.tar.gz
```
```bash
cd popt-1.19
```
```bash
./configure --prefix=/usr --disable-static &&
make
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf popt-1.19
```
### efibootmgr-18
```bash
tar -xvf efibootmgr-18.tar.gz
```
```bash
cd efibootmgr-18
```
```bash
make EFIDIR=LFS EFI_LOADER=grubx64.efi
```
```bash
make install EFIDIR=LFS
```
```bash
cd ..
```
```bash
rm -rvf efibootmgr-18
```
### FreeType-2.14.1
```bash
tar -xvf freetype-2.14.1.tar.xz
```
```bash
cd freetype-2.14.1
```
```bash
tar -xvf ../freetype-doc-2.14.1.tar.xz --strip-components=2 -C docs
```
```bash
sed -ri "s:.*(AUX_MODULES.*valid):\1:" modules.cfg &&

sed -r "s:.*(#.*SUBPIXEL_RENDERING) .*:\1:" \
    -i include/freetype/config/ftoption.h   &&

./configure --prefix=/usr            \
            --disable-static         \
            --enable-freetype-config \
            --with-harfbuzz=dynamic  &&
make
```
```bash
make install
```
```bash
cp -v -R docs -T /usr/share/doc/freetype-2.14.1 &&
rm -v /usr/share/doc/freetype-2.14.1/freetype-config.1
```
```bash
cd ..
```
```bash
rm -rvf freetype-2.14.1
```
### GRUB-2.14 for EFI
```bash
md5sum ../grub-2.14.tar.xz
```
```bash
tar -xvf ../grub-2.14.tar.xz
```
```bash
cd grub-2.14/ 
```
```bash
mkdir -pv /usr/share/fonts/unifont &&
zcat ../unifont-17.0.03.pcf.gz > /usr/share/fonts/unifont/unifont.pcf
```
```bash
./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --disable-efiemu     \
            --with-platform=efi  \
            --target=x86_64      \
            --disable-werror     &&
make
```
```bash
make install &&
mv -v /etc/bash_completion.d/grub /usr/share/bash-completion/completions
```
```bash
install -vm755 grub-mkfont /usr/bin/ &&
install -vm644 ascii.h widthspec.h *.pf2 /usr/share/grub/
```
```bash
cd ../..
```
### Gzip-1.14
```bash
tar -xvf gzip-1.14.tar.xz
```
```bash
cd gzip-1.14
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf gzip-1.14
```
### IPRoute2-6.18.0
```bash
tar -xvf iproute2-6.18.0.tar.xz
```
```bash
cd iproute2-6.18.0
```
```bash
sed -i /ARPD/d Makefile
```
```bash
rm -fv man/man8/arpd.8
```
```bash
make NETNS_RUN_DIR=/run/netns
```
```bash
make SBINDIR=/usr/sbin install
```
```bash
install -vDm644 COPYING README* -t /usr/share/doc/iproute2-6.18.0
```
```bash
cd ..
```
```bash
rm -rvf iproute2-6.18.0
```
### Kbd-2.9.0
```bash
tar -xvf kbd-2.9.0.tar.xz
```
```bash
cd kbd-2.9.0 
```
```bash
patch -Np1 -i ../kbd-2.9.0-backspace-1.patch
```
```bash
sed -i '/RESIZECONS_PROGS=/s/yes/no/' configure
sed -i 's/resizecons.8 //' docs/man/man8/Makefile.in
```
```bash
./configure --prefix=/usr --disable-vlock
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cp -R -v docs/doc -T /usr/share/doc/kbd-2.9.0
```
```bash
cd ..
```
```bash
rm -rvf kbd-2.9.0
```
### Libpipeline-1.5.8
```bash
tar -xvf libpipeline-1.5.8.tar.gz
```
```bash
cd libpipeline-1.5.8
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf libpipeline-1.5.8
```
### Make-4.4.1
```bash
tar -xvf make-4.4.1.tar.gz
```
```bash
cd make-4.4.1 
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
chown -R tester .
su tester -c "PATH=$PATH make check"
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf make-4.4.1
```
### Patch-2.8
```bash
tar -xvf patch-2.8.tar.xz
```
```bash
cd patch-2.8
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf patch-2.8
```
### Tar-1.35
```bash
tar -xvf tar-1.35.tar.xz
```
```bash
cd tar-1.35
```
```bash
FORCE_UNSAFE_CONFIGURE=1  \
./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
make -C doc install-html docdir=/usr/share/doc/tar-1.35
```
```bash
cd ..
```
```bash
rm -rvf tar-1.35
```
### Texinfo-7.2
```bash
tar -xvf texinfo-7.2.tar.xz
```
```bash
cd texinfo-7.2
```
```bash
sed 's/! $output_file eq/$output_file ne/' -i tp/Texinfo/Convert/*.pm
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
make TEXMF=/usr/share/texmf install-tex
```
```bash
pushd /usr/share/info
  rm -v dir
  for f in *
    do install-info $f dir 2>/dev/null
  done
popd
```
```bash
cd ..
```
```bash
rm -rvf texinfo-7.2
```
### Vim-9.2.0078
```bash
tar -xvf vim-9.2.0078.tar.gz
```
```bash
cd vim-9.2.0078
```
```bash
echo '#define SYS_VIMRC_FILE "/etc/vimrc"' >> src/feature.h
```
```bash
./configure --prefix=/usr
```
```bash
make
```
```bash
chown -R tester .
sed '/test_plugin_glvs/d' -i src/testdir/Make_all.mak
```
```bash
su tester -c "TERM=xterm-256color LANG=en_US.UTF-8 make -j1 test" \
   &> vim-test.log
```
```bash
less vim-test.log
```
```bash
make install
```
```bash
ln -sv ../vim/vim92/doc /usr/share/doc/vim-9.2.0078
```
```bash
cat > /etc/vimrc << "EOF"
" Begin /etc/vimrc

" Ensure defaults are set before customizing settings, not after
source $VIMRUNTIME/defaults.vim
let skip_defaults_vim=1

set nocompatible
set backspace=2
set mouse=
syntax on
if (&term == "xterm") || (&term == "putty")
  set background=dark
endif

" End /etc/vimrc
EOF
```
```bash
vim -c ':options'
```
```bash
cd ..
```
```bash
rm -rvf vim-9.2.0078
```
### MarkupSafe-3.0.3
```bash
tar -xvf markupsafe-3.0.3.tar.gz
```
```bash
cd markupsafe-3.0.3 
```
```bash
pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD
```
```bash
pip3 install --no-index --find-links dist Markupsafe
```
```bash
cd ..
```
```bash
rm -rvf markupsafe-3.0.3
```
### Jinja2-3.1.6
```bash
tar -xvf jinja2-3.1.6.tar.gz
```
```bash
cd jinja2-3.1.6
```
```bash
pip3 wheel -w dist --no-cache-dir --no-build-isolation --no-deps $PWD
```
```bash
pip3 install --no-index --find-links dist Jinja2
```
```bash
cd ..
```
```bash
rm -rvf jinja2-3.1.6
```
### Systemd-259.1
```bash
tar -xvf systemd-259.1.tar.gz
```
```bash
cd systemd-259.1
```
```bash
sed -e 's/GROUP="render"/GROUP="video"/' \
    -e 's/GROUP="sgx", //'               \
    -i rules.d/50-udev-default.rules.in
```
```bash
mkdir -p build
cd       build
```
```bash
meson setup ..                \
      --prefix=/usr           \
      --buildtype=release     \
      -D default-dnssec=no    \
      -D firstboot=false      \
      -D install-tests=false  \
      -D ldconfig=false       \
      -D sysusers=false       \
      -D rpmmacrosdir=no      \
      -D homed=disabled       \
      -D man=disabled         \
      -D mode=release         \
      -D pamconfdir=no        \
      -D dev-kvm-mode=0660    \
      -D nobody-group=nogroup \
      -D sysupdate=disabled   \
      -D ukify=disabled       \
      -D docdir=/usr/share/doc/systemd-259.1
```
```bash
ninja
```
```bash
echo 'NAME="Linux From Scratch"' > /etc/os-release
unshare -m ninja test
```
```bash
ninja install
```
```bash
tar -xf ../../systemd-man-pages-259.1.tar.xz \
    --no-same-owner --strip-components=1     \
    -C /usr/share/man
```
```bash
systemd-machine-id-setup
```
```bash
systemctl preset-all
```
```bash
cd ../..
```
```bash
rm -rvf systemd-259.1
```
### D-Bus-1.16.2
```bash
tar -xvf dbus-1.16.2.tar.xz
```
```bash
cd dbus-1.16.2
```
```bash
mkdir build
cd    build
```
```bash
meson setup --prefix=/usr --buildtype=release --wrap-mode=nofallback ..
```
```bash
ninja
```
```bash
ninja test
```
```bash
ninja install
```
```bash
ln -sfv /etc/machine-id /var/lib/dbus
```
```bash
cd ../..
```
```bash
rm -rvf dbus-1.16.2
```
### Man-DB-2.13.1
```bash
tar -xvf man-db-2.13.1.tar.xz
```
```bash
cd man-db-2.13.1
```
```bash
./configure --prefix=/usr                         \
            --docdir=/usr/share/doc/man-db-2.13.1 \
            --sysconfdir=/etc                     \
            --disable-setuid                      \
            --enable-cache-owner=bin              \
            --with-browser=/usr/bin/lynx          \
            --with-vgrind=/usr/bin/vgrind         \
            --with-grap=/usr/bin/grap
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf man-db-2.13.1
```
### Procps-ng-4.0.6
```bash
tar -xvf procps-ng-4.0.6.tar.xz
```
```bash
cd procps-ng-4.0.6
```
```bash
./configure --prefix=/usr                           \
            --docdir=/usr/share/doc/procps-ng-4.0.6 \
            --disable-static                        \
            --disable-kill                          \
            --enable-watch8bit                      \
            --with-systemd
```
```bash
make
```
```bash
chown -R tester .
su tester -c "PATH=$PATH make check"
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf procps-ng-4.0.6
```
### Util-linux-2.41.3
```bash
tar -xvf util-linux-2.41.3.tar.xz
```
```bash
cd util-linux-2.41.3
```
```bash
./configure --bindir=/usr/bin     \
            --libdir=/usr/lib     \
            --runstatedir=/run    \
            --sbindir=/usr/sbin   \
            --disable-chfn-chsh   \
            --disable-login       \
            --disable-nologin     \
            --disable-su          \
            --disable-setpriv     \
            --disable-runuser     \
            --disable-pylibmount  \
            --disable-liblastlog2 \
            --disable-static      \
            --without-python      \
            ADJTIME_PATH=/var/lib/hwclock/adjtime \
            --docdir=/usr/share/doc/util-linux-2.41.3
```
```bash
make
```
```bash
touch /etc/fstab
chown -R tester .
su tester -c "make -k check"
```
```bash
make install
```
```bash
cd ..
```
```bash
rm -rvf util-linux-2.41.3
```
### E2fsprogs-1.47.3
```bash
tar -xvf e2fsprogs-1.47.3.tar.gz
```
```bash
cd e2fsprogs-1.47.3
```
```bash
mkdir -v build
cd       build
```
```bash
../configure --prefix=/usr       \
             --sysconfdir=/etc   \
             --enable-elf-shlibs \
             --disable-libblkid  \
             --disable-libuuid   \
             --disable-uuidd     \
             --disable-fsck
```
```bash
make
```
```bash
make check
```
```bash
make install
```
```bash
rm -fv /usr/lib/{libcom_err,libe2p,libext2fs,libss}.a
```
```bash
gunzip -v /usr/share/info/libext2fs.info.gz
install-info --dir-file=/usr/share/info/dir /usr/share/info/libext2fs.info
```
```bash
makeinfo -o      doc/com_err.info ../lib/et/com_err.texinfo
install -v -m644 doc/com_err.info /usr/share/info
install-info --dir-file=/usr/share/info/dir /usr/share/info/com_err.info
```
```bash
cd ../..
```
```bash
rm -rvf e2fsprogs-1.47.3
```
### Cleaning Up
```bash
rm -rf /tmp/{*,.*}
```
```bash
find /usr/lib /usr/libexec -name \*.la -delete
```
```bash
find /usr -depth -name $(uname -m)-lfs-linux-gnu\* | xargs rm -rf
```
```bash
userdel -r tester
```
## General Network Configuration
### Network Device Naming
```bash
ip link
```
```bash
cat > /etc/systemd/network/10-eth-dhcp.network << "EOF"
[Match]
Name=wlo1

[Network]
DHCP=ipv4

[DHCPv4]
UseDomains=true
EOF
```
```bash
echo "abrahamneho" > /etc/hostname
```
```bash
cat > /etc/hosts << "EOF"
# Begin /etc/hosts

127.0.0.1 localhost
::1       ip6-localhost ip6-loopback
ff02::1   ip6-allnodes
ff02::2   ip6-allrouters

# End /etc/hosts
EOF
```
### Configuring the System Clock
#### View hardware clock
```bash
hwclock --localtime --show
```
### Configuring the System Clock
```bash
cat > /etc/adjtime << "EOF"
0.0 0 0.0
0
LOCAL
EOF
```
### Configuring the Linux Console
```bash
echo FONT=Lat2-Terminus16 > /etc/vconsole.conf
```
```bash
cat > /etc/vconsole.conf << "EOF"
KEYMAP=us
FONT=Lat2-Terminus16
EOF
```
### Configuring the System Locale
```bash
locale -a
```
```bash
LC_ALL=C.utf8 locale language
```
```bash
LC_ALL=C.utf8 locale charmap
```
```bash
LC_ALL=C.utf8 locale int_curr_symbol
```
```bash
LC_ALL=C.utf8 locale int_prefix
```
```bash
cat > /etc/locale.conf << "EOF"
LANG=C.UTF-8
EOF
```
```bash
cat > /etc/profile << "EOF"
# Begin /etc/profile

for i in $(locale); do
  unset ${i%=*}
done

if [[ "$TERM" = linux ]]; then
  export LANG=C.UTF-8
else
  source /etc/locale.conf

  for i in $(locale); do
    key=${i%=*}
    if [[ -v $key ]]; then
      export $key
    fi
  done
fi

# End /etc/profile
EOF
```
### Creating the /etc/inputrc File
```bash
cat > /etc/inputrc << "EOF"
# Begin /etc/inputrc
# Modified by Chris Lynn <roryo@roryo.dynup.net>

# Allow the command prompt to wrap to the next line
set horizontal-scroll-mode Off

# Enable 8-bit input
set meta-flag On
set input-meta On

# Turns off 8th bit stripping
set convert-meta Off

# Keep the 8th bit for display
set output-meta On

# none, visible or audible
set bell-style none

# All of the following map the escape sequence of the value
# contained in the 1st argument to the readline specific functions
"\eOd": backward-word
"\eOc": forward-word

# for linux console
"\e[1~": beginning-of-line
"\e[4~": end-of-line
"\e[5~": beginning-of-history
"\e[6~": end-of-history
"\e[3~": delete-char
"\e[2~": quoted-insert

# for xterm
"\eOH": beginning-of-line
"\eOF": end-of-line

# for Konsole
"\e[H": beginning-of-line
"\e[F": end-of-line

# End /etc/inputrc
EOF
```
### Creating the /etc/shells File
```bash
cat > /etc/shells << "EOF"
# Begin /etc/shells

/bin/sh
/bin/bash

# End /etc/shells
EOF
```
### Systemd Usage and Configuration
#### Disabling Screen Clearing at Boot Time
```bash
mkdir -pv /etc/systemd/system/getty@tty1.service.d
```
```bash
cat > /etc/systemd/system/getty@tty1.service.d/noclear.conf << EOF
[Service]
TTYVTDisallocate=no
EOF
```
#### Working with Core Dumps
```bash
mkdir -pv /etc/systemd/coredump.conf.d
```
```bash
cat > /etc/systemd/coredump.conf.d/maxuse.conf << EOF
[Coredump]
MaxUse=5G
EOF
```
### Long Running Processes
```bash
vim /etc/systemd/logind.conf
```
`KillUserProcesses=no`
## Making the LFS System Bootable
### Creating the /etc/fstab File
```bash
lsblk -f
```
```bash
cat > /etc/fstab << "EOF"
# Begin /etc/fstab

# file system       mount-point   type   options         dump  fsck
#                                                              order

/dev/nvme0n1p10     /             ext4   defaults        1    1
/dev/nvme0n1p9      /boot/efi     vfat   defaults        0    2
/dev/nvme0n1p11     swap          swap   sw              0    0

# End /etc/fstab
EOF
```
### Linux-6.18.10
```bash
tar -xvf linux-6.18.10.tar.xz
```
```bash
cd linux-6.18.10
```
```bash
make mrproper
```
```bash
make defconfig
```
```bash
make menuconfig
```
#### Be sure to enable/disable/set
`General setup --->`
`  [ ] Compile the kernel with warnings as errors                        [WERROR]`
`  CPU/Task time and stats accounting --->`
`    [*] Pressure stall information tracking                                [PSI]`
`    [ ]   Require boot parameter to enable pressure stall information tracking`
`                                                     ...  [PSI_DEFAULT_DISABLED]`
`  < > Enable kernel headers through /sys/kernel/kheaders.tar.xz      [IKHEADERS]`
`  [*] Control Group support --->                                       [CGROUPS]`
`    [*]   Memory controller                                              [MEMCG]`
`    [ /*] CPU controller --->                                     [CGROUP_SCHED]`
`      # This may cause some systemd features malfunction:`
`      [ ] Group scheduling for SCHED_RR/FIFO                    [RT_GROUP_SCHED]`
`  [ ] Configure standard kernel features (expert users) --->            [EXPERT]`

`Processor type and features --->`
`  [*] Build a relocatable kernel                                   [RELOCATABLE]`
`  [*]   Randomize the address of the kernel image (KASLR)       [RANDOMIZE_BASE]`

`General architecture-dependent options --->`
`  [*] Stack Protector buffer overflow detection                 [STACKPROTECTOR]`
`  [*]   Strong Stack Protector                           [STACKPROTECTOR_STRONG]`

`[*] Networking support --->                                                [NET]`
`  Networking options --->`
`    [*] TCP/IP networking                                                 [INET]`
`    <*>   The IPv6 protocol --->                                          [IPV6]`

`Device Drivers --->`
`  Generic Driver Options --->`
`    [ ] Support for uevent helper                                [UEVENT_HELPER]`
`    [*] Maintain a devtmpfs filesystem to mount at /dev               [DEVTMPFS]`
`    [*]   Automount devtmpfs at /dev, after the kernel mounted the rootfs`
`                                                           ...  [DEVTMPFS_MOUNT]`
`    Firmware loader --->`
`      < /*> Firmware loading facility                                [FW_LOADER]`
`      [ ]   Enable the firmware sysfs fallback mechanism [FW_LOADER_USER_HELPER]`
`  Firmware Drivers --->`
`    [*] Export DMI identification via sysfs to userspace                 [DMIID]`
`    [*] Mark VGA/VBE/EFI FB as generic system framebuffer       [SYSFB_SIMPLEFB]`
`  Graphics support --->`
`    <*>    Direct Rendering Manager (XFree86 4.1.0 and higher DRI support) --->`
`                                                                      ...  [DRM]`
`    [*]    Display a user-friendly message when a kernel panic occurs`
`                                                                ...  [DRM_PANIC]`
`    (kmsg)   Panic screen formatter                           [DRM_PANIC_SCREEN]`
`    Supported DRM clients --->`
`      [*] Enable legacy fbdev support for your modesetting driver`
`                                                      ...  [DRM_FBDEV_EMULATION]`
`    Drivers for system framebuffers --->`
`      <*> Simple framebuffer driver                              [DRM_SIMPLEDRM]`
`    Console display driver support --->`
`      [*] Framebuffer Console support                      [FRAMEBUFFER_CONSOLE]`

`File systems --->`
`  [*] Inotify support for userspace                               [INOTIFY_USER]`
`  Pseudo filesystems --->`
`    [*] Tmpfs virtual memory file system support (former shm fs)         [TMPFS]`
`    [*]   Tmpfs POSIX Access Control Lists                     [TMPFS_POSIX_ACL]`

#### Building a 64-bit system
`Processor type and features --->`
`  [*] x2APIC interrupt controller architecture support              [X86_X2APIC]`

`Device Drivers --->`
`  [*] PCI support --->                                                     [PCI]`
`    [*] Message Signaled Interrupts (MSI and MSI-X)                    [PCI_MSI]`
`  [*] IOMMU Hardware Support --->                                [IOMMU_SUPPORT]`
`    [*] Support for Interrupt Remapping                              [IRQ_REMAP]`

#### Enable NVME support
`Device Drivers --->`
`  NVME Support --->`
`    <*> NVM Express block device                                  [BLK_DEV_NVME]`

### Using GRUB to Set Up the Boot Process with UEFI
#### Kernel Configuration for UEFI support
`Processor type and features --->`
`  [*] EFI runtime service support                                          [EFI]`
`  [*]   EFI stub support                                              [EFI_STUB]`

`-*- Enable the block layer --->                                          [BLOCK]`
`  Partition Types --->`
`    [ /*] Advanced partition selection                      [PARTITION_ADVANCED]`
`    [*]     EFI GUID Partition support                           [EFI_PARTITION]`

`File systems --->`
`  DOS/FAT/EXFAT/NT Filesystems --->`
`    <*/M> VFAT (Windows-95) fs support                                 [VFAT_FS]`
`  Pseudo filesystems --->`
`    <*/M> EFI Variable filesystem                                    [EFIVAR_FS]`
`  -*- Native language support --->                                         [NLS]`
`    <*/M> Codepage 437 (United States, Canada)                [NLS_CODEPAGE_437]`
`    <*/M> NLS ISO 8859-1  (Latin 1; Western European Languages)  [NLS_ISO8859_1]`

#### Back in Linux-6.18.10
```bash
make
```
```bash
cp -iv arch/x86/boot/bzImage /boot/vmlinuz-6.18.10-lfs-13.0-systemd
```
```bash
cp -iv System.map /boot/System.map-6.18.10
```
```bash
cp -iv .config /boot/config-6.18.10
```
```bash
cp -r Documentation -T /usr/share/doc/linux-6.18.10
```
```bash
chown -R 0:0 ../linux-6.18.10
```
### Back to Using GRUB to Set Up the Boot Process with UEFI
```bash
cat >> /etc/fstab << "EOF"
/dev/nvme0n1p9 /boot/efi vfat codepage=437,iocharset=iso8859-1 0 1
EOF
```
#### Setting Up the Configuration
```bash
grub-install --bootloader-id=LFS --recheck
```
```bash
efibootmgr | cut -f 1
```
#### Creating the GRUB Configuration File
```bash
cat > /boot/grub/grub.cfg << EOF
# Begin /boot/grub/grub.cfg
set default=0
set timeout=5

insmod part_gpt
insmod ext2
set root=(hd0,gpt10)

insmod efi_gop
insmod efi_uga
if loadfont /boot/grub/fonts/unicode.pf2; then
  terminal_output gfxterm
fi

menuentry "GNU/Linux, Linux 6.18.10-lfs-13.0" {
  linux   /boot/vmlinuz-6.18.10-lfs-13.0-systemd root=/dev/nvme0n1p10 ro
}

menuentry "Firmware Setup" {
  fwsetup
}
EOF
```
## The End
```bash
echo 13.0-systemd > /etc/lfs-release
```
```bash
cat > /etc/lsb-release << "EOF"
DISTRIB_ID="Linux From Scratch"
DISTRIB_RELEASE="13.0-systemd"
DISTRIB_CODENAME="Abraham Neho"
DISTRIB_DESCRIPTION="Linux From Scratch"
EOF
```
```bash
cat > /etc/os-release << "EOF"
NAME="Linux From Scratch"
VERSION="13.0-systemd"
ID=lfs
PRETTY_NAME="Linux From Scratch 13.0-systemd"
VERSION_CODENAME="Abraham Neho"
HOME_URL="https://www.linuxfromscratch.org/lfs/"
RELEASE_TYPE="stable"
EOF
```
### Rebooting the System
```bash
logout
```
```bash
umount -v $LFS/dev/pts
mountpoint -q $LFS/dev/shm && umount -v $LFS/dev/shm
umount -v $LFS/dev
umount -v $LFS/run
umount -v $LFS/proc
umount -v $LFS/sys
```
```bash

```
```bash
umount -vR $LFS
```
```bash
reboot
```
## Result

After all steps you will have `LFS OS` ready.

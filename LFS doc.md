# Linux from scratch (LSF) commands in 2K26.
---

## Step 1 — Check whether your host system has all the appropriate versions
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

## Step 2 — Start fdisk on your disk
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
### Create a new partition
```bash
n
```
### Choose partition number
### Choose first sector
### Choose last sector
### change type
```bash
t
```
### Choose partition number
### Partition type or alias
### Write changes
```bash
w
```
```bash
lsblk
```
---

## Step 3 — Create an file system
### ext4 file system
```bash
mkfs.ext4 /dev/<xxx>
```
### boot file system
```bash
mkfs.fat -F32 /dev/<zzz>
```
### Swap partition
```bash
mkswap /dev/<yyy>
```
---

## Step 4 — Choose a directory location
### Set the variable
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

## Step 5 — Create the mount point
```bash
mount --mkdir /dev/<xxx> $LFS
```
```bash
mount --mkdir /dev/<zzz> $LFS/boot/efi
```
```bash
/sbin/swapon -v /dev/<zzz>
```
```bash
lsblk
```
### Set the owner and permission mode
```bash
chown root:root $LFS
```
```bash
chmod 755 $LFS
```
```bash
ls -la $LFS
```
---

## Step 6 — Create sources directory
```bash
mkdir -v $LFS/sources
```
### Make this directory writable and sticky
```bash
chmod -v a+wt $LFS/sources
```
---

## Step 7 — All packages
### Download lists
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
### Download the packages
```bash
wget --input-file=wget-list-systemd --continue --directory-prefix=$LFS/sources
```
```bash
cd -
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

## Step 8 — Creating a Limited Directory Layout
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
```bash
uname -m
```
```bash
mkdir -pv $LFS/tools
```
---

## Step 9 — Adding the LFS User
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

## Step 10 — Creating two new startup files for the bash shell
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
###  Set MAKEFLAGS
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

## Step 11 - Compiling a Cross-Toolchain
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
```
```bash
readelf -l a.out | grep ': /lib'
```
*[Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]*
```bash
grep -E -o "$LFS/lib.*/S?crt[1in].*succeeded" dummy.log
```
```bash
grep -B3 "^ $LFS/usr/include" dummy.log
```
```bash
grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g'
```
```bash
grep "/lib.*/libc.so.6 " dummy.log
```
```bash
grep found dummy.log
```
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
## Step - 12 Cross Compiling Temporary Tools
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
```
```bash
ln -sv libncursesw.so $LFS/usr/lib/libncurses.so
```
```bash
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
cd..
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
## Step 13 - Entering Chroot and Building Additional Temporary Tools
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
chown --from <user_name> -R root:root $LFS/{usr,var,etc,tools}
```
```bash
case $(uname -m) in
  x86_64) chown --from <user_name> -R root:root $LFS/lib64 ;;
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
```bash
cd sources
```
### Gettext-1.0
```bash
tar - xvf gettext-1.0.tar.xz
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
## Step 14 - Cleaning up and Saving the Temporary System
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
## Step 13 - Installing Basic System Software
### Man-pages-6.17
```bash
tar
```
```bash
cd
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
rm
```
### Iana-Etc-20260202
```bash
tar
```
```bash
cd 
```
```bash
cp -v services protocols /etc
```
```bash
cd ..
```
```bash
rm 
```
### Glibc-2.43
```bash
tar 
```
```bash
cd
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
```bash

```
```bash

```
```bash

```
```bash
ln -sfv /usr/share/zoneinfo/<xxx> /etc/localtime
```
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
rm
```
### Zlib-1.3.2
```bash
tar 
```
```bash
cd 
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
rm
```
### Bzip2-1.0.8
```bash
tar
```
```bash
cd 
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
```
```bash
ln -sfv libbz2.so.1.0.8 /usr/lib/libbz2.so
```
```bash
ln -sfv libbz2.so.1.0.8 /usr/lib/libbz2.so.1
```
```bash
cp -v bzip2-shared /usr/bin/bzip2
```
```bash
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
rm 
```
### Xz-5.8.2
```bash
tar
```
```bash
cd 
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
rm 
```
### Lz4-1.10.0
```bash
tar
```
```bash
cd
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
rm
```
### Zstd-1.5.7
```bash
tar
```
```bash
cd
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
rm
```
### File-5.46
```bash
tar
```
```bash
cd
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
rm
```
### Readline-8.3
```bash
tar
```
```bash
cd
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
rm
```
### Pcre2-10.47
```bash
tar
```
```bash
cd
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
rm
```
### M4-1.4.21
```bash
tar
```
```bash
cd
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
rm
```
### Bc-7.0.3
```bash
tar
```
```bash
cd
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
rm
```
### Flex-2.6.4
```bash
tar
```
```bash
cd
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
rm
```
### Tcl-8.6.17
```bash
tar
```
```bash
cd
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
rm
```
### Expect-5.45.4
```bash
tar
```
```bash
cd 
```
```bash
python3 -c 'from pty import spawn; spawn(["echo", "ok"])'
```
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
rm
```
### DejaGNU-1.6.3
```bash
tar 
```
```bash
cd 
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
cd ..
```
```bash
rm
```
### Pkgconf-2.5.1
```bash
tar
```
```bash
cd
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
rm
```
### Binutils-2.46.0
```bash
tar
```
```bash
cd
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
```bash
make tooldir=/usr install
```
```bash
rm -rfv /usr/lib/lib{bfd,ctf,ctf-nobfd,gprofng,opcodes,sframe}.a \
        /usr/share/doc/gprofng/
```
```bash
cd ..
```
```bash
rm
```
### GMP-6.3.0
```bash
tar
```
```bash
cd
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
rm
```
### MPFR-4.2.2
```bash
tar
```
```bash
cd
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
rm
```
### MPC-1.3.1
```bash
tar
```
```bash
cd
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
rm
```
### Attr-2.5.2
```bash
tar
```
```bash
cd
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
rm
```
### Acl-2.3.2
```bash
tar
```
```bash
cd
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
rm
```
### Libcap-2.77
```bash
tar
```
```bash
cd
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
rm
```
### Libxcrypt-4.5.2
```bash
tar
```
```bash
cd
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
rm
```
### Shadow-4.19.3
```bash
tar
```
```bash
cd
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
rm
```
### GCC-15.2.0
```bash
tar
```
```bash
cd
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
```bash
grep -E -o '/usr/lib.*/S?crt[1in].*succeeded' dummy.log
```
```bash
grep -B4 '^ /usr/include' dummy.log
```
```bash
grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g'
```
```bash
grep "/lib.*/libc.so.6 " dummy.log
```
```bash
grep found dummy.log
```
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
rm
```
### Ncurses-6.6
```bash
tar
```
```bash
cd
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
rm
```
### Sed-4.9
```bash
tar
```
```bash
cd
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
rm
```
### Psmisc-23.7
```bash
tar
```
```bash
cd
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
rm
```
### Gettext-1.0
```bash
tar
```
```bash
cd 
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
rm
```
### Bison-3.8.2
```bash
tar
```
```bash
cd 
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
rm
```
### Grep-3.12
```bash
tar 
```
```bash
cd
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
rm
```
### Bash-5.3
```bash
tar
```
```bash
cd
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
rm
```
### Libtool-2.5.4
```bash
tar
```
```bash
cd
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
rm
```
### GDBM-1.26
```bash
tar
```
```bash
cd
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
rm
```
### Gperf-3.3
```bash
tar
```
```bash
cd 
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
rm
```
### Expat-2.7.4
```bash
tar
```
```bash
cd
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
rm
```
### Inetutils-2.7
```bash
tar
```
```bash
cd
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
rm
```
### Less-692
```bash
tar
```
```bash
cd 
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
rm
```
### Perl-5.42.0
```bash
tar
```
```bash
cd 
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
rm
```
### XML::Parser-2.47
```bash
tar
```
```bash
cd 
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
rm 
```
### Intltool-0.51.0
```bash
tar
```
```bash
cd
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
rm
```
### Autoconf-2.72
```bash
tar
```
```bash
cd
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
rm
```
### Automake-1.18.1
```bash
tar
```
```bash
cd
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
rm
```
### OpenSSL-3.6.1
```bash
tar
```
```bash
cd
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
rm
```
### Libelf from Elfutils-0.194
```bash
tar
```
```bash
cd
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
rm
```
### Libffi-3.5.2
```bash
tar
```
```bash
cd
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
rm
```
### Sqlite-3510200
```bash
tar
```
```bash
cd 
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
rm
```
### Python-3.14.3
```bash
tar 
```
```bash
cd 
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
rm
```
### Flit-Core-3.12.0
```bash
tar
```
```bash
cd 
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
rm
```
### Packaging-26.0
```bash
tar
```
```bash
cd
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
rm
```
### Wheel-0.46.3
```bash
tar
```
```bash
cd 
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
rm
```
### Setuptools-82.0.0
```bash
tar
```
```bash
cd 
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
rm
```
### Ninja-1.13.2
```bash
tar
```
```bash
cd
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
rm
```
### Meson-1.10.1
```bash
tar
```
```bash
cd
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
rm
```
### Kmod-34.2
```bash
tar
```
```bash
cd
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
rm
```
### Coreutils-9.10
```bash
tar
```
```bash
cd
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
rm
```
### Diffutils-3.12
```bash
tar
```
```bash
cd
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
rm
```
### Gawk-5.3.2
```bash
tar
```
```bash
cd
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
rm
```
### Findutils-4.10.0
```bash
tar
```
```bash
cd
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
rm
```
### Groff-1.23.0
```bash
tar
```
```bash
cd
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
rm
```
### GRUB-2.14
```bash
mkdir BLFS
```
```bash
cd BLFS
```
#### Ctrl+A then D >>> to Detach from session
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
wget https://ftp.osuosl.org/pub/rpm/popt/releases/popt-1.x/popt-1.19.tar.gz
```
```bash
wget https://downloads.sourceforge.net/freetype/freetype-2.14.1.tar.xz
```
#### Check MD5 sum
```bash
md5sum *
```
#### Reattach session
```bash
screen -r lfs
```
#### check list
```bash
ls
```
### efivar-39
```bash
tar
```
```bash
cd
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
rm
```
### Popt-1.19
```bash
tar
```
```bash
cd
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
rm
```
### efibootmgr-18
```bash
tar
```
```bash
cd
```
```bash
make EFIDIR=LFS EFI_LOADER=grubx64.efi
```
```bash
make install EFIDIR=LFS
```
```bash

```
```bash

```
### FreeType-2.14.1
```bash
tar
```
```bash
cd
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
cd ..
```
```bash
rm
```
### GRUB-2.14 for EFI
```bash
md5sum ../grub-2.14.tar.xz
```
```bash
tar xvf ../gr
```
```bash
cd 
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
tar
```
```bash
cd
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
rm
```
### IPRoute2-6.18.0
```bash
tar
```
```bash
cd
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
rm
```
### Kbd-2.9.0
```bash
tar
```
```bash
cd 
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
rm
```
### Libpipeline-1.5.8
```bash
tar
```
```bash
cd
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
rm
```
### Make-4.4.1
```bash
tar
```
```bash
cd 
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
rm
```
### Patch-2.8
```bash
tsr
```
```bash
cd
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
rm
```
### Tar-1.35
```bash
tar
```
```bash
cd
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
rm
```
### Texinfo-7.2
```bash
tar
```
```bash
cd
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
rm
```
### Vim-9.2.0078
```bash
tar
```
```bash
cd
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

```
```bash
cd ..
```
```bash
rm
```
### MarkupSafe-3.0.3
```bash
tar
```
```bash
cd 
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
rm
```
### Jinja2-3.1.6
```bash
tar
```
```bash
cd
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
rm
```
### Systemd-259.1
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

```
```bash

```
### D-Bus-1.16.2
```bash
tar
```
```bash
cd
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
cd ..
```
```bash
rm
```
### Man-DB-2.13.1
```bash
tsr
```
```bash
cd
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
rm
```
### Procps-ng-4.0.6
```bash
tar
```
```bash
cd
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
rm
```
### Util-linux-2.41.3
```bash
tar
```
```bash
cd
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
rm
```
### E2fsprogs-1.47.3
```bash
tar
```
```bash
cd
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

```
```bash
cd ..
```
```bash
rm
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
## Step -   General Network Configuration
### Network Device Naming
```bash
systemctl disable systemd-networkd-wait-online
```
```bash
ip link
```
```bash
cat > /etc/systemd/network/10-eth-dhcp.network << "EOF"
[Match]
Name=

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
```bash

```
```bash

```
```bash

```
### Configuring the System Clock
```bash

```
```bash

```
```bash

```
### Configuring the Linux Console
```bash

```
```bash

```
```bash

```
### Configuring the System Locale
```bash

```
```bash

```
```bash

```
### Creating the /etc/inputrc File
```bash

```
### Creating the /etc/shells File
```bash

```
### Systemd Usage and Configuration
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
```bash

```
## Result

After all steps you will have `LFS OS` ready.

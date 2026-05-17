# Linux from scratch (LSF) commands in 2K26.
---

## Requirements

- Any Linux Distro
- 4GB RAM minimum
- 50GB free disk space
- Internet connection

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
### If any Error seems
```bash
sudo apt update

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
fdisk /dev/<xxxx>
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
mkfs -v -t ext4 /dev/<xxx>
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

### Enable swap partition
```bash
/sbin/swapon -v /dev/<zzz>
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

### Download the packages
```bash
wget -P $LFS/sources https://www.linuxfromscratch.org/lfs/view/stable-systemd/wget-list-systemd
```
```bash
wget --input-file=wget-list-systemd --continue --directory-prefix=$LFS/sources
```
### Verify packages
```bash
wget -P $LFS/sources https://www.linuxfromscratch.org/lfs/view/stable-systemd/md5sums
```
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

for i in bin lib sbin; do
  ln -sv usr/$i $LFS/$i
done

case $(uname -m) in
  x86_64) mkdir -pv $LFS/lib64 ;;
esac
```
```bash
mkdir -pv $LFS/tools
```
---

## Step 9 — Adding the LFS User

```bash
groupadd <user_name>
useradd -s /bin/bash -g <user_name> -m -k /dev/null <user_name>
```
```bash
passwd <user_name>
```
### Grant <user_name> full access
```bash
chown -v <user_name> $LFS/{usr{,/*},var,etc,tools}
case $(uname -m) in
  x86_64) chown -v <user_name> $LFS/lib64 ;;
esac
```
### Login <user_name>
```bash
su - <user_name>
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
### Change the exiting host file name (Optional)
```bash
su -
```
```bash
[ ! -e /etc/bash.bashrc ] || mv -v /etc/bash.bashrc /etc/bash.bashrc.NOUSE
```
###  Set MAKEFLAGS
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
### Check the powerprofilesctl
```bash
powerprofilesctl
```
### change to performance
```bash
su -
```
```bash
powerprofilesctl set performance
```
```bash
su - <user_name>
```
---

## Step 11 - Compiling a Cross-Toolchain
### Binutils-2.46.0 - Pass 1
```bash
cd $LFS/sources
```
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
../configure --prefix=$LFS/tools \
             --with-sysroot=$LFS \
             --target=$LFS_TGT   \
             --disable-nls       \
             --enable-gprofng=no \
             --disable-werror    \
             --enable-new-dtags  \
             --enable-default-hash-style=gnu
```
```bash
make
```
```bash
make install
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
tar -xf ../mpc-1.3.1.tar.gz
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
../configure                  \
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
    --enable-languages=c,c++
```
```bash
make
```
```bash
make install
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
find usr/include -type f ! -name '*.h' -delete
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
mkdir -pv $LFS/usr/share/man/man8
mv -v $LFS/usr/share/man/man1/chroot.1 $LFS/usr/share/man/man8/chroot.8
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
## Step 13 - Entering Chroot and Building Additional Temporary Tools
### Changing Ownership
```bash
echo $LFS
```
#### If any error
```bash
export LFS=/mnt/lfs
```
```bash
chown --from <user_name> -R root:root $LFS/{usr,var,etc,tools}
```
```bash
case $(uname -m) in
  x86_64) chown --from <user_name> -R root:root $LFS/lib64 ;;
esac
```
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
### Backup
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

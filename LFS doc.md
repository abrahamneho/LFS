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

bash version-check.sh
```

---

## Step 2 — Start fdisk on your disk
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
wget https://www.linuxfromscratch.org/lfs/view/stable-systemd/wget-list-systemd
```
```bash
wget --input-file=wget-list-systemd --continue --directory-prefix=$LFS/sources
```
### Verify packages
```bash
wget https://www.linuxfromscratch.org/lfs/view/stable-systemd/md5sums
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
### Any needed Patches
```bash
wget -P $LFS/sources <Download>
```
```bash
cd $LFS/sources
md5sum -c <<< "MD5 sum  Download"
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
### Change the exiting host file name
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
---

## Step 11 - Compiling a Cross-Toolchain
### Binutils-2.46.0 - Pass 1
```bash
tar -xf binutils-2.46.0.tar.xz 
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
rm -rf binutils-2.46.0
```
---

## Step 12 - GCC-15.2.0 - Pass 1 
```bash
tar -xf gcc-15.2.0.tar.xz
```
```bash
cd gcc-15.2.0
```
### Additional steps
```bash
tar -xf ../mpfr-4.2.2.tar.xz
```
```bash
mv -v mpfr-4.2.2 mpfr
```
```bash
tar -xf ../gmp-6.3.0.tar.xz
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
### Set the default directory name for 64-bit libraries to “lib”
```bash
case $(uname -m) in
  x86_64)
    sed -e '/m64=/s/lib64/lib/' \
        -i.orig gcc/config/i386/t-linux64
 ;;
esac
```
### steps
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
cd ../..
```
```bash
rm -rf gcc-15.2.0
```
## Step - 12 Linux-6.18.10 API Headers
```bash
tar -xf 
```
```bash
cd 
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
cd ../..
```
```bash
rm -rf 
```
```bash

```
```bash

```
## Result

After all steps you will have `LFS OS` ready.

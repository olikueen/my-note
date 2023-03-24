### 源码drbd

### drbd-utils
```

drbd共有两部分组成：内核模块和用户空间的管理工具。
其中drbd内核模块代码已经整合进Linux内核2.6.33以后的版本中，因此内核版本高于核2.6.33版本的话，只需要安装管理工具drbd-utils即可

zypper in gcc gcc-c++ make automake autoconf kernel-devel kernel-headers flex libxslt libxslt-devel asciidoc
zypper in -y gcc gcc-c++ make kernel-devel kernel-headers flex

./autogen.sh
./configure --prefix=/usr/data/drbd-utils-9.1.0 --localstatedir=/var --sysconfdir=/etc

make KDIR=/usr/src/linux-4.4.73-5
make install


报错：
        http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl drbdsetup.xml
        error : Operation timed out
        warning: failed to load external entity "http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl"
        cannot parse http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl
        Makefile:100: recipe for target 'drbdsetup.8' failed
        make[1]: *** [drbdsetup.8] Error 4
        make[1]: Leaving directory '/root/drbd/drbd-utils-9.5.0/documentation/v9'
        Makefile:82: recipe for target 'doc' failed
        make: *** [doc] Error 2

解决方法：
        直接略过 不用理会
```

drbd
```
tar -zxvf drbd-9.0.15-1.tar.gz
cd drbd-9.0.15-1/
make
make install

vim /etc/modprobe.d/10-unsupported-modules.conf
        allow_unsupported_modules 1
        
depmod
modprobe drbd
lsmod | grep drbd
drbd                  577536  0 
libcrc32c              16384  1 drbd
```
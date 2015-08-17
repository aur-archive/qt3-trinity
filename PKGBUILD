# $Id$
# Maintainer: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: John Proctor <jproctor@prium.net>

pkgname=qt3-trinity
pkgver=3.3.8
pkgrel=1
pkgdesc="The QT gui toolkit. Trinity Desktop Environment patched"
arch=(i686 x86_64)
license=('GPL')
url="http://www.trolltech.com/products/qt/index.html"
pkgfqn=qt-x11-free-${pkgver}
install=qt.install
depends=('libpng>=1.4.0' 'libxmu' 'libxcursor' 'libxinerama' 'mesa' \
	 'libxft' 'libxrandr' 'libmng>=1.0.10-3')
makedepends=('mysql' 'postgresql>=8.2.3' 'unixodbc' 'sqlite3')
optdepends=('libmysqlclient' 'postgresql-libs' 'unixodbc')
replaces=('qt3')
conflicts=('qt3')
source=(ftp://ftp.trolltech.com/qt/source/${pkgfqn}.tar.bz2 qt3-png14.patch qt.profile \
        qt-copy-kde-patches.tar.bz2 qt-patches.tar.bz2 utf8-bug-qt3.diff \
	qt-font-default-subst.diff mysql.patch eastern_asian_languagues.diff qt-odbc.patch qt3_3.3.8c.diff)
options=(!libtool)
md5sums=( 'cf3c43a7dfde5bfb76f8001102fe6e85'
          '1dc671df42b9030dbdf68bb61cd3375e'
          'f72d1eb4eb49b9a9467c1f6035194266'
          'f2a2dbdbfee9422c90efc3ef3f86197c'
          '2f00e5c0c1e2c2a23dddc982cd79f3e0'
          'f6b3b39040f2b8f19ba1cf1445468c28'
          '9370d82e85f2c799335ed0dcc1d53189'
          '7d40ed1bd40d33d8b9b27a2076a5d22a'
          '616f1f3029cf8375256ad6a406de3549'
          '2178ca88dfd75a230918593b30eb0dbe'
          'e7575430889707424e8a24f12727fa55')

# qt-copy-kde-patches come from http://websvn.kde.org/trunk/qt-copy/patches/
# other qt-patches come from fedora and gentoo

build() {
  unset QMAKESPEC
  export QTDIR=${srcdir}/$pkgfqn
  export PATH=${QTDIR}/bin:${PATH}
  export LD_LIBRARY_PATH=${QTDIR}/lib:${LD_LIBRARY_PATH}
  export QMAKESPEC=$QTDIR/mkspecs/linux-g++
  cd ${srcdir}/$pkgfqn
  # apply qt patches from kde.org
  for i in ../qt-copy-kde-patches/*; do
    patch -Np0 -i $i || return 1
  done
  # apply other qt patches and one security fix from debian/gentoo
  for i in ../qt-patches/*; do
    patch -Np1 -i $i || return 1
  done
  # fix utf8 bug
  patch -Np0 -i ../utf8-bug-qt3.diff || return 1
  # fix asia fonts
  patch -Np0 -i ../qt-font-default-subst.diff || return 1
  # fix segfaults on exit when using mysql DB driver
  patch -Np0 -i ../mysql.patch || return 1
  # fix CJK font/chars select error (FS#11245)
  patch -p1 -i ${srcdir}/eastern_asian_languagues.diff || return 1
  # fix build problem against new unixODBC
  patch -p1 -i ${srcdir}/qt-odbc.patch || return 1
  # apply patch for Trinity Desktop.
  patch -p2 -i ${srcdir}/qt3_3.3.8c.diff || return 1
  
  patch -p0 -i ${srcdir}/qt3-png14.patch || return 1
  # start compiling qt
  sed -i 's|-cp -P -f|-cp -L -f|' qmake/Makefile.unix
  rm -rf doc/html examples tutorial
  sed -i "s|sub-tutorial sub-examples||" Makefile
  sed -i "s|-O2|$CXXFLAGS|" mkspecs/linux-g++/qmake.conf
  sed -i "s|-O2|$CXXFLAGS|" mkspecs/linux-g++-32/qmake.conf
  sed -i "s|-O2|$CXXFLAGS|" mkspecs/linux-g++-64/qmake.conf
  sed -i "s|-I. |$CXXFLAGS -I. |" qmake/Makefile.unix
  sed -i "s|read acceptance|acceptance=yes|" configure

 # remove unwanted mkspecs
  rm -rf mkspecs/{*aix*,*bsd*,cygwin*,dgux*,darwin*,hpux*,hurd*,irix*,lynxos*,macx*,qnx*,reliant*,sco*,solaris*,tru64*,unixware*,win32*}

  if [ "$CARCH" = "x86_64" ]; then
      export ARCH="-64"	
    else unset ARCH
  fi

  ./configure -prefix /opt/qt -platform linux-g++$ARCH \
    -system-zlib -qt-gif -release -shared -sm -nis -thread -stl \
    -system-lib{png,jpeg,mng} \
    -no-g++-exceptions -plugin-sql-{mysql,psql,sqlite,odbc}

  # fix /opt/qt/lib path
  [ "$CARCH" = "x86_64" ] && sed -i "s|/opt/qt/lib64|/opt/qt/lib|g" ${srcdir}/$pkgfqn/src/Makefile
  [ "$CARCH" = "x86_64" ] && sed -i "s|/opt/qt/lib64|/opt/qt/lib|g" ${srcdir}/$pkgfqn/tools/designer/designer/Makefile
  [ "$CARCH" = "x86_64" ] && sed -i "s|/opt/qt/lib64|/opt/qt/lib|g" ${srcdir}/$pkgfqn/tools/designer/editor/Makefile
  [ "$CARCH" = "x86_64" ] && sed -i "s|/opt/qt/lib64|/opt/qt/lib|g" ${srcdir}/$pkgfqn/tools/assistant/lib/Makefile
  [ "$CARCH" = "x86_64" ] && sed -i "s|/opt/qt/lib64|/opt/qt/lib|g" ${srcdir}/$pkgfqn/tools/designer/uilib/Makefile

  cd ${srcdir}/$pkgfqn
  make -C qmake || return 1
  cd ${srcdir}/$pkgfqn/plugins/src/sqldrivers/mysql
  ${srcdir}/$pkgfqn/bin/qmake -o Makefile "INCPATH+=/usr/include/mysql" "LIBS+=-L/usr/lib/mysql -lmysqlclient" mysql.pro
  cd ${srcdir}/$pkgfqn/plugins/src/sqldrivers/psql
  ${srcdir}/$pkgfqn/bin/qmake -o Makefile "INCPATH+=/usr/src/include /usr/include/postgresql/server" "LIBS+=-L/usr/lib -lpq" psql.pro

  cd ${srcdir}/$pkgfqn
  # fix the broken makefiles
  #sed -i 's|[[:space:]]*strip.*doc/html.*$|#|g' src/Makefile
  make || return 1
  make INSTALL_ROOT=${pkgdir} install
  rm -rf ${pkgdir}/opt/qt/{phrasebooks,templates,translations}
  sed -i "s|-L${srcdir}/$pkgfqn/lib ||g" ${pkgdir}/opt/qt/lib/*.prl
  install -D -m755 qmake/qmake ${pkgdir}/opt/qt/bin/qmake
  install -D -m755 ${srcdir}/qt.profile ${pkgdir}/etc/profile.d/qt3.sh
  ln -sf /opt/qt/bin/qtconfig ${pkgdir}/opt/qt/bin/qt3config 
  rm -f ${pkgdir}/opt/qt/mkspecs/linux-g++$ARCH/linux-g++$ARCH

  # install man pages
  mkdir -p ${pkgdir}/opt/qt/man
  cp -r ${srcdir}/$pkgfqn/doc/man/{man1,man3} ${pkgdir}/opt/qt/man/

  install -d -m755 ${pkgdir}/etc/ld.so.conf.d/
  echo '/opt/qt/lib' > ${pkgdir}/etc/ld.so.conf.d/qt3.conf
}

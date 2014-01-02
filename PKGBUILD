# Maintainer: Chris Prather <chris.prather@tamarou.com>
# Contributor: Rustam Tsurik <rustam.tsurik@gmail.com>
# Contributor: Andrea Scarpino <andrea@archlinux.org>
# Contributor: Douglas Soares de Andrade <douglas@archlinux.org>

pkgname=mysql-cluster
true && pkgname=('libmysqlclient' 'mysql-clients' 'mysql-cluster')
pkgbase=mysql-cluster-gpl
pkgver=7.3.3
pkgrel=1
arch=('i686' 'x86_64')
license=('GPL')
url="https://www.mysql.com/products/community/"
makedepends=('cmake' 'openssl' 'zlib')
options=('!libtool')
source=(http://dev.mysql.com/get/Downloads/MySQL-Cluster-7.3/${pkgbase}-${pkgver}.tar.gz)
md5sums=('2787777e14730729b1c955833f19f7ba')

build() {
  cd "${srcdir}/${pkgbase}-${pkgver}"
  patch -p1 < ../../bison.patch
#  patch -p0 < ../../mdev4902.patch
  patch -p0 < ../../mysql-srv_buf_size.patch
  cd .. 

  mkdir -p build
  rm -rf build/*
  cd build

  cmake ../${pkgbase}-${pkgver} \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DSYSCONFDIR=/etc/${pkgbase} \
    -DMYSQL_DATADIR=/var/lib/${pkgbase} \
    -DMYSQL_UNIX_ADDR=/run/mysqld/mysqld.sock \
    -DDEFAULT_CHARSET=utf8 \
    -DDEFAULT_COLLATION=utf8_general_ci \
    -DENABLED_LOCAL_INFILE=ON \
    -DINSTALL_INFODIR=share/${pkgbase}/docs \
    -DINSTALL_MANDIR=share/man \
    -DINSTALL_PLUGINDIR=lib/${pkgbase}/plugin \
    -DINSTALL_SCRIPTDIR=bin \
    -DINSTALL_INCLUDEDIR=include/${pkgbase} \
    -DINSTALL_DOCREADMEDIR=share/${pkgbase} \
    -DINSTALL_SUPPORTFILESDIR=share/${pkgbase} \
    -DINSTALL_MYSQLSHAREDIR=share/${pkgbase} \
    -DINSTALL_DOCDIR=share/${pkgbase}/docs \
    -DINSTALL_SHAREDIR=share/${pkgbase} \
    -DWITH_READLINE=ON \
    -DWITH_ZLIB=system \
    -DWITH_SSL=system \
    -DWITH_LIBWRAP=OFF \
    -DWITH_MYSQLD_LDFLAGS="${LDFLAGS}" \
    -DWITH_EXTRA_CHARSETS=complex \
    -DWITH_EMBEDDED_SERVER=ON \
    -DWITHOUT_EXAMPLE_STORAGE_ENGINE=ON \
    -DWITH_NDB_JAVA=OFF \
    -DCMAKE_C_FLAGS="-fPIC ${CFLAGS} -fno-strict-aliasing -DBIG_JOINS=1 -fomit-frame-pointer" \
    -DCMAKE_CXX_FLAGS="-fPIC ${CXXFLAGS} -fno-strict-aliasing -DBIG_JOINS=1 -felide-constructors -fno-rtti"

  make
}

package_libmysqlclient(){
  pkgdesc="MySQL client libraries"
  depends=('openssl')
  conflicts=('libmariadbclient')
  provides=("libmariadbclient=$pkgver")

  cd build
  for dir in include libmysql libmysqld libservices; do
    make -C ${dir} DESTDIR="${pkgdir}" install
  done

  install -d "${pkgdir}"/usr/bin
  install -m755 scripts/mysql_config "${pkgdir}"/usr/bin/
  install -d "${pkgdir}"/usr/share/man/man1
  for man in mysql_config mysql_client_test_embedded mysqltest_embedded; do
    install -m644 "${srcdir}"/${pkgbase}-${pkgver}/man/$man.1 "${pkgdir}"/usr/share/man/man1/$man.1
  done
}

package_mysql-clients(){
  pkgdesc="MySQL client tools"
  depends=('libmysqlclient')
  conflicts=('mariadb-clients')
  provides=("mariadb-clients=$pkgver")

  cd build
  make -C client DESTDIR="${pkgdir}" install

  # install man pages
  install -d "${pkgdir}"/usr/share/man/man1
  for man in mysql mysqladmin mysqlcheck mysqldump mysqlimport mysqlshow mysqlslap; do
    install -m644 "${srcdir}"/${pkgbase}-${pkgver}/man/$man.1 "${pkgdir}"/usr/share/man/man1/$man.1
  done

  # provided by mysql
  rm "${pkgdir}"/usr/bin/{mysql_{plugin,upgrade,config_editor},mysqlbinlog,mysqltest}
}

package_mysql-cluster(){
  pkgdesc="A fast SQL database server"
  backup=('etc/mysql/my.cnf')
  install=mysql.install
  depends=('mysql-clients' 'systemd-tools')
  conflicts=('mariadb')
  provides=("mariadb=$pkgver")
  options=('emptydirs')

  cd build
  make DESTDIR="${pkgdir}" install

  # provided by libmysqlclient
  rm "${pkgdir}"/usr/bin/{mysql_config,mysql_client_test_embedded,mysqltest_embedded}
  rm "${pkgdir}"/usr/lib/libmysql*
  rm -r "${pkgdir}"/usr/include/
  rm "${pkgdir}"/usr/share/man/man1/{mysql_config,mysql_client_test_embedded,mysqltest_embedded}.1
  
  # provided by mysql-clients
  rm "${pkgdir}"/usr/bin/{mysql,mysqladmin,mysqlcheck,mysqldump,mysqlimport,mysqlshow,mysqlslap}
  rm "${pkgdir}"/usr/share/man/man1/{mysql,mysqladmin,mysqlcheck,mysqldump,mysqlimport,mysqlshow,mysqlslap}.1

  # not needed
  rm -r "${pkgdir}"/usr/{data,mysql-test,sql-bench}
  rm "${pkgdir}"/usr/share/man/man1/mysql-test-run.pl.1

  #install -dm700 "${pkgdir}"/var/lib/mysql
}

pkgdesc="A fast SQL database server"

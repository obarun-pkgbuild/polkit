# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://projects.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/polkit
# 						Maintainer: Jan de Groot <jgc@archlinux.org>

pkgname=polkit
pkgver=0.113+29+g3272a98
pkgrel=3
pkgdesc="Application development toolkit for controlling system-wide privileges"
arch=(x86_64)
license=(LGPL)
url="https://www.freedesktop.org/wiki/Software/polkit"
depends=(glib2 pam expat js)
makedepends=(intltool gtk-doc gobject-introspection git autoconf-archive)
provides=('polkit-consolekit')
replaces=('polkit-consolekit')
conflicts=('polkit-consolekit')
install=polkit.install
_commit=3272a988655c3236b55bad70e9a3af20857f384b  # master
source=("git+https://anongit.freedesktop.org/git/polkit#commit=$_commit"
		"dbus-rules.patch"
		"polkit.pam")
md5sums=('SKIP'
         'e8e65e9291d83c4b91fa8de424a938ef'
         '6564f95878297b954f0572bc1610dd15')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

pkgver() {
  cd $pkgname
  git describe --tags | sed 's/-/+/g'
}

prepare(){
	
	cd ${pkgname}

	#patch remove systemd service for dbus rules
	patch -Np1 -i ${srcdir}/dbus-rules.patch

	NOCONFIGURE=1 ./autogen.sh

		# Many thank to void dev to find this fucking systemd behaviour
	# Drop requirement of /sys/fs/cgroup/systemd test in configure... WTF.
	sed -e 's,/sys/fs/cgroup/systemd/,/sys/fs/cgroup,g' -i configure
}

build() {
  cd ${pkgname}

  ./configure 	--prefix=/usr \
				--sysconfdir=/etc \
				--localstatedir=/var \
				--libexecdir=/usr/lib/polkit-1 \
				--enable-libsystemd-login=no \
				--with-systemdsystemunitdir=no \
				--disable-static \
				--enable-gtk-doc \
				--with-os-type=redhat \
				--with-polkitd-user=polkitd \
				--with-authfw=pam \
				--with-mozjs=mozjs185	
  
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool
  	
  make
}

check(){
	cd ${pkgname}
	make -k check || :
}

package() {
  cd ${pkgname}
  make DESTDIR="$pkgdir" install

  chown root:102 "$pkgdir"/{etc,usr/share}/polkit-1/rules.d
  chmod 750      "$pkgdir"/{etc,usr/share}/polkit-1/rules.d
  
  install -m644 "$srcdir/polkit.pam" "$pkgdir/etc/pam.d/polkit-1"

}

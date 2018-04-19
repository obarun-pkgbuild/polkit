# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://projects.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/polkit
# 						Maintainer: Jan de Groot <jgc@archlinux.org>

pkgname=polkit
pkgver=0.114
pkgrel=2
pkgdesc="Application development toolkit for controlling system-wide privileges"
arch=(x86_64)
license=(LGPL)
url="https://www.freedesktop.org/wiki/Software/polkit"
depends=(glib2 pam expat js52)
makedepends=(intltool gtk-doc gobject-introspection git autoconf-archive)
provides=('polkit-consolekit')
replaces=('polkit-consolekit')
conflicts=('polkit-consolekit')
_commit=ed06baed179166389d536420a6fc532781d48178 # tags/0.114^0
source=("git+https://anongit.freedesktop.org/git/polkit#commit=$_commit"
		"dbus-rules.patch"
		"polkit.pam"
		"polkit.sysusers")
md5sums=('SKIP'
         'e8e65e9291d83c4b91fa8de424a938ef'
         '6564f95878297b954f0572bc1610dd15'
         '57b3a16112267fa099c06de443dafd56')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

pkgver() {
  cd $pkgname
  git describe --tags | sed 's/-/+/g'
}

prepare(){
	
	cd ${pkgname}
	git cherry-pick -n 373705b35e7f6c7dc83de5e0a3ce11ecd15d0409
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
				--libexecdir=/usr/lib/ \
				--enable-libsystemd-login=no \
				--with-systemdsystemunitdir=no \
				--enable-libelogind=no \
				--enable-static=no \
				--enable-gtk-doc=yes \
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
  
  install -Dm644 "$srcdir/polkit.sysusers" "$pkgdir/usr/lib/sysusers.d/polkit.conf"
  
  install -d -o root -g 102 -m 750 "$pkgdir"/{etc,usr/share}/polkit-1/rules.d
  
  install -m644 "$srcdir/polkit.pam" "$pkgdir/etc/pam.d/polkit-1"

}

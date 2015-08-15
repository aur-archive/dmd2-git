# Maintainer: Zenja Ivkovic <izenja@gmail.com> aka izenja
pkgname=dmd2-git
true && pkgname=('dmd2-git' 'druntime-git' 'libphobos2-git')
pkgver=20120128
pkgrel=1
arch=('i686' 'x86_64')
url="http://www.digitalmars.com/d/2.0/"
license=('GPL')
makedepends=('git')
options=('!strip' 'docs')

_gitroot=("git://github.com/D-Programming-Language/dmd.git" "git://github.com/D-Programming-Language/druntime.git" "git://github.com/D-Programming-Language/phobos.git")
_gitname=('dmd' 'druntime' 'phobos')

if [ $CARCH = 'x86_64' ] ; then
	archstr="64"
fi
if [ $CARCH = 'i686' ] ; then
	archstr="32"
fi

build() {
	cd "$srcdir"
	msg "Retrieving from repository: dmd"
	if [ -d ${_gitname[0]} ] ; then
		cd ${_gitname[0]} && git pull origin
	else
		git clone ${_gitroot[0]} ${_gitname[0]}
	fi
	
	cd "$srcdir"
	msg "Retrieving from repository: druntime"
	if [ -d ${_gitname[1]} ] ; then
		cd ${_gitname[1]} && git pull origin
	else
		git clone ${_gitroot[1]} ${_gitname[1]}
	fi

	cd "$srcdir"
	msg "Retrieving from repository: phobos"
	if [ -d ${_gitname[2]} ] ; then
		cd ${_gitname[2]} && git pull origin
	else
		git clone ${_gitroot[2]} ${_gitname[2]}
	fi
	
	msg "Making dmd"
	rm -rf "$srcdir/${_gitname[0]}-build"
	git clone "$srcdir/${_gitname[0]}" "$srcdir/${_gitname[0]}-build"
	cd "$srcdir/${_gitname[0]}-build/src"
	make -f posix.mak MODEL=$archstr
		
	msg "Making druntime"
	rm -rf "$srcdir/${_gitname[1]}-build"
	git clone "$srcdir/${_gitname[1]}" "$srcdir/${_gitname[1]}-build"
	cd "$srcdir/${_gitname[1]}-build"
	# stacktrace.d fails to compile on 64 bit, at least for me, and probably pointless to have on linux anyway
	patch posix.mak <<-'ENDPATCH'
		145d144
		< 	src/core/sys/windows/stacktrace.d \
		480d478
		< 	$(IMPDIR)/core/sys/windows/stacktrace.di \
	ENDPATCH
	PATH="$srcdir/${_gitname[0]}-build/src:$PATH" make -f posix.mak MODEL=$archstr
	
	msg "Making phobos"
	rm -rf "$srcdir/${_gitname[2]}-build"
	git clone "$srcdir/${_gitname[2]}" "$srcdir/${_gitname[2]}-build"
	cd "$srcdir/${_gitname[2]}-build"
	# phobos' makefile tries to make ../druntime - make that the build directory we use
	patch posix.mak <<-ENDPATCH
		48c48
		< DRUNTIME_PATH = ../druntime
		---
		> DRUNTIME_PATH = ../${_gitname[1]}-build
	ENDPATCH
	PATH="$srcdir/${_gitname[0]}-build/src:$PATH" make -f posix.mak MODEL=$archstr
}

package_dmd2-git() {
	provides=("dmd2" "d-compiler")
	conflicts=("dmd2")
	cd "$srcdir/${_gitname[0]}-build/src"
	msg "Packaging dmd"
	
	install -Dm755 ./dmd $pkgdir/usr/bin/dmd
	echo -e "[Environment]\nDFLAGS=-m$archstr -I/usr/include/d -I/usr/include/d/druntime/import -L-L/usr/lib -L-lrt" > $srcdir/dmd.conf
	install -Dm644 $srcdir/dmd.conf $pkgdir/etc/dmd.conf
	
	msg "Packaging dmd man files"
	cd "$srcdir/${_gitname[0]}-build/docs/man/man1"
	for x in ./*.1 ; do
		install -Dm644 $x "$pkgdir/usr/share/man/man1/$(basename $x)"
	done
	for x in ./*.5 ; do
		install -Dm644 $x "$pkgdir/usr/share/man/man5/$(basename $x)"
	done
}

package_druntime-git() {
	provides=("druntime")
	conflicts=("druntime")
	msg "Packaging druntime"
	
	install -Dm644 $srcdir/${_gitname[1]}-build/lib/libdruntime-*.a $pkgdir/usr/lib/libdruntime.a
	mkdir -p $pkgdir/usr/include/d/druntime
	cp -Rf $srcdir/${_gitname[1]}-build/import $pkgdir/usr/include/d/druntime
}

package_libphobos2-git() {
	provides=("libphobos2")
	conflicts=("libphobos2" "libtango")
	msg "Packaging phobos"
	
	cd "$srcdir/${_gitname[2]}-build"
	install -Dm644 ./generated/linux/release/$archstr/libphobos2.a $pkgdir/usr/lib/libphobos2.a
	mkdir -p $pkgdir/usr/include/d
	cp -Rf std $pkgdir/usr/include/d
	cp -Rf etc $pkgdir/usr/include/d
	cp -f {crc32,index,unittest}.d $pkgdir/usr/include/d
}

pkgdesc="Digital Mars D compiler, runtime and standard library (git master branch)"

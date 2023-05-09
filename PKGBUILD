# Maintainer: An Van Quoc <andtf2002 at gmail dot com>

pkgname=ns3
pkgver=3.38
pkgrel=1
pkgdesc="A discrete-event network simulator for building internet systems"
arch=("any")
url="https://www.nsnam.org/"
license=("GPL2")

depends=(
  "gcc>=8" "cmake" "cmake-format" "boost" "boost-libs"  # build system
  "sphinx" "texlive-core" # documentation
  "openmpi" "dpdk" "eigen" "qt5-base" "gsl" "python-pygraphviz" "python-gobject" "python-sphinx" # modules
)
makedepends=(
  "mercurial" "git" "bzip2" "tar" "doxygen"
)
optdepends=(
  "ninja: alternative for make"
  "cppyy: python binding and visualizer"
  "netmap: native packet transmission"
  "doxygen: for viewing documentation"
  "dia: for viewing documentation"
)
conflicts=(
  "ns3-git" "ns3-hg"
)

install=$pkgname.install

source=(
  "https://www.nsnam.org/releases/ns-allinone-${pkgver}.tar.bz2"
  "brite::hg+https://code.nsnam.org/BRITE"
  "hg+https://code.nsnam.org/openflow"
  "git+https://github.com/kohler/click.git"
  
  "dpdk-net-device.patch"
  "ns3.sh"
  "NetAnim.desktop"
)
sha256sums=('af37156e9f069829a1ba81ff28cb57e4c95d067c9f53a37d554652c59242b5b5'
            'SKIP'
            'SKIP'
            'SKIP'
            '7d495d0bc476b8bae1f32bf6d2031d5a0b02d085a43d3a3fa9f7feb710c48470'
            '7d8e6b41c49a9ed55596e1f73f5fe8478b684e1e0eeb61e51ed183edab5aca53'
            '2379a8e99e0c49d39ff78e75a65c5075b4d22681249aac5756f985cb536b67cd')

shopt -s extglob

prepare() {
  # Setup file structure for consistent directory
  tar -xf "${srcdir}/ns-allinone-${pkgver}.tar.bz2"
  mkdir -p "${srcdir}/${pkgname}/"
  mkdir -p "${srcdir}/netanim/"
  cp -r "${srcdir}/ns-allinone-${pkgver}/ns-${pkgver}/"* "${srcdir}/${pkgname}/"
  cp -r "${srcdir}/ns-allinone-${pkgver}/netanim"-*/* "${srcdir}/netanim/"

  # dpdk-net-device.cc uses obsolete function and constant calls
  patch -Np1 \
    -d "${srcdir}/${pkgname}/src/fd-net-device/model" \
    -i "${srcdir}/dpdk-net-device.patch"
}

build() {
  # Brite
  cd "${srcdir}/brite/"
  make

  # Openflow
  cd "${srcdir}/openflow/"
  ./waf configure --prefix="/usr"
  ./waf build

  # Click
  cd "${srcdir}/click/"
  ./configure \
    --enable-userlevel \
    --disable-linuxmodule \
    --enable-nsclick \
    --enable-wifi \
    --prefix=/usr
  make

  # NetAnim
  cd "${srcdir}/netanim/"
  qmake NetAnim.pro
  make

  # Main source
  cd "${srcdir}/${pkgname}/"
  ./ns3 configure \
    --build-profile=default \
    --prefix=/usr \
    --disable-werror \
    --enable-logs \
    --enable-dpdk \
    --enable-eigen \
    --enable-gsl \
    --enable-mpi \
    --enable-des-metrics \
    --enable-examples \
    --enable-tests \
    --enable-python-bindings \
    --enable-build-version \
    --with-brite "${srcdir}/brite/" \
    --with-click "${srcdir}/click/" \
    --with-openflow "${srcdir}/openflow/"
  ./ns3 build
}

package() {
  install -d -m755 "${pkgdir}/usr/bin/"
  install -d -m755 "${pkgdir}/usr/include/${pkgname}/"
  install -d -m775 "${pkgdir}/opt/${pkgname}/"

  # Source
  cp -r "${srcdir}/${pkgname}/" "${pkgdir}/opt/"
  cp -r "${srcdir}/brite/" "${pkgdir}/usr/include/${pkgname}/"
  cp -r "${srcdir}/openflow/" "${pkgdir}/usr/include/${pkgname}/"
  cp -r "${srcdir}/netanim/" "${pkgdir}/usr/include/${pkgname}/"

  # Shortcuts
  install -D -m755 "${srcdir}/${pkgname}.sh" "${pkgdir}/usr/share/${pkgname}/bin/${pkgname}.sh"
  ln -s "/usr/share/${pkgname}/bin/${pkgname}.sh" "${pkgdir}/usr/bin/${pkgname}"
  ln -s "/usr/include/${pkgname}/netanim/NetAnim" "${pkgdir}/usr/bin/netanim"
  install -D -m755 "${srcdir}/NetAnim.desktop" "${pkgdir}/usr/share/applications/NetAnim.desktop"

  # Icons
  install -D -m644 "${srcdir}/netanim/netanim-logo.png" "${pkgdir}/usr/share/pixmaps/netanim-logo.png"

  # Replace each instance of srcdir to correct directory for cmake to work
  find "${pkgdir}/opt/${pkgname}/" -type f -print0 \
    | xargs -0 grep -Il "" \
    | xargs sed -i \
      -e "s,${srcdir}\/brite,\/usr\/include\/${pkgname}\/brite,g" \
      -e "s,${srcdir}\/openflow,\/usr\/include\/${pkgname}\/openflow,g" \
      -e "s,${srcdir}\/click,\/usr\/include\/${pkgname}\/click,g" \
      -e "s,${srcdir},\/opt,g"
}

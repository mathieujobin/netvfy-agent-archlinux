# Maintainer: somekool <somekool _ at _ gmail _ dot _com>

pkgname=netvfy-agent-git
srcgiturl=https://github.com/netvfy/netvfy-agent
pkgver=v1.0.0
pkgrel=1
pkgdesc="Netvfy - Connect to your secure virtual network."
arch=('i686' 'x86_64')
url="https://my.netvfy.org/"
license=('Unknown Free')
depends=('jansson' 'curl' 'libevent' 'openssl')
optdepends=()
makedepends=('git' 'scons' 'cmake')
# - libevent-dev
# - libssl-dev
# - libjansson-dev
# - libcurl4-openssl-dev)
source=("${pkgname}::git+${srcgiturl}.git")
md5sums=('SKIP')

pkgver() {
  gitinfo=`git ls-remote ${srcgiturl} | sort -Vk2`
  sha=`echo "$gitinfo" | head -n 1 | cut -c -7`
  tag=`echo "$gitinfo" | tail -n 1 | awk -F "/" {'print $3'}`
  commits=`curl -s ${srcgiturl}/releases/tag/${tag} | grep -o "[0-9]* commits" | awk {'print $1'}`
  if [ "$commit" = "" ]; then
    printf $tag
  else
    printf "%s.r%s.g%s" $tag $commits $sha
  fi

}

prepare() {
  cd ${srcdir}/${pkgname}
  set -e
  git submodule update --init --recursive

  pushd tapcfg
  ./buildall.sh linuxonly || true > /dev/null
  popd
  set +e
  echo ======================== prepare completed ========================
}

build() {
  [ -d ${srcdir}/${pkgname}/build ] && rm -fr ${srcdir}/${pkgname}/build
  mkdir ${srcdir}/${pkgname}/build
  cd ${srcdir}/${pkgname}/build
  cmake .. -DCMAKE_INSTALL_PREFIX:PATH=${pkgdir}/usr -DWITH_GUI=OFF
  make nvagent
  echo ======================== build completed ========================
}

package() {
  cd ${srcdir}/${pkgname}/build
  make install

  #TODO: this could/should become a post_install item
  echo "#!/bin/sh
  sudo chmod 666 /dev/net/tun
  sudo setcap cap_net_bind_service,cap_net_admin=ep /usr/bin/netvfy-agent
  " > ${pkgdir}/usr/bin/netvirt-allow_user

  mkdir -p ${pkgdir}/usr/lib/systemd/system

  # ArchLinux uses SystemD, Yeah
  cat << EOF > ${pkgdir}/usr/lib/systemd/system/netvfy-agent.service
[Unit]
Description=Netvfy Agent
After=network.target

[Service]
Environment="HOME=/root"
ExecStart=/usr/bin/netvfy-agent
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=always

[Install]
WantedBy=multi-user.target

EOF

}


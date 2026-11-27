# Maintainer: Shadichy <shadichy@blisslabs.org>

pkgname=cros-libsegmentation
pkgver=0.0.1
pkgrel=1
pkgdesc='ChromiumOS system properties library (libsegmentation)'
arch=('x86_64')
url='https://chromium.googlesource.com/chromiumos/platform2/+/HEAD/libsegmentation'
license=('BSD-3-Clause')
depends=('cros-libchrome' 'cros-libbrillo' 'cros-libcrossystem' 'cros-rootdev' 'gcc-libs' 'glibc' 'protobuf')
makedepends=('clang' 'cros-common-mk' 'git' 'gn' 'lld' 'llvm' 'compiler-rt' 'ninja' 'pkgconf' 'python')
options=('!lto')

_local_src_dir=""

_resolve_local_src() {
  _local_src_dir=""
  local base=""
  [[ -n "${CROS_LOCAL_SRC:-}" ]] || return 0
  base="$(cd "${startdir:-$PWD}" && cd "${CROS_LOCAL_SRC}" 2>/dev/null && pwd)" || return 0
  [[ -d "${base}/tree/src/platform2/libsegmentation" ]] && _local_src_dir="${base}/tree/src/platform2"
  [[ -z "${_local_src_dir}" && -d "${base}/src/platform2/libsegmentation" ]] && _local_src_dir="${base}/src/platform2"
  [[ -z "${_local_src_dir}" && -d "${base}/platform2/libsegmentation" ]] && _local_src_dir="${base}/platform2"
  return 0
}

_resolve_local_src
if [[ -n "${_local_src_dir}" ]]; then
  source=('0001-arch-portability.patch')
  sha256sums=('SKIP')
else
  source=(
    'platform2::git+https://chromium.googlesource.com/chromiumos/platform2.git#branch=main'
    '0001-arch-portability.patch'
  )
  sha256sums=('SKIP' 'SKIP')
fi

prepare() {
  local src_dir="${srcdir}/platform2"
  _resolve_local_src
  [[ -z "${_local_src_dir}" ]] || src_dir="${_local_src_dir}"
  rm -rf "${srcdir}/work"
  mkdir -p "${srcdir}/work"
  install -Dm644 "${src_dir}/LICENSE" "${srcdir}/work/LICENSE"
  tar -C "${src_dir}/libsegmentation" --exclude='./.git' --exclude='__pycache__' --exclude='*.pyc' --exclude='*.pyo' -cf - . | tar -C "${srcdir}/work" -xf -
  cd "${srcdir}/work"
  patch -Np1 -i "${srcdir}/0001-arch-portability.patch"
  # Provide feature-management protos from the platform source tree.
  local proto_src=""
  if [[ -n "${_local_src_dir}" ]]; then
    local base_dir
    base_dir="$(dirname "${_local_src_dir}")"
    for candidate in \
      "${base_dir}/platform/feature-management/proto" \
      "${_local_src_dir}/../platform/feature-management/proto"; do
      if [[ -f "${candidate}/feature_management.proto" ]]; then
        proto_src="${candidate}"
        break
      fi
    done
  fi
  if [[ -z "${proto_src}" && -n "${CROS_LOCAL_SRC:-}" ]]; then
    local cros_base
    cros_base="$(cd "${startdir:-$PWD}" && cd "${CROS_LOCAL_SRC}" 2>/dev/null && pwd)" || cros_base=""
    if [[ -n "${cros_base}" ]]; then
      for candidate in \
        "${cros_base}/tree/src/platform/feature-management/proto" \
        "${cros_base}/src/platform/feature-management/proto" \
        "${cros_base}/platform/feature-management/proto"; do
        if [[ -f "${candidate}/feature_management.proto" ]]; then
          proto_src="${candidate}"
          break
        fi
      done
    fi
  fi
  if [[ -z "${proto_src}" ]]; then
    echo "error: feature-management protos not found" >&2
    return 1
  fi
  install -d "${srcdir}/work/feature_management_proto"
  install -m644 \
    "${proto_src}/feature_management.proto" \
    "${proto_src}/device_selection.proto" \
    "${proto_src}/hwid_feature_requirement.proto" \
    "${srcdir}/work/feature_management_proto/"
}

build() {
  cd "${srcdir}/work"
  export CC=clang
  export CXX=clang++
  export AR=llvm-ar
  rm -rf "${srcdir}/out"
  platform2-arch-build \
    --pkg libsegmentation \
    --src "${srcdir}/work" \
    --out "${srcdir}/out" \
    --libdir /usr/lib \
    --use-flags "" \
    --action deviterate \
    -- libsegmentation libsegmentation_pc
}

package() {
  install -Dm755 "${srcdir}/out/out/Default/lib/libsegmentation.so" \
    "${pkgdir}/usr/lib/libsegmentation.so"
  install -Dm644 "${srcdir}/out/out/Default/obj/libsegmentation/libsegmentation.pc" \
    "${pkgdir}/usr/lib/pkgconfig/libsegmentation.pc"
  install -Dm644 "${srcdir}/work/feature_management.h" \
    "${pkgdir}/usr/include/libsegmentation/feature_management.h"
  install -Dm644 "${srcdir}/work/feature_management_interface.h" \
    "${pkgdir}/usr/include/libsegmentation/feature_management_interface.h"
  install -Dm644 "${srcdir}/work/LICENSE" \
    "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}

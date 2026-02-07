# Building .deb packages

Instructions for building bees `.deb` packages locally on a Debian/Ubuntu system.

## Prerequisites

```sh
apt-get install build-essential btrfs-progs libbtrfs-dev markdown file git pkg-config
```

Additional packages for static builds:

```sh
apt-get install libblkid-dev uuid-dev zlib1g-dev liblzo2-dev libzstd-dev
```

## Build the binary

Clone and build from a tagged release:

```sh
git clone --depth 1 --branch v0.11 https://github.com/Zygo/bees.git
cd bees
make -j$(nproc)
strip bin/bees
```

For a static binary:

```sh
make -j$(nproc) LDFLAGS="-static"
strip --strip-all bin/bees
```

## Create the package tree

Install into a staging directory:

```sh
make install DESTDIR=/tmp/pkg PREFIX=/usr
```

This installs:

- `/usr/lib/bees/bees` — the binary
- `/usr/sbin/beesd` — the daemon wrapper script
- `/etc/bees/beesd.conf.sample` — sample configuration
- Systemd unit file (if systemd is detected via pkg-config)

## Write the control file

```sh
mkdir -p /tmp/pkg/DEBIAN
```

For a **dynamic** build, derive the `Depends` line from the linked libraries:

```sh
DEP_LIST=$(ldd /tmp/pkg/usr/lib/bees/bees \
  | grep -oE '/[^ ]+' \
  | sort -u \
  | while read -r lib; do
      pkg=$(dpkg -S "$lib" 2>/dev/null | head -1 | sed 's/:.*//')
      if [ -n "$pkg" ]; then
        ver=$(dpkg-query -W -f '${Version}' "$pkg" 2>/dev/null)
        echo "${pkg} (>= ${ver})"
      fi
    done \
  | sort -u \
  | paste -sd ', ')
```

Then write the control file:

```sh
cat > /tmp/pkg/DEBIAN/control << EOF
Package: bees
Version: 0.11
Architecture: $(dpkg --print-architecture)
Maintainer: bees build <noreply@github.com>
Description: Best-Effort Extent-Same, a btrfs dedup agent
Depends: ${DEP_LIST}
Conflicts: bees-static
Replaces: bees-static
EOF
```

For a **static** build, use `bees-static` as the package name and omit `Depends`:

```sh
cat > /tmp/pkg/DEBIAN/control << EOF
Package: bees-static
Version: 0.11
Architecture: $(dpkg --print-architecture)
Maintainer: bees build <noreply@github.com>
Description: Best-Effort Extent-Same, a btrfs dedup agent
Conflicts: bees
Replaces: bees
EOF
```

## Build the .deb

```sh
dpkg-deb --build /tmp/pkg bees_0.11_$(dpkg --print-architecture).deb
```

## Verify

```sh
dpkg -I bees_0.11_amd64.deb   # show package metadata
dpkg -c bees_0.11_amd64.deb   # list contained files
```

## Install

```sh
sudo dpkg -i bees_0.11_amd64.deb
```

The `bees` and `bees-static` packages conflict with each other, so only one can be installed at a time. Installing one will replace the other.

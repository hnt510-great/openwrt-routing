# OpenWrt routing feed

[简体中文](README.md) | **English**

This private source feed is used by the customized
[hnt510-great/openwrt](https://github.com/hnt510-great/openwrt) main tree.
The project targets x86_64/glibc, APK, native systemd and GNOME. Package
selection follows the main tree's saved configuration.

| Item | Value |
| --- | --- |
| Feed name | `routing` |
| Repository branch | `master` |
| Original upstream | [https://github.com/openwrt/routing](https://github.com/openwrt/routing) (`master`) |

Based on OpenWrt routing, this feed contains optional routing, neighbor
discovery and mesh recipes. Current directories include `batmand`, `cjdns`,
`luci-app-cjdns`, `ndppd`, `pimbd` and `ahcpd`. The main tree's `.config`
determines which packages enter an image.

## Project role

This independent private Git feed participates in full source reconstruction.
The main docs define no additional feed-specific network topology. A recipe's
presence does not establish service enablement or physical-network acceptance.

Routing datapath changes, full cone NAT, the dnsmasq nftset fast path and daed
eBPF interception belong to the main tree's kernel/network recipes and
`package/custom`. Native service management follows the preserved UCI/procd
interfaces.

Complete-image DHCP, DNS and routing/NAT smoke checks do not replace protocol
acceptance tests for optional software such as cjdns or batmand.

## Get the source and build

Authenticate on the build machine with an account that can read every private
repository, then obtain the complete main tree:

```sh
gh auth login --hostname github.com --git-protocol https --web
gh auth setup-git
git clone https://github.com/hnt510-great/openwrt.git
cd openwrt
./scripts/feeds update -a
./scripts/feeds install -a
```

The main tree's `feeds.conf.default` already contains:

```text
src-git routing https://github.com/hnt510-great/openwrt-routing.git;master
```

To update only this feed in an existing main tree:

```sh
./scripts/feeds update routing
./scripts/feeds install -a -p routing
```

Select the desired package in the main configuration and run `make defconfig`.
Example individual package build:

```sh
make package/feeds/routing/cjdns/compile V=s
```

Run these commands from the main-tree root. Preserve an existing `.config`;
see [local build](https://github.com/hnt510-great/openwrt/blob/main/docs/local-build.md) for setup in a new checkout.
`install` creates build-recipe links rather than installing software on a
device. The full `update -a` also checks and can modify main-tree custom recipes.

## Maintenance and validation

`origin` points to this private repository. To merge original upstream changes
in a new clone, add `upstream` once:

```sh
git -C feeds/routing remote add upstream https://github.com/openwrt/routing.git
git -C feeds/routing fetch upstream
git -C feeds/routing merge upstream/master
# Review, resolve conflicts and validate before pushing
git -C feeds/routing push origin master
```

Skip the add command if `upstream` already exists. `scripts/feeds update` pulls
the private branch; it does not merge original upstream automatically. Feed
integration and main-tree commits are separate. Preserve existing commits,
stashes and uncommitted work. Main-tree `patches/feeds/` files and feature
manifests are historical backups; do not reapply changes already present.

Changes to published APK contents require a new package version, preserving
immutable package identities. Record source checks, package builds, VM tests
and physical-device validation separately. This page summarizes existing
design and evidence without claiming new build or device results. See
[private GitHub hosting](https://github.com/hnt510-great/openwrt/blob/main/docs/github-source-hosting.md).

## Design and validation documents

- [Source and saved configuration reconstruction](https://github.com/hnt510-great/openwrt/blob/main/docs/local-build.md)
- [Full cone NAT implementation](https://github.com/hnt510-great/openwrt/blob/main/docs/fullcone-nat.md)
- [dnsmasq nftset fast path](https://github.com/hnt510-great/openwrt/blob/main/docs/dnsmasq-nftset.md)
- [daed eBPF proxy](https://github.com/hnt510-great/openwrt/blob/main/docs/daed.md)
- [Complete-image network validation scope](https://github.com/hnt510-great/openwrt/blob/main/docs/full-image-validation-20261031.md)

## License and upstream

Retain upstream copyrights and licenses. Each package's source follows its
recipe's `PKG_LICENSE` and source declarations.

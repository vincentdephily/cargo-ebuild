# cargo-ebuild

[![Build Status](https://github.com/gentoo/cargo-ebuild/actions/workflows/rust.yml/badge.svg)](https://github.com/gentoo/cargo-ebuild/actions)
[![Rust version]( https://img.shields.io/badge/rust-1.26-blue.svg)]()
[![Latest Version](https://img.shields.io/crates/v/cargo-ebuild.svg)](https://crates.io/crates/cargo-ebuild)
[![All downloads](https://img.shields.io/crates/d/cargo-ebuild.svg)](https://crates.io/crates/cargo-ebuild)
[![Downloads of latest version](https://img.shields.io/crates/dv/cargo-ebuild.svg)](https://crates.io/crates/cargo-ebuild)

`cargo ebuild` is a Cargo subcommand that generates an
[ebuild](https://wiki.gentoo.org/wiki/Ebuild) recipe that uses
[cargo.eclass](https://gitweb.gentoo.org/repo/gentoo.git/tree/eclass/cargo.eclass)
to build a Cargo based project for [Gentoo](https://gentoo.org/)

## Installation

Install it with Emerge:

```
emerge cargo-ebuild
```

or with Cargo (using git until the crates.io version can get updated):

```
$ cargo install cargo-ebuild --git https://github.com/gentoo/cargo-ebuild
```

## Usage

You will first need to get the sources to the crate you want to install.
Your best bet is to search for the crate at [crates.io](https://crates.io)
and follow the *Repository* link. This should give you the ability to clone
the repo. Once you have cloned the repo, change into the directory and
ensure that you checkout the tag corresponding to the version you would like
to package. Lastly you will execute the `cargo ebuild` command to generate the
ebuild for that crate.

### Example

```bash
$ git clone https://github.com/cardoe/cargo-ebuild.git
$ cd cargo-ebuild
$ git checkout v0.3.0
$ cargo ebuild
$ cat cargo-ebuild-0.3.0.ebuild
```

```ebuild
# Copyright 2017-2020 Gentoo Authors
# Distributed under the terms of the GNU General Public License v2

# Auto-Generated by cargo-ebuild 0.3.0

EAPI=7

CRATES="
ansi_term-0.11.0
anyhow-1.0.26
atty-0.2.13
bitflags-1.2.0
cargo_metadata-0.9.1
clap-2.33.0
either-1.5.3
heck-0.3.1
itertools-0.8.2
itoa-0.4.4
libc-0.2.62
proc-macro-error-0.2.6
proc-macro2-1.0.5
quote-1.0.2
redox_syscall-0.1.56
ryu-1.0.0
semver-0.9.0
semver-parser-0.7.0
serde-1.0.101
serde_derive-1.0.101
serde_json-1.0.41
strsim-0.8.0
structopt-0.3.3
structopt-derive-0.3.3
syn-1.0.5
textwrap-0.11.0
time-0.1.42
unicode-segmentation-1.3.0
unicode-width-0.1.6
unicode-xid-0.2.0
vec_map-0.8.1
winapi-0.3.8
winapi-i686-pc-windows-gnu-0.4.0
winapi-x86_64-pc-windows-gnu-0.4.0
"

inherit cargo

DESCRIPTION="Generates an ebuild for a package using the in-tree eclasses."
# Double check the homepage as the cargo_metadata crate
# does not provide this value so instead repository is used
HOMEPAGE="https://github.com/cardoe/cargo-ebuild"
SRC_URI="$(cargo_crate_uris ${CRATES})"
RESTRICT="mirror"
# License set may be more restrictive as OR is not respected
# use cargo-license for a more accurate license picture
LICENSE="Apache-2.0 BSL-1.0 MIT"
SLOT="0"
KEYWORDS="~amd64"
IUSE=""

DEPEND=""
RDEPEND=""
```

## API

API documentation is available at [docs.rs](https://docs.rs/cargo-ebuild/).

## Templates

`cargo-ebuild` allows you to use a custom [tera](https://crates.io/crates/tera) template.

The built-in `base.tera` provided is the following:

``` tera
{%- block header -%}
# Copyright 2017-{{ this_year }} Gentoo Authors
# Distributed under the terms of the GNU General Public License v2

# Auto-Generated by cargo-ebuild {{ cargo_ebuild_ver }}
{% endblock %}
EAPI={%- block eapi -%}7{%- endblock %}

{% block crates -%}
CRATES="
{% for crate in crates -%}
{{ crate }}
{%- endfor -%}"
{%- endblock %}

inherit {% block inherit -%}cargo{%- endblock %}

DESCRIPTION={%- block description -%}"{{ description | trim }}"{%- endblock %}
# Double check the homepage as the cargo_metadata crate
# does not provide this value so instead repository is used
HOMEPAGE={%- block homepage -%}"{{ homepage }}"{%- endblock %}
SRC_URI={%- block src_uri -%}{% raw -%}"$(cargo_crate_uris ${CRATES})"{%- endraw %}{%- endblock %}
# License set may be more restrictive as OR is not respected
# use cargo-license for a more accurate license picture
LICENSE={%- block license -%}"{{ license }}"{%- endblock %}
SLOT={%- block slot -%}"0"{%- endblock %}
KEYWORDS={%- block keyword -%}"~amd64"{%- endblock %}
{% block variables -%}
RESTRICT="mirror"
{%- endblock %}

{%- block phases -%}
{%- endblock -%}
```

``` tera
{%- extends "base.tera" -%}

{% block variables -%}
USE="+capi"

ASM_DEP=">=dev-lang/nasm-2.14"
BDEPEND="
	amd64? ( ${ASM_DEP} )
	capi? ( dev-util/cargo-c )
"
{%- endblock %}

{% block phases -%}
src_compile() {
	export CARGO_HOME="${ECARGO_HOME}"
	local args=$(usex debug "" --release)

	cargo build ${args} \
		|| die "cargo build failed"

	if use capi; then
		cargo cbuild ${args} \
			--prefix="/usr" --libdir="/usr/$(get_libdir)" \
			|| die "cargo cbuild failed"
	fi
}

src_install() {
	export CARGO_HOME="${ECARGO_HOME}"
	local args=$(usex debug "" --release)

	if use capi; then
		cargo cinstall $args \
			--prefix="/usr" --libdir="/usr/$(get_libdir)" --destdir="${ED}" \
			|| die "cargo cinstall failed"
	fi

	cargo_src_install
}
{%- endblock -%}
```

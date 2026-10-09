# libvips

Container images with [libvips](https://www.libvips.org/), the image processing library, and its command-line tools: `vips` runs any libvips operation from the shell, `vipsthumbnail` makes thumbnails, and `vipsheader` prints the size, format and metadata of an image. libvips handles very large images in little memory because it streams them in small pieces through a pipeline of operations. It is compiled here from the upstream release on Ubuntu and Alpine, together with libheif and the HEVC decoder libde265, for `linux/amd64` and `linux/arm64`, and the images are rebuilt when libvips publishes a release and when the base image changes.

This is an unofficial build, not affiliated with or endorsed by the libvips project. Report problems with the image in this repository and libvips bugs [upstream](https://github.com/libvips/libvips/issues).

## Quick start

Make a thumbnail that fits in 800x800 pixels:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint vipsthumbnail \
  ghcr.io/randomcontainers/libvips photo.jpg -s 800 --path photo-800.jpg
```

`-s 800x` sets only the width, and `-s 800x600` a box of another shape. `--path` names the output, and its suffix picks the format; `%s` in it stands for the input name without its suffix, so `--path 'tn_%s.webp'` with several input files writes one thumbnail for each. Without `--path`, the thumbnail is written next to the input as `tn_<name>.jpg`. `vipsthumbnail` turns images upright according to their EXIF orientation, and `--smartcrop attention` crops to fill the box instead of fitting the whole image in it.

The entrypoint runs `vips` under `tini` in `/work`, and without arguments it prints the help. The first argument is the operation:

```sh
# Convert a HEIC photo to AVIF
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  ghcr.io/randomcontainers/libvips copy photo.heic photo.avif

# Halve the size of an image, and save it as a JPEG at quality 80 without metadata
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  ghcr.io/randomcontainers/libvips resize scan.tif 'scan-half.jpg[Q=80,keep=none]' 0.5

# Make a Deep Zoom tile pyramid: scan.dzi and the tiles in scan_files/
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  ghcr.io/randomcontainers/libvips dzsave scan.tif scan

# Print the size, format and loader of an image; -a prints every field
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint vipsheader \
  ghcr.io/randomcontainers/libvips photo.jpg
```

Save options go in square brackets after the output name, as in `[Q=80,keep=none]`; quote the name so that the shell leaves the brackets alone. `dzsave` writes a single zip file when the output name ends in `.zip`, and `--layout zoomify`, `google`, `iiif` or `iiif3` gives the layouts of other viewers. `vips -l` lists the operations, `vips <operation>` without arguments prints the usage of one, such as `vips resize`, and `vips -l foreign` lists the loaders and savers with their file suffixes. The [libvips documentation](https://www.libvips.org/API/current/) describes every operation and its options.

## What is in the image

| Format | Load | Save | Library |
|---|---|---|---|
| JPEG | yes | yes | libjpeg-turbo |
| PNG | yes | yes | libpng |
| GIF | yes | yes | built-in decoder, cgif and libimagequant for saving |
| WebP | yes | yes | libwebp |
| TIFF | yes | yes | libtiff |
| HEIC (HEVC) | yes | yes | libheif with libde265 and x265 |
| AVIF (AV1) | yes | yes | libheif with dav1d and aom |
| JPEG XL | yes | yes | libjxl |
| JPEG 2000 | yes | yes | OpenJPEG |
| SVG | yes | no | librsvg |
| OpenEXR | yes | no | OpenEXR |
| FITS | yes | yes | CFITSIO |
| Camera raw (DNG, CR2, NEF, ARW and others) | yes | no | LibRaw |
| PPM, PFM, Radiance HDR, CSV, matrix, `.v` | yes | yes | built-in |
| Analyze | yes | no | built-in |
| Deep Zoom, Zoomify, Google Maps and IIIF tile pyramids | no | yes | built-in, libarchive for zip output |

libvips also uses Little CMS for ICC color management, libexif for EXIF metadata, FFTW for Fourier transforms (`fwfft`, `invfft`, `freqmult`), Highway for SIMD code, and Pango, fontconfig and the DejaVu fonts for `vips text`. Every library except libheif and libde265 is the Ubuntu or Alpine package. The distro libde265 packages lack upstream security fixes, so libde265 and libheif are compiled here from their upstream releases, with the codecs built into libheif: libde265 and dav1d for decoding, x265 and aom for encoding.

Not included: PDF loading (Poppler or PDFium), ImageMagick and GraphicsMagick formats, OpenSlide whole-slide images, Matlab and NIfTI files, Ultra HDR JPEG, the C++ API, GObject introspection data and the C headers. The meson options, the libheif and libde265 versions and the output of `vips --vips-config` are in `/usr/local/share/randomcontainers/libvips/buildinfo`.

## Default or slim

The default image adds no other tools, so `latest` and `slim` are the same image: `vips`, `vipsthumbnail`, `vipsheader` and `vipsedit`, and the libraries they link against. Use `latest` to run it and the `slim` tags as a base for your own image.

## Tags

`<version>` is a libvips release such as `8.18.7`. `<minor>` and `<major>` are its shorter forms, `8.18` and `8`, and follow the newest release in that series. Each row lists the default tag and its `slim` twin, which point to the same image.

| Tags | Base |
|---|---|
| `latest`, `slim` | Ubuntu |
| `<version>`, `<version>-slim` | Ubuntu |
| `<minor>`, `<minor>-slim`, `<major>`, `<major>-slim` | Ubuntu |
| `ubuntu`, `slim-ubuntu` | Ubuntu |
| `<version>-ubuntu`, `<version>-slim-ubuntu` | Ubuntu |
| `<minor>-ubuntu`, `<minor>-slim-ubuntu`, `<major>-ubuntu`, `<major>-slim-ubuntu` | Ubuntu |
| `<version>-ubuntu26.04`, `<version>-slim-ubuntu26.04` | Ubuntu 26.04 |
| `alpine`, `slim-alpine` | Alpine |
| `<version>-alpine`, `<version>-slim-alpine` | Alpine |
| `<minor>-alpine`, `<minor>-slim-alpine`, `<major>-alpine`, `<major>-slim-alpine` | Alpine |
| `<version>-alpine3.24`, `<version>-slim-alpine3.24` | Alpine 3.24 |

The images are currently built on Ubuntu 26.04 and Alpine 3.24. Tags without a distro version move to the next distro release when the project does; tags ending in `ubuntu26.04` or `alpine3.24` stay on that release and are no longer rebuilt once the project moves to the next one. Every tag of the current libvips version, including the exact version, is rebuilt in place (see [Updates](#updates)), so pin a digest when you need the same bytes every time.

## Platforms

`linux/amd64` and `linux/arm64`, for both Ubuntu and Alpine. Both are compiled natively on GitHub-hosted runners, without emulation.

## Files and permissions

The working directory is `/work`. The image runs as UID 1000, and any other UID works too: `HOME` is then `/`, and caches go to `/cache`, which anyone can write to. How to get output files owned by you depends on how you run containers:

| Runtime | Flag |
|---|---|
| Docker on Linux (rootful), GitHub Actions | `--user "$(id -u):$(id -g)"` |
| Rootless Podman | `--userns=keep-id` |
| Rootless Docker | `--user 0:0` (root in the container is your user on the host) |
| Docker Desktop on macOS or Windows | none, file ownership is mapped for you |

With `--read-only`, add `--tmpfs /cache`, so that fontconfig can write its cache if it needs to, and `--tmpfs /tmp` for the temporary files of large images.

## Untrusted files

libvips marks the loaders and savers that are not hardened for untrusted input, and `vips -l foreign` shows `untrusted` on their lines. In this build they are the loaders for SVG, JPEG XL, JPEG 2000, OpenEXR, FITS, camera raw, PPM, Radiance, Analyze, CSV and raw pixel files and for libvips' own `.v` format, and the JPEG XL, JPEG 2000 and FITS savers. With `VIPS_BLOCK_UNTRUSTED` set to any value, libvips refuses to run them, and JPEG, PNG, GIF, WebP, TIFF, HEIC and AVIF stay available. libvips counts its HEIC and AVIF loader as hardened only when it is built against libheif 1.23.2 or later, which is one reason libheif is compiled here. For files from unknown sources, also take away what the container does not need:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" -e VIPS_BLOCK_UNTRUSTED=1 \
  --network none --read-only --tmpfs /tmp --tmpfs /cache --cap-drop ALL --security-opt no-new-privileges \
  --memory 1g --cpus 1 -e VIPS_CONCURRENCY=1 --pids-limit 64 \
  ghcr.io/randomcontainers/libvips thumbnail upload.heic thumbnail.jpg 800
```

## Memory, disk and threads

- libvips starts one worker thread per CPU it can see, and `docker run --cpus` does not change that number. Set `VIPS_CONCURRENCY` to the number of CPUs you give the container.
- When an operation needs random access to an image in a format that can only be read from start to end, such as PNG or JPEG, libvips decompresses the image first: into memory up to 100 MB of pixels, and into a temporary file in `/tmp` above that. `VIPS_DISC_THRESHOLD` changes the limit (for example `500m`), and `TMPDIR` the directory. Mount a volume there for large jobs, or add `--tmpfs /tmp` when you run the container with `--read-only`.

## Extending the slim image

Use a `slim` tag as the base for your own image. `slim`, `slim-ubuntu` and `slim-alpine` move to each new libvips release and are rebuilt when the base image changes. Switch to root to install more, for example GNU parallel to make thumbnails of many files at once, then back:

```dockerfile
FROM ghcr.io/randomcontainers/libvips:slim-ubuntu@sha256:...
USER root
RUN apt-get update \
 && apt-get install -y --no-install-recommends parallel \
 && rm -rf /var/lib/apt/lists/*
USER 1000:1000
```

On Alpine, start from `slim-alpine` and use `apk add --no-cache parallel`. Fonts installed the same way are available to `vips text` through fontconfig. The entrypoint is `["tini", "--", "vips"]`; set your own `ENTRYPOINT` if your image runs something else. To pick up new libvips releases and base image fixes, let Dependabot or Renovate update the digest in your `FROM` line.

Everything the image adds is under `/usr/local`. `/usr/local/share/randomcontainers/libvips/` holds the version, the source URLs, the build options, the license files and `runtime-deps`, the list of distro packages the tools need at run time. The [ImageMagick](https://github.com/randomcontainers/imagemagick) and [libheif](https://github.com/randomcontainers/libheif) images have their own builds of libheif and libde265 at the same paths, so build on one of these images instead of copying one into another.

## Verifying

Each image has a build provenance attestation from this repository's GitHub Actions run, signed by the shared build workflow in `randomcontainers/ci`:

```sh
gh attestation verify oci://ghcr.io/randomcontainers/libvips:latest \
  --repo randomcontainers/libvips --signer-repo randomcontainers/ci
```

Each platform image also carries an SPDX SBOM that lists every distro package with its version:

```sh
docker buildx imagetools inspect ghcr.io/randomcontainers/libvips:latest --format '{{ json .SBOM }}'
```

libvips, libheif and libde265 are compiled here, so they are not in the SBOM and image scanners do not check them. Their versions are in `/usr/local/share/randomcontainers/libvips/buildinfo`. The Ubuntu image also contains Ubuntu's `libheif1` package, which librsvg pulls in through gdk-pixbuf and its glycin image loaders, and scanners report its advisories. libvips does not load it: its RUNPATH makes it load the libheif in `/usr/local/lib`.

The build downloads `vips-<version>.tar.xz` from the libvips release on GitHub and the libheif and libde265 archives from theirs, and checks each one against the SHA-256 recorded in `package.yml` before compiling.

## Updates

The project checks the releases of [libvips/libvips](https://github.com/libvips/libvips/releases) every 15 minutes. A release is picked up once it is 24 hours old; release candidates are skipped. Its tarball is checked against the `vips-<version>.tar.xz.sha256sum` file published with the release. libvips publishes no signatures, and the checksum file comes from the same GitHub release as the tarball. The new version and the tarball's SHA-256 are then committed to `package.yml`, and the images are rebuilt. Only the newest release is built; tags of older versions stay as they were last built.

The images of the current version are also rebuilt when the Ubuntu or Alpine base image changes and at least every 7 days, so distro security fixes reach the current tags.

libheif and libde265 are pinned to a version in `package.yml` and do not follow their upstream releases automatically. Updating one is a commit to `package.yml`, which rebuilds the images of the current libvips version.

## Building

```sh
docker build -f Dockerfile.ubuntu --target slim \
  --build-arg VERSION=<version> \
  --build-arg SOURCE_SHA256=<sha256 from package.yml> \
  --build-arg LIBDE265_VERSION=<version> \
  --build-arg LIBDE265_SHA256=<sha256> \
  --build-arg LIBHEIF_VERSION=<version> \
  --build-arg LIBHEIF_SHA256=<sha256> \
  -t libvips:local .
```

The `LIBDE265_*` and `LIBHEIF_*` values are the `version` and `sha256` of the `extra-artifacts` entries in `package.yml`. Use `Dockerfile.alpine` for the Alpine image. `--build-arg JOBS=<n>` limits the number of parallel compile jobs.

## Licenses

libvips is licensed under the GNU Lesser General Public License, version 2.1 or later (LGPL-2.1-or-later). Parts of it have their own licenses:

- MIT: libnsgif, the GIF decoder (`COPYING.libnsgif`).
- The Radiance Software License, versions 2.0 and 1.0: the Radiance HDR code in `libvips/foreign/radiance.c`, `libvips/colour/float2rad.c` and `libvips/colour/rad2float.c` (`COPYING.radiance`). Version 2.0 has the same text as BSD-3-Clause-LBNL; version 1.0 has no SPDX identifier. This product includes Radiance software (http://radsite.lbl.gov/) developed by the Lawrence Berkeley National Laboratory (http://www.lbl.gov/).
- CC0-1.0: the built-in sRGB, Display P3 and grey ICC profiles.
- Public domain: the built-in CMYK profile ("Chemical proof" by Graeme W. Gill) and the Analyze header definitions.

libheif and libde265 are licensed under the GNU Lesser General Public License, version 3 or later (LGPL-3.0-or-later). libvips also links libimagequant, licensed under the GNU General Public License, version 3 or later, and FFTW, licensed under version 2 or later. libheif links the HEVC encoder x265, also licensed under version 2 or later. All three come from the distro, so the libvips programs and library in the image are distributed under GPL-3.0-or-later, together with the notices of the licenses listed above. The image's license label is `LGPL-2.1-or-later AND MIT AND BSD-3-Clause-LBNL AND LicenseRef-Radiance-1.0 AND CC0-1.0 AND LicenseRef-Public-Domain AND LGPL-3.0-or-later AND GPL-2.0-or-later AND GPL-3.0-or-later`, where the two `LicenseRef-` entries stand for the Radiance Software License 1.0 and the public-domain parts. The license files are in `/usr/local/share/randomcontainers/libvips/licenses/`: libvips' `LICENSE`, the files named above, `COPYING.libheif` and `COPYING.libde265`. The other libraries and the fonts from Ubuntu or Alpine keep their own licenses, for example BSD-2-Clause for aom and dav1d and the DejaVu font license. The SBOM lists them.

The corresponding source for each image:

- libvips, libheif and libde265: every libvips version has a GitHub release in this repository, named `v<version>`, with the exact `vips-<version>.tar.xz`, `libheif-<version>.tar.gz` and `libde265-<version>.tar.gz` that were compiled. The build applies no patches. `/usr/local/share/randomcontainers/libvips/source` lists that release and the download URLs of the three archives. When libheif or libde265 is updated, the release also keeps the archive of the earlier version.
- Build scripts: this repository at the commit in the image's `org.opencontainers.image.revision` label. The Dockerfiles hold every meson and CMake option.
- Ubuntu packages: the source packages on [Launchpad](https://launchpad.net/ubuntu) for the versions listed in the SBOM. `apt-get source <package>=<version>` fetches a version that is still in the Ubuntu archive.
- Alpine packages: Alpine has no source packages. For the versions listed in the SBOM, the source is the APKBUILD and patches in [aports](https://gitlab.alpinelinux.org/alpine/aports/-/tree/3.24-stable), branch `3.24-stable`, and the archives on [distfiles.alpinelinux.org](https://distfiles.alpinelinux.org/distfiles/v3.24/).

HEVC, the codec of HEIC files, and AV1 may be covered by patents in some countries. HEIC files are encoded with x265 and decoded with libde265, AVIF files encoded with aom and decoded with dav1d. Check what applies where you use them.

The files in this repository are available under the MIT license, see [LICENSE](LICENSE).

## Requesting a tool

To suggest another tool, use the [Request a tool](https://github.com/randomcontainers/.github/issues/new?template=tool-request.yml) form.

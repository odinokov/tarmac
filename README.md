# tarmac

[![License: MIT](https://img.shields.io/github/license/odinokov/tarmac)](LICENSE)
[![Shell: Bash](https://img.shields.io/badge/shell-bash-4EAA25?logo=gnubash&logoColor=white)](tarmac)
[![Platform](https://img.shields.io/badge/platform-linux%20%7C%20macOS-lightgrey)](#requirements)

Archive and restore directories as `.tar.gz` using all your cores.

`tarmac` wraps `tar` + `pigz` + `pv` in a single script: parallel compression,
a progress bar with ETA, a free-space check before it starts, atomic writes,
and path validation before extraction. It is a safer, faster `tar czf` /
`tar xzf` for backups of large directories.

```bash
tarmac a ./project ./backups        # -> ./backups/project_20260908_165811.tar.gz
tarmac d ./backups/project_20260908_165811.tar.gz ./restore
```

## Requirements

- `bash` ≥ 4
- GNU `tar`
- `pigz`
- `pv`
- GNU coreutils (`df`, `du`, `nproc`, `numfmt`, `realpath`, `mktemp`, …) and `awk`

Debian/Ubuntu:

```bash
sudo apt install tar pigz pv coreutils
```

macOS (Homebrew). The script needs the GNU versions of `tar` and coreutils,
so put their `gnubin` directories first on `PATH`:

```bash
brew install pigz pv coreutils gnu-tar
export PATH="$(brew --prefix coreutils)/libexec/gnubin:$(brew --prefix gnu-tar)/libexec/gnubin:$PATH"
```

`tarmac` checks for every tool it needs at startup and names the missing one.

## Install

Single file, no build step:

```bash
mkdir -p ~/.local/bin
curl -fsSL https://raw.githubusercontent.com/odinokov/tarmac/main/tarmac -o ~/.local/bin/tarmac
chmod +x ~/.local/bin/tarmac
tarmac --help
```

Or from a clone:

```bash
git clone https://github.com/odinokov/tarmac.git
sudo install -m 0755 tarmac/tarmac /usr/local/bin/tarmac
```

## Usage

```
tarmac [-j jobs] a <target_directory> [destination_directory]
tarmac [-j jobs] d <archive.tar.gz> [destination_directory]
```

`a` archives, `d` extracts. Destination defaults to the current directory and
is created if missing. Options go before the subcommand.

| Option | Meaning |
| --- | --- |
| `-j N`, `--jobs N`, `--cpu N`, `--cpus N` (or `--jobs=N`) | pigz worker threads. Default: cores − 1, capped at 8. |
| `-h`, `--help`, `help` | Show usage. |
| `--` | End of options. |

### Examples

```bash
tarmac a ./project                      # archive into the current directory
tarmac a ./project ./backups            # archive into ./backups
tarmac -j 4 a ./project ./backups       # limit pigz to 4 threads
tarmac d ./backups/project_20260908_165811.tar.gz ./restore
tarmac --cpu=4 d project_20260908_165811.tar.gz
```

What a run looks like:

```
[INFO] Source:      /data/project
[INFO] Destination: /backups/project_20260908_165903.tar.gz
[INFO] Size:        77M
[INFO] Workers:     7
[RUN]  Archiving...
76.3MiB 0:00:00 [ 246MiB/s] [================================>] 100%
[OK]   Archive created
[OK]   Done: /backups/project_20260908_165903.tar.gz
```

## Behaviour

- Archives are named `<directory>_YYYYMMDD_HHMMSS.tar.gz` and contain
  `<directory>/` as their top-level entry.
- Compression level is fixed at 6.
- Before archiving, the source size must fit in the destination's free space.
  Before extracting, the uncompressed size must fit.
- Archives are written to a hidden temp file in the destination and moved into
  place only after `tar` and `pigz` both succeed. The temp file is removed on
  failure or interrupt.
- If the destination lies inside the source directory, it is excluded from the
  archive.
- Extraction refuses archives containing absolute paths, empty names, or `..`
  components, and unpacks with `--no-same-owner`.
- Extraction decompresses the archive three times: once to validate paths, once
  to measure the unpacked size, once to extract. Safe, but CPU-bound on very
  large archives.
- The archive progress bar is sized from `du -sb`, so it can tick slightly past
  100 % because of tar header overhead.
- Colour output only when stdout is a terminal. Set `NO_COLOR=1` to disable it.

## License

MIT. See [LICENSE](LICENSE).

# tarmac

`tarmac` is a small Bash utility for archiving and restoring directories with
parallel gzip compression and progress output.

## Requirements

Install these command-line tools first:

- `bash`
- `tar`
- `gzip`
- `pigz`
- `pv`
- GNU core utilities: `awk`, `df`, `du`, `realpath`, `numfmt`, `mktemp`, `wc`

On Debian/Ubuntu:

```bash
sudo apt install tar gzip pigz pv coreutils gawk
```

On macOS with Homebrew:

```bash
brew install pigz pv coreutils gawk
```

## Install

```bash
git clone https://github.com/odinokov/tarmac.git
cd tarmac
chmod +x tarmac
sudo install -m 0755 tarmac /usr/local/bin/tarmac
```

Check that it is available:

```bash
tarmac --help
```

If `/usr/local/bin` is not in your `PATH`, either add it or install to another
directory that is already in `PATH`, such as `~/.local/bin`:

```bash
mkdir -p ~/.local/bin
install -m 0755 tarmac ~/.local/bin/tarmac
```

## Usage

Archive a directory:

```bash
tarmac [-j jobs] a <target_directory> [destination_directory]
```

Decompress and restore an archive:

```bash
tarmac [-j jobs] d <archive.tar.gz> [destination_directory]
```

Destination defaults to the current directory.

## CPU Workers

By default, `tarmac` uses most CPU cores, capped at 8, while leaving one core
free. Override this with `-j`, `--jobs`, `--cpu`, or `--cpus`:

```bash
tarmac -j 4 a ./project ./backups
tarmac --cpu=4 d ./backups/project_20260429_142927.tar.gz ./restore
```

## Examples

Create an archive in the current directory:

```bash
tarmac a ./project
```

Create an archive in a backup directory:

```bash
tarmac a ./project ./backups
```

Restore an archive into `./restore`:

```bash
tarmac d ./backups/project_20260429_142927.tar.gz ./restore
```

## Notes

- Archives are named `<directory>_YYYYMMDD_HHMMSS.tar.gz`.
- Archives are written to a temporary file in the destination directory and
  moved into place only after `tar` and `pigz` finish successfully.
- Extraction verifies the archive first and rejects unsafe absolute or `../`
  paths.
- Set `NO_COLOR=1` to disable colored terminal output.

## License

MIT. See [LICENSE](LICENSE).

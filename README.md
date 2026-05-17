# pkgfilter

An Arch Linux bloat hunter. Queries every installed package via `pacman`, then lets you progressively strip out packages you know you need — what's left is your candidate removal list. Optionally scans the filesystem for large directories and files to find disk hogs outside of packages.

---

## Requirements

- Arch Linux (uses `pacman`)
- Python 3.6+
- No third-party libraries

---

## Setup

Put `pkgfilter.py` in its own empty folder. That folder is where all revision files will be saved.

```
mkdir ~/pkgfilter
cp pkgfilter.py ~/pkgfilter/
cd ~/pkgfilter
```

---

## How it works

Each run produces the next `revisionN.txt` file. The idea is to start with everything installed and filter down by stripping out packages you know belong — plasma, kde, qt, fonts, whatever your environment needs. Whatever's left in the final revision is what you don't recognize and might want to remove.

```
revision1.txt  ← full package list from pacman (auto-created on first run)
revision2.txt  ← after stripping 'plasma'
revision3.txt  ← after stripping 'kde'
revision4.txt  ← after stripping 'qt' and applying a size filter
...
```

Every revision file stores package names alongside their installed sizes in bytes, so size data carries through without re-querying pacman on every run.

---

## Usage

### Interactive (recommended)

```bash
python3 pkgfilter.py
```

You'll be walked through each step with prompts.

### Pass flags directly (no prompts)

```bash
python3 pkgfilter.py -k plasma
python3 pkgfilter.py -s 5MiB
python3 pkgfilter.py -k kde -s 10MiB
```

When any flag is provided, all interactive prompts are suppressed — unset options are skipped automatically.

### Flags

| Flag | Description |
|------|-------------|
| `-k WORD` / `--keyword WORD` | Strip all packages whose name contains `WORD` |
| `-s SIZE` / `--minsize SIZE` | Drop packages smaller than `SIZE` (e.g. `5MiB`, `500KiB`, `1GiB`) |
| `-h` / `--help` | Show help and exit |

---

## Run order (interactive session)

```
1. Load packages
      First run: queries pacman -Qq and pacman -Qi for sizes → saves revision1.txt
      Later runs: loads the latest revisionN.txt

2. Keyword loop
      Enter keywords one at a time to strip matching packages
      Each match shows you what was removed and its size
      Blank input or 'quit' exits the loop

3. Size filter
      Optional — drop everything below a size threshold
      Good for cutting tiny libs after you've stripped the big groups

4. Save + display
      Saves next revision file
      Prints all remaining packages sorted largest → smallest with a total

5. Filesystem scan (optional)
      Prompts: "Scan filesystem for large files/folders? [y/N]"
      If yes, prompts for a path — defaults to your home directory
      Shows top 20 largest directories and top 20 largest individual files
```

---

## Running as sudo

```bash
sudo python3 pkgfilter.py
```

Recommended if you want to scan outside your home directory (e.g. `/`, `/var`, `/opt`). The script detects `SUDO_USER` and still defaults the filesystem scan to your actual home, not `/root`.

Worth scanning `/var/cache/pacman/pkg` — pacman's package cache grows large over time and is a common disk hog unrelated to what's currently installed.

---

## Suggested keyword filter order for KDE Plasma

The goal is to strip out things you *want to keep* so that what remains is unfamiliar and worth scrutinizing.

```
plasma     ← Plasma shell and session packages
kde        ← KDE apps and frameworks
qt         ← Qt libraries
lib        ← shared libraries (optional, aggressive)
font       ← fonts
python     ← Python and python-* packages (if not needed)
```

After those passes, apply a size filter like `5MiB` to cut anything too small to be worth removing. The final list is your bloat candidates.

---

## Revision files

Stored in the same folder as the script. Format is two tab-separated columns:

```
packagename    <installed bytes>
```

Safe to open in any text editor. You can also diff revisions to see exactly what each pass removed:

```bash
diff revision2.txt revision3.txt
```

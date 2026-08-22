# Linux Text Manipulation

## Table of Contents

- [Introduction](#introduction)
- [Basic File Viewing](#basic-file-viewing)
- [Head and Tail Commands](#head-and-tail-commands)
- [Line Numbering](#line-numbering)
- [Filtering with grep](#filtering-with-grep)
- [Find and Replace with sed](#find-and-replace-with-sed)
- [Paged Viewing with More and Less](#paged-viewing-with-more-less)
- [Key Tips](#key-tips)
- [Quick Command Chains](#quick-command-chains)
- [Quick Start](#quick-start)

---

## Introduction

In Linux, almost everything is stored as a file, and configuration files are predominantly plain text. To reconfigure applications or analyze logs (like Snort NIDS), mastering text manipulation commands is essential for effective system administration and ethical hacking.

---

## Basic File Viewing

| Command | Usage | Example |
| --- | --- | --- |
| `cat <file>` | Display entire file content at once | `cat /etc/snort/snort.conf` |
| `cat > <file>` | Create interactive file (overwrite) | `cat > notes.txt` |
| `cat >> <file>` | Append text to an existing file | `cat >> notes.txt` |

---

## Head and Tail Commands

| Command | Usage | Example |
| --- | --- | --- |
| `head <file>` | Display the first 10 lines (default) | `head /etc/snort/snort.conf` |
| `head -n <file>` | Display first $n$ lines | `head -20 /etc/snort/snort.conf` |
| `tail <file>` | Display the last 10 lines (default) | `tail /etc/snort/snort.conf` |
| `tail -n <file>` | Display last $n$ lines | `tail -20 /etc/snort/snort.conf` |
| `tail -n+<n> <file>` | Output starting from line number $n$ | `tail -n+507 /etc/snort/snort.conf` |

```bash
# View first 20 lines of a file
head -20 /etc/snort/snort.conf

# View last 20 lines of a file
tail -20 /etc/snort/snort.conf

```

---

## Line Numbering

| Command | Usage | Example |
| --- | --- | --- |
| `nl <file>` | Display file contents with line numbers (skips blank lines) | `nl /etc/snort/snort.conf` |

```bash
# Numbering lines for easier referencing in large files
nl /etc/snort/snort.conf

```

---

## Filtering with grep

| Command | Usage | Example |
| --- | --- | --- |
| `grep <keyword> <file>` | Search for a specific word/phrase | `grep output /etc/snort/snort.conf` |
| `<command> | grep <keyword>` | Filter piped command output | `cat /etc/snort/snort.conf | grep output` |

```bash
# Filter and display only lines containing 'output'
cat /etc/snort/snort.conf | grep output

```

---

## Find and Replace with sed

`sed` (Stream Editor) allows searching and performing actions like find-and-replace on text streams.

| Syntax | Description | Example |
| --- | --- | --- |
| `sed s/old/new/g <file>` | Replace **all** occurrences globally | `sed s/mysql/MySQL/g snort.conf > snort2.conf` |
| `sed s/old/new/ <file>` | Replace **only first** occurrence per line | `sed s/mysql/MySQL/ snort.conf > snort2.conf` |
| `sed s/old/new/N <file>` | Replace **only Nth** occurrence | `sed s/mysql/MySQL/2 snort.conf > snort2.conf` |

```bash
# Replace 'mysql' with 'MySQL' globally and save to a new file
sed s/mysql/MySQL/g /etc/snort/snort.conf > snort2.conf

# Replace only the 2nd instance of 'mysql'
sed s/mysql/MySQL/2 snort.conf > snort2.conf

```

---

## Paged Viewing with More and Less

| Command | Usage | Key Controls |
| --- | --- | --- |
| `more <file>` | Display output page-by-page | `Enter` (next line), `Space` (next page), `q` (quit) |
| `less <file>` | Advanced page display with interactive search | `/keyword` (search), `n` (next match), `q` (quit) |

```bash
# Open file in less interactive view
less /etc/snort/snort.conf

# Inside 'less', search for a term:
/output
# Press 'n' to jump to the next matching term

```

---

## Key Tips

* **Case Sensitivity** — `sed` and `grep` are case-sensitive by default (`mysql` $\neq$ `MySQL`).
* **Blank Lines in `nl**` — `nl` automatically skips numbering blank lines.
* **Global Flag (`g`) in `sed**` — Omitting `g` in `sed` will only replace the first match on each line.
* **Stream Redirection (`>`)** — Using `>` redirects command output into a new file rather than displaying it in the terminal.
* **Search Navigation in `less**` — Type `/` followed by a string inside `less` to find keywords, then press `n` for subsequent matches.

---

## Quick Command Chains

```bash
# Combine nl and grep to locate line numbers of matches
nl /etc/snort/snort.conf | grep output

# Extract a specific line range (from line 507, print 6 lines)
tail -n+507 /etc/snort/snort.conf | head -n 6

# Find, replace globally, and verify result
sed s/mysql/MySQL/g /etc/snort/snort.conf > snort2.conf && grep MySQL snort2.conf

```

---

## Quick Start

Open terminal $\rightarrow$ Use `head`/`tail` to preview files $\rightarrow$ Use `nl` to find line numbers $\rightarrow$ Filter text with `grep` $\rightarrow$ Modify contents with `sed` $\rightarrow$ Inspect interactive output using `less`

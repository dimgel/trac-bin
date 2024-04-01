# dimgel's trac

Issue tracker. **Free, closed-source.**

This is **MVP** (minimum viable product), it's even **single-user, for personal home use**.

Started as proof of concept: website written in C++ as nginx module, plus some ideas on webapp efficiency.

Short presentation (in Russian): [https://youtu.be/Fkqxg-3j4TI](https://youtu.be/Fkqxg-3j4TI)

Contact: [dimgel@mail.ru](mailto:dimgel@mail.ru)

## Design goals (all achieved)

- **Very** small -- distro size is less than 1MB.
- No heavy dependencies -- besides `glibc` and `gcc-libs` (C++ stdlib), the heaviest dep is [lucene++](https://github.com/luceneplusplus/LucenePlusPlus) fulltext search library which depends on [boost](https://www.boost.org).
- Easy to install -- just copy files and add virtual server to `nginx.conf` (see below).
- **Very** fast & traffic-efficient -- yeah both: just open `F12` / `Network` in your browser, see response sizes & timings -- and try to believe your eyes!
- Server-side is CPU and RAM efficient.
- Requires no maintenance.

## Features

### What is done:

- Projects / issues.
- Manageble labels to tag issues, global.
- Search issues by labels & fulltext, per-project.
- Issues edit history.
- English & Russian languages.

### What is not:

- Multi-user, comments, file uploads, VCS integration, wiki, milestones, API, etc.

## System requirements

### Server-side

- Linux / x86-64.
- You must have `nginx` installed & working, and be able to edit its config.
- Written in C++, compiled with `gcc` against `glibc` (on [ArtixLinux](https://artixlinux.org/)).
- Uses embedded [SQLite](https://www.sqlite.org/), so separate SQL server setup is **not** needed.

### Client-side

Modern browsers which support [DecompressionStraam](https://caniuse.com/?search=DecompressionStream). I tested with [LibreWolf](https://librewolf.net/) 115+ and [Ungoogled Chromium](https://github.com/ungoogled-software/ungoogled-chromium) 117+. I recommend Chrome: Firefox does not colorize `<option>`-s and has some selection & undo bugs in WYSIWYG HTML editor I couldn't completely work around.


## Installation

Install required libraries via your package manager. On ArchLinux:

     pacman -S expat gcc-libs lucene++ sqlite xxhash zlib

Extract archive. Copy `files` subdir contents to `/`, or anywhere. Just preserve directory structure: I locate my `lib/dimgel-trac1` directory as relative to config file path specified in `nginx.conf` (see below).

You don't need to edit config file -- unless your `/var/lib` is network-mounted, or you want multiple `nginx` virtual servers running `trac`.

Add to `/etc/nginx/nginx.conf` (here I assume you copied files to `/`):

    load_module /lib/dimgel_trac1_ngxmod.so;
    ...
    http {
        ...
        server {
            listen 80;
            server_name trac;
            dimgel_trac1 /etc/dimgel-trac1.conf;
        }
    }

Restart `nginx`.

Add to `/etc/hosts`:

    127.0.0.1 trac

Point your browser to: [http://trac/](http://trac/). If your browser auto-redirects to `https://`, and you don't have SSL configured in your `nginx.conf`, then edit URL back to `http://`.

If you don't see `trac` homepage, see "Troubleshooting" section below.

Before you start creating issues, choose langunage in settings: it affects not only interface localization, but also `lucene++` indexing rules. Or you can always stop `nginx` and delete `lucene++` data subdirectory, then on start `trac` will re-create FTS index using current language setting.


## HTTPS?

[https://nginx.org/en/docs/http/configuring_https_servers.html](https://nginx.org/en/docs/http/configuring_https_servers.html)


## Backup / Restore

Please stop `nginx` before backup / restore.

You need to backup only 2 files in data directory: `main.sqlite3` and (if exists) `main.sqlite3-wal`:

- Missing `lucene++` subdirectory will be rebuilt on start (note: this may take a while).
- As well as empty `internal-server-errors` subdirectory.
- File `main.sqlite-shm` (if exists) is just in-memory index into `-wal` which is (re)created automatically by SQLite.

On restore, make sure to clear data directory first!

## Upgrade

1. Stop `nginx`.
2. Backup database (see above) & trac itself (just in case).
3. Overwrite files in `lib` (make sure **NO** old version files are left in `lib/dimgel-trac1`), overwrite/merge config in `etc`.
4. Start `nginx`.

Stopping `nginx` **AFTER** overwriting `lib` files may result in `nginx` worker processes crash with `SIGSEGV`.


## Troubleshooting

### You get "`500 Internal Server Error`" page

- Check `nginx` error log.
- Check files in `var/lib/dimgel-trac1/internal-server-errors` directory (you can delete files, but don't delete directory itself while `nginx` is running).
- Investiage, and maybe report me a bug.

### Nginx freezes on shut down

- If you have huge database (**you don't**, but to be pedantic...), then you might have occasionally hit very rare case of SQLite performing [database maintenance](https://www.sqlite.org/pragma.html#pragma_optimize) scheduled in background thread. Just wait a bit.

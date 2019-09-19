# make-portable
Simple shell script that bundles everything needed for a GNU/Linux application to run on a different system (even if target system's glibc/libstdc++ versions are older than those of source system!)

# How it works
The script copies the specified binary, all its dynamically linked libraries (including glibc and, if appropriate, libstdc++), and source system's linker (ld) into a tarball. A wrapper script is also created. Both the tarball and wrapper script are placed in user's home directory.

# Usage directions
1. Install the desired application on the source GNU/Linux system the usual way (from repository or source code)
2. Run this script on the source system, passing the name of the binary as the sole argument
3. Copy the resulting tarball and wrapper script to the target GNU/Linux system
4. Use the wrapper script as you'd use the application

# Usage example
Main reason I created this script is so that I can build the newest mpv media player on my personal laptop, then run it on my digital media player, which uses an extremely ancient GNU/Linux OS. So I'll use mpv as an example.

1. Build newest mpv from source on my laptop, install it, make sure it works and is in my PATH
2. Run this command on my laptop: `$ make-portable mpv`
3. Copy $HOME/mpv.tgz and $HOME/mpv from laptop to media player
4. In media player: `$ /path/to/mpv /path/to/nicemovie.mkv`

# Dependencies
coreutils, grep, awk, tar, and ldd (in Debian, ldd is part of libc-bin). I tried to use only tools that are expected to be present on every GNU/Linux system, so you shouldn't need to install anything on source or target OS. Also, root privileges are not needed to create or run the portable application.

# Caveats
1. If the application is a single binary written in C/C++ that needs only glibc, libstdc++, and ancillary shared libraries, then *make-portable* takes care of everything automatically for you. If the application consists of multiple binaries and/or has data files, then manual tweaks will be needed.
2. Bloat. The portable application will be bigger and use more RAM than a traditional installation of the same application.
3. The portable application will contain only the shared libraries that the binary is linked to. If the application needs to load additional libraries while it's running, it may look for those libraries in the places where the libraries normally reside in the *source* OS. Sometimes there's an environmental variable you can use in the target OS that fixes the problem (for example, if app can't find libGL dri drivers, use `export LIBGL_DRIVERS_PATH=/path/to/dri/`). If there isn't an environmental variable you can use, you can always fall back on creating symlinks in the target OS (pointing from location of library in source OS to correct location of library in the target OS).
4. Needing a wrapper script on the target system is clunky. A more elegant alternative is to use my script to collect things on source system (binary, linker, libraries), put these in a permanent place on target system, then use patchelf (https://nixos.org/patchelf.html) to patch the binary (so it points to the transplanted linker and libraries) and patch the libraries (so they can find their fellow transplanted libraries). I'll use mpv as an example of how to do this:
  i. Run `$ make-portable` on source system to collect what we need
  ii. Copy mpv.tgz to target system's */tmp* directory
  iii. Follow run these commands on target system:
  ```
  $ cd /tmp
  $ tar -xvzf mpv.tgz
  $ sudo mkdir -p /var/lib/mpv
  # move the transplanted ld unaltered to /var/lib/mpv:
  $ sudo mv /tmp/lib/ld /var/lib/mpv/
  # patch the libraries so that they can find one another, then move them to /var/lib/mpv:
  $ for library in /tmp/lib/*; do patchelf --set-rpath /var/lib/mpv $library; done
  $ sudo mv /tmp/lib/* /var/lib/mpv/
  # patch the binary so that it can find the transplanted ld and libraries, then move it to /usr/local/bin:
  $ patchelf --set-rpath /var/lib/mpv /tmp/bin
  $ patchelf --set-interpreter /var/lib/mpv/ld /tmp/bin
  $ sudo mv /tmp/bin /usr/local/bin/mpv
```

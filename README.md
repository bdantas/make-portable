# make-portable
Simple shell script that bundles everything needed for a GNU/Linux binary to run on a different system, even if target system's glibc/libstdc++ versions are older than those of source system

# How it works
The script copies the specified binary, all its dynamically linked libraries (including glibc and, if appropriate, libstdc++), and source system's linker (ld) into a tarball. A wrapper script is also created. Both the tarball and wrapper script are placed in user's home directory.

# Usage directions
1. Install the desired application on the source GNU/Linux system the usual way (from repository or source code)
2. Run this script on the source system, passing the name of the binary as the sole argument
3. Copy the resulting tarball and wrapper script to the target GNU/Linux system (you can copy them into any directory on target system, just make sure the tarball and wrapper script stay together in the same directory)
4. Use the wrapper script as you'd use the application

# Usage example
Main reason I created this script is so that I can build the newest mpv media player on my personal laptop, then run it on my digital media player, which uses an extremely ancient GNU/Linux OS. So I'll use mpv as an example.

1. Build newest mpv from source on my laptop, install it, make sure it works and is in my PATH
2. Run this command on my laptop: `$ make-portable mpv`
3. Copy *mpv.tgz* and *mpv* from laptop's home directory to media player's */usr/local/bin/*
4. In media player: `$ mpv /path/to/nicemovie.mkv`

# Dependencies
grep, awk, tar, and ldd. In other words, the dependencies are minimal and should be present on every GNU/Linux system. Also, root privileges are not required to create or run the portable application.

# Caveats
1. If the application is a single binary written in C/C++ that needs only glibc, libstdc++, and ancillary shared libraries, then *make-portable* takes care of everything automatically for you. If the application consists of multiple binaries and/or has data files, then manual tweaks will be needed.
2. Bloat. The portable application will be bigger and use more RAM than a traditional installation of the same application.
3. The portable application will contain only the shared libraries that the binary is linked to. If the application needs to load additional libraries while it's running, it may look for those libraries in the places where the libraries normally reside in the *source* OS. Sometimes there's an environmental variable you can use in the target OS that fixes the problem (for example, if app can't find graphics drivers, use `export LIBGL_DRIVERS_PATH=/path/to/dri/`). If there isn't an environmental variable you can use, you can always fall back on creating symlinks in the target OS (pointing from location of library in source OS to correct location of library in the target OS).
4. Needing a wrapper script on the target system is clunky. A more elegant (but root-requiring and labor-intensive) solution is to use my script to collect things on source system (binary, linker, libraries), put these things in a permanent place on target system, then use patchelf (https://nixos.org/patchelf.html) to patch the libraries and binary so that they can find one another in the target system. Here's an example of this approach:
  - Run `$ make-portable foo` on source system to collect what we need
  - Copy foo.tgz from your home folder on source system to target system's */tmp* directory
  - Run these commands on target system:
  ```
  $ cd /tmp
  $ tar -xvzf foo.tgz
  $ sudo mkdir -p /var/lib/foo
   1. move the transplanted ld, unaltered, to /var/lib/foo:
  $ sudo mv /tmp/lib/ld /var/lib/foo/
   2. patch the libraries so that they can find one another, then move them to /var/lib/foo:
  $ for library in /tmp/lib/*; do patchelf --set-rpath /var/lib/foo $library; done
  $ sudo mv /tmp/lib/* /var/lib/foo/
   3. patch the binary so it can find the transplanted ld and libraries, then move it to /usr/local/bin:
  $ patchelf --set-interpreter /var/lib/foo/ld /tmp/bin
  $ patchelf --set-rpath /var/lib/foo /tmp/bin
  $ sudo mv /tmp/bin /usr/local/bin/foo
```

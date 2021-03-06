# make-portable
Simple shell script that bundles everything needed for a GNU/Linux binary to run on a different system, even if target system's glibc/libstdc++ versions are older than those of source system

# How it works
The script copies the specified binary, all its dynamically linked libraries (including glibc and, if appropriate, libstdc++), and source system's linker (ld) into a tarball. A wrapper script is also created. Both the tarball and wrapper script are placed in user's home directory.

# Usage directions
1. Install the desired application on the source GNU/Linux system the usual way (from repository or source code)
2. Run this script on the source system, passing the name of the binary as the sole argument
3. Copy the resulting tarball and wrapper script to the target GNU/Linux system (you can copy them into any directory on target system, as long as tarball and wrapper script are both in the same directory)
4. Use the wrapper script as you'd use the application

# Usage example
The main reason I created this script is so that I can build the newest mpv media player on my personal laptop then run it on my digital media player, which runs an extremely ancient GNU/Linux OS. So I'll use mpv as an example:

1. Build newest mpv from source on my laptop, install it, make sure it works and is in my PATH
2. Run this command on my laptop: `$ make-portable mpv`
3. Copy the resulting tarball (*mpv.tgz*) and wrapper script (*mpv*) from laptop's home directory to any directory on media player (I chose */usr/local/bin/*)
4. In media player: `$ mpv /path/to/nicemovie.mkv`

# Dependencies
grep, awk, tar, and ldd. In other words, the dependencies are minimal and should be present on every GNU/Linux system. Also, root privileges are not required to create or run the portable application.

# Caveats
1. If the application is a single binary written in C/C++ that needs only glibc, libstdc++, and ancillary shared libraries, then *make-portable* takes care of everything automatically for you. If the application consists of multiple binaries and/or has data files, then manual tweaks will be needed.
2. Bloat. The portable application will be bigger and use more RAM than a traditional installation of the same application.
3. The portable application will contain only the shared libraries that the binary is linked to. If the application needs to load additional libraries while it's running, it may look for those libraries in the places where the libraries normally reside in the *source* OS. Sometimes there's an environmental variable you can use in the target OS that fixes the problem (for example, if app can't find graphics drivers, use `export LIBGL_DRIVERS_PATH=/path/to/dri/`). If there isn't an environmental variable you can use, you can always fall back on creating symlinks in the target OS (pointing from location of library in source OS to correct location of library in the target OS).

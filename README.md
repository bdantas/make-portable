# make-portable
Shell script that creates portable version of a GNU/Linux application that can be used on an older target OS.

# Why would I want this?
So that, when that dusty old machine you haven't upgraded in years needs the newest version of some application, you can avoid the pain of having to compile software on the old machine. If you want to be able to build an application on your daily-driver GNU/Linux OS then bundle everything that's needed for the application to run on an older OS, then this script may do the trick for you.

# How do I use it?
1. Install the desired application on the source GNU/Linux OS the normal way (from repository or source code).
2. Run this script on the source OS. The script grabs the application's binary and all its dependent shared libraries (including glibc and libstdc++ if applicable). It also grabs source OS's linker (ld). Then script creates a wrapper script. All of that is put into a tarball. Script then creates a trivial launcher script. When the script finishes, the tarball and launcher script will be in your home folder.
3. Copy the tarball and launcher script to the target GNU/Linux OS. You can put them in any directory, just make sure that both are in the same directory.
4. Use the launcher script exactly as you'd use the application.

# Can you give a usage example?
Sure. Main reason I created this script is so that I can build the newest mpv media player on my personal laptop, then run it on my digital media player, which uses an extremely ancient GNU/Linux OS. So I'll use mpv as an example.

1. Build newest mpv from source on my laptop and install it. I test it and see that it works properly.
2. Run this command on my laptop: $ make-portable mpv
3a. Copy "mpv-portable.tgz" and the "mpv" launcher script from my home directory onto a thumbdrive
3b. Copy "mpv-portable.tgz" and the "mpv" launcher script from thumbdrive to some directory in my media player
4. In media player: $ /path/to/mpv /path/to/nicemovie.mkv

# Hasn't this been done before?
Not exactly.

There's AppImage, which is a life-saver when I can find one for the application I need. However, AppImage's approach is different: They are not creating applications in new OS for old OS. Their approach is building applications in as-old-as-possible OS and combining that with an LD_LIBRARY_PATH hack. It is brilliant but the trouble they go through for their users (creating suitable build environment on an old machine) is the very trouble I'm trying to spare myself.

In Debian-like OSes it is possible to have multiple versions of gcc and g++ installed. Each version automatically installs its own corresponding libraries. It's nicely explained here: https://askubuntu.com/a/923342 Switching to older gcc/g++ version can produce a *binary* that works in older OS. However, the binary alone is useless without all the ancillary shared libraries it needs. So it is also necessary to compile all the shared libraries that are missing in the target system. It is a very labor-intensive approach but can work.

I've never had success with using older glibc on newer machine with newer machine's default gcc. It seems that gcc version is tightly coupled with glibc version. However, there is an extremely clever way around this: https://github.com/wheybags/glibc_version_header The limitation here is that a workaround for g++/libstdc++ is still in the works. And even when it is found, there is still the issue of missing ancillary shared libraries as I mentioned above.

# So how does it work?
An LD_LIBRARY_PATH hack works great when the portability issue is missing *ancillary* shared libraries. However, glibc cannot be overridden by LD_LIBRARY_PATH. Fortunately, if the linker (ld) is called directly using the `--library-path` flag, then ancillary shared libraries as well as glibc/libstdc++ can be overriden. This is the central idea of my script.

# What are the caveats?
1. If the application is a single binary written in C/C++ that needs only glibc, libstdc++, and ancillary shared libraries, then the script takes care of everything automatically for you. If the application consists of multiple binaries and/or has data files, then manual intervention will be needed.
2. The tarball may be huge.
3. Running the application on target OS will use a lot more RAM than it would if you were to do the "right thing" (see #4 below).
4. Using this script proves that your are lazy--that you desperately need an application but refuse to pull up your sleeves and do the right thing by compiling on the old target machine. I won't judge you.

# Your hack is very ugly.
I know. 
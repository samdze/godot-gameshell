# Godot for GameShell
Godot 3.2.3 export templates and instructions for the GameShell portable game console.


**Export templates download link:**

https://github.com/samdze/godot-gameshell/releases/download/3.2.3-stable/godot-3.2.3-gameshell-templates.zip

## How to make Godot games work

Coming soon...

## Compile Godot 3.2.3 manually

0. Make sure you have installed Clockwork OS v0.5 and have at least 5 or 6 GB of free space on the sd.

1. SSH into your GameShell, the launcher app **Tiny Cloud** shows the ip address and the default password you have to type in order to get access.

2. Update your GameShell and install Godot dependencies.
    ```
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get install build-essential scons pkg-config libx11-dev libxcursor-dev libxinerama-dev libgl1-mesa-dev libglu-dev libasound2-dev libpulse-dev libxi-dev libxrandr-dev libtheora-dev libvorbis-dev libpcre2-dev
    ```

3. Clone my Godot 3.2.3-gameshell repository branch and change your working directory.
It's a fork with a few code fixes. It's preferred to only get the latest commit not to occupy too much disk space.
    ```
    git clone --depth 1 -b 3.2.3-gameshell https://github.com/samdze/godot.git
    cd godot
    ```
    My fixes have been merged upstream starting from the Godot 3.2.4 release, so you'll hopefully be able to directly clone and build the official repository in the future.

4. Optional: create a swap file, it may be needed during the linking stage, especially if you're compiling a release build. This is temporary, it will be disabled after reboot.
    ```
    sudo fallocate -l 3G /swap
    sudo chmod 600 /swap
    sudo mkswap /swap
    sudo swapon /swap
    ```

5. Configure SCons to use Python 3.5, as it enables to build in a concurrent manner.
    Change the first line of the `/usr/bin/scons` file to:
    ```
    #! /usr/bin/python3.5
    ```
    You can do it typing:
    ```
    sudo nano /usr/bin/scons
    ```
    Make the change, then press Ctrl+S and finally Ctrl+X.

6. Compile Godot. This is a scons configuration that tries to exclude everything related to 3D as it was found to be non-functional during my tests.
    - Debug:
    ```
    scons platform=x11 -j3 tools=no target=debug bits=32 module_webm_enabled=no module_bullet_enabled=no module_csg_enabled=no module_camera_enabled=no module_arkit_enabled=no module_gridmap_enabled=no module_mobile_vr_enabled=no module_vhacd_enabled=no module_xatlas_enabled=no module_cvtt_enabled=no module_assimp_enabled=no disable_3d=yes
    ```
    - Release:
    ```
    scons platform=x11 -j3 tools=no target=release debug_symbols=no bits=32 use_lto=yes module_webm_enabled=no module_bullet_enabled=no module_csg_enabled=no module_camera_enabled=no module_arkit_enabled=no module_gridmap_enabled=no module_mobile_vr_enabled=no module_vhacd_enabled=no module_xatlas_enabled=no module_cvtt_enabled=no module_assimp_enabled=no disable_3d=yes
    ```
    You could get this error during linking:
      `collect2: fatal error: ld terminated with signal 9 [Killed]`
      Make sure to have enough free space on the sd and to create a large enough swap file to fix it.

    The compilation may take a long time, 2+ hours, maybe a bit less if you're compiling a debug build.
    
    The Godot binary files will be placed inside the bin folder, they will be named `godot.x11.debug.32` and `godot.x11.opt.32` for debug and release builds respectively.
    
    If you previously created a swap file, you can now reboot and then delete it:
    ```
    sudo reboot
    // wait for reboot and ssh into the GameShell
    sudo rm /swap
    ```

# Godot for GameShell
Godot 3.2.3 export templates and instructions for the GameShell portable game console.

## How to make Godot games work

...

## Compile Godot 3.2.3 manually

0. Make sure you have at least 5 or 6 GB of free space on the sd.

1. SSH into your GameShell, the launcher app **Tiny Cloud** shows the ip address and password you have to type in order to get access.

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

4. Optional: create a swap file, it may be needed during the linking stage. This is temporary, it will be disabled after reboot.
    ```
    sudo fallocate -l 3G /swap
    sudo chmod 600 /swap
    sudo mkswap /swap
    sudo swapon /swap
    ```

5. Compile Godot. This is a scons configuration that tries to exclude everything related to 3D as it was found to be non-functional during my tests.
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

    The compilation may take a long time, 2+ hours.
    
    The Godot binary files will be placed inside the bin folder, they will be named `godot.x11.debug.32` and `godot.x11.opt.32` for debug and release builds respectively.
    
    If you previously created a swap file, you can now reboot and then delete it:
    ```
    sudo reboot
    // wait for reboot and ssh into the GameShell
    sudo rm /swap
    ```

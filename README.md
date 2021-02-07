# Godot for GameShell
Godot 3.2.3 export templates and instructions for the GameShell portable game console.

<p align="center">
    <img src="/gameshell-godot.png" />
</p>



**Export templates download link:**

3.2.3: https://github.com/samdze/godot-gameshell/releases/download/3.2.3-stable/godot-3.2.3-gameshell-templates.zip

## How to make Godot games work

Your Godot game will probably not work out of the box on the GameShell, a few tweaks are usually needed.
Here are listed the details you should be aware of and the guidelines to follow to have your game up and running!

1. Create a new export preset, call it GameShell or however you like.
    Disable the `64 Bits` option and select the custom templates you can download from this repository. The zip contains two files, `godot.x11.debug.32` is the debug build and `godot.x11.opt.32` is te release one.
    If you don't plan to release the game on the GameShell exclusively, it is a good idea to create a custom feature for it.
    <p align="middle">
    <img src="https://i.imgur.com/cwTHUd6.png" />
    
    <img src="https://i.imgur.com/m60t30z.png" />
    </p>
2. Change your project settings. You can override values for the GameShell only if you select the label of a property and click on `Override For...` in the top right corner of the popup window, then select the feature you created earlier. A new property named `<property_name>.<feature_name>` will appear.

    - Project Settings > Display > Window: set the window width and heigth to 320x240. This is the GameShell screen resolution. Also set the always on top option.
        <p align="center">
        <img src="https://i.imgur.com/IZxbjWs.png" />
        </p>
        
        Set the aspect and the mode too. `keep_width` and `2d` usually work best, but it may depend on the game you're making.
        <p align="center">
        <img src="https://i.imgur.com/PNohbKb.png" />
        </p>
    - Project Settings > Rendering > Quality: the GameShell supports GLES2 only. Change the driver name.
        <p align="center">
        <img src="https://i.imgur.com/tYzi5qL.png" />
        </p>
        
        It's recommended to change the framebuffer allocation to `2D Without Sampling`, but you can try setting it to `2D` if you need.
        <p align="center">
        <img src="https://i.imgur.com/zI8uhK2.png" />
        </p>
3. One of the GameShell conventions is that when you press the `MENU` button, you exit the app or game you're currently in.
The `MENU` button is mapped to the escape key, so just create a new action `ui_escape` in the `Input Map` settings that activates with the escape key and add this snippet somewhere in your code, where it is always running:
    ```gdscript
    if OS.has_feature("GameShell") and Input.is_action_pressed("ui_escape"):
        get_tree().quit()
    ```
    Remove the feature check if you didn't create one.
4. The inputs need to be remapped. This is the GameShell keypad mapping:
    <p align="center">
    <img src="https://raw.githubusercontent.com/clockworkpi/Keypad/master/keymaps.png" />
    </p>
    
    One way you can do it is by adding a similar piece of code inside an initialization function:
    ```gdscript
    if OS.has_feature("GameShell"):
        InputMap.action_erase_events("ui_accept")   # Erase the previous mapping.
        var ev = InputEventKey.new()                # Create a new event that will activate the action.
        ev.pressed = true                           # Action activates on key pressed.
        ev.scancode = KEY_K                         # The key to press is K, which is the B button.
        InputMap.action_add_event("ui_accept", ev)  # Add the action back into the InputMap.
        ... # Repeat for each action you want to remap.
    ```

5. Add the game into your GameShell. Coming soon.

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

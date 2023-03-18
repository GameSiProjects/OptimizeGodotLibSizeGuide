
# Optimize Godot lib size for simple Android game

Godot is a great engine for creating game with an easy-to-use editor. It has many modules that support you building all kinds of game.
The problem is the default options of exporting game compile all of the features, and cause the build unnecessary large for simple games.
Godot's document also have [guides](https://docs.godotengine.org/en/latest/contributing/development/compiling/optimizing_for_size.html) to optimize builds for size, but I think it's not clearly enough and hasn't mention about using engine build configuration profile yet.
You will need to build a custom export template to reduce the engine's binary lib size. After here is a step-by-step instruction for Android platform, my release APK size for all 4 architectures is **40MB** after these steps:

1. Clone [Godot repository](https://github.com/godotengine/godot) then switch to your current using version if needed.
2. Install [Scons](https://scons.org/pages/download.html), the buildsystem that being used for Godot. ([Godot's official document](https://docs.godotengine.org/en/3.1/development/compiling/introduction_to_the_buildsystem.html))
3. Using Scons to build the custom export template ([Godot's official document](https://docs.godotengine.org/en/latest/contributing/development/compiling/compiling_for_android.html#doc-compiling-for-android)). The basic build command is:
 `scons platform=<platform> target=debug/release_debug/release`
 and we could add more params to this to reduce the output size. For a list of available params, run this command `scons`.
> Here is my final build command for a arm64 export template of an simple 2D Android game:
> `scons platform=android target=template_release arch=arm64v8      optimize=size disable_advanced_gui=yes disable_3d=yes tools=no minizip=no  deprecated=no module_bmp_enabled=no module_bullet_enabled=no module_dds_enabled=no module_enet_enabled=no module_basis_universal_enabled=no module_jsonrpc_enabled=no module_lightmapper_rd_enabled=no module_mbedtls_enabled=no module_multiplayer_enabled=no module_msdfgen_enabled=no module_navigation_enabled=no module_noise_enabled=no module_ogg_enabled=no module_raycast_enabled=no module_regex_enabled=no module_squish_enabled=no module_svg_enabled=no module_text_server_fb_enabled=yes module_text_server_adv_enabled=no module_tga_enabled=no module_theora_enabled=no module_upnp_enabled=no module_webrtc_enabled=no module_websocket_enabled=no build_profile=./custom.build`
> 
> You could find how the build script handle these params in `./SConstruct` file

   - `optimize=size` params will enable `-0s` GCC flag, which is optimize for binary size and try not to sacrifice too much speed.
- `disable_advanced_gui=yes` params remove modules for complex GUI controls such as Tree, ItemList, TextEdit or GraphEdit.
- `disable_3d=yes` params remove modules for building 3D games.
- `module_\<module name>_enabled=no` will remove module with `module name`. You could find the size of compiled modules by running build command once, then go to `./modules/` and you will see `.a` output file for each module.
- `build_profile=<Path to build configuration profile>` will use a build configuration profile that allow you to disable classes in Godot engine.
    -  To create one, open Godot Editor and go to **Project > Customize Engine Build Configuration…**. After uncheck the unwanted class, click **Save As** to save the file.
4. The build command must be called once for each architecture that include inside the build. You could find the out put `.so` file in `./platform/android/java/lib/libs`
5. Navigate to `platform/android/java` and call `gradlew  generateGodotTemplates` to get your custom export template.
6. Move the export template from `.\bin\` to Godot's template folder ([Official document](https://docs.godotengine.org/en/latest/contributing/development/compiling/compiling_for_android.html#using-the-export-templates)):
    - Windows: `%APPDATA%\Godot\export_templates\<version>\` 
    > I'm using Godot 4.0 in Windows so I copied to `C:\Users\<user name>\AppData\Roaming\Godot\export_templates\4.0.stable`
    - Linux: `$HOME/.local/share/godot/export_templates/<version>/`
    - MacOS: `$HOME/Library/Application Support/Godot/export_templates/<version>/`
 7. Now you could use the custom export template to build you Android build. I assume you already could [export a normal Android build](https://docs.godotengine.org/en/3.1/getting_started/step_by_step/exporting.html?highlight=export#android) before this.
     - You still could not export `aab` file using custom export template, you must copy the `aar` lib file from `./bin/` to `<Your project>/android/build/libs/<release/debug folder>`, then export using Gradle.

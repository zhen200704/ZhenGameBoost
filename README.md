# GameBoost T606

Shizuku-powered game booster for **TECNO Spark 10C** (Android 12 · Unisoc T606 · Mali-G57 MP1 · 720×1612 @ 90 Hz).
The whole UI is `app/src/main/assets/index.html`. The Kotlin/Java code only gives that page a safe Shizuku shell.

## 1. Get Shizuku running (one time per reboot)
1. Install **Shizuku** (Play Store or GitHub: RikkaApps/Shizuku).
2. Settings → About phone → tap *Build number* 7× → Developer options → turn on **Wireless debugging**.
3. Open Shizuku → *Start via Wireless debugging* → pair with the code shown in Developer options.
4. Shizuku must be started again after every reboot (no root on this phone).

## 2. Build the APK
**Android Studio (PC):** File → Open → this folder → let Gradle sync (it downloads the Gradle wrapper,
AGP 8.5 and Shizuku API 13.1.5) → Build → Build APK(s). Install `app-debug.apk`.

**On the phone:** open this folder in *AndroidIDE* and build the same way.

> Plain "HTML to APK" converters can't do this part. They wrap the page in a WebView with no Shizuku bridge,
> so GameBoost will open in **Preview mode**: nothing runs, but the Shell tab still exports a complete
> script you can paste into Shizuku's `rish`, aShell or LADB.

## 3. First run
Open GameBoost → tap **Grant** → allow in the Shizuku popup. The app reads your device, then saves a
**snapshot** of every setting it might change. *Tweaks → Revert all* restores that snapshot.

## Backup mode (when Shizuku is off)
Run this **once** from Shizuku's rish, LADB or a PC (or tap *Tweaks → Backup mode → Grant via Shizuku*):
```
pm grant com.zhen.gameboost android.permission.WRITE_SECURE_SETTINGS
```
It survives reboots. With it, GameBoost still does settings tweaks (refresh rate, animations, scanning,
freezer, ANGLE/driver), gentle RAM cleaning (`killBackgroundProcesses`) and launch + post-launch sweep.
Downscale, Game Mode, renderer, doze exemption, force-stop, screen size and the Shell tab need Shizuku:
Android only allows those for the shell user, no app permission unlocks them.

## What it actually runs in Shizuku mode (shell uid 2000)
| Feature | Command |
|---|---|
| Per-game downscale | `cmd game downscale 0.7 <pkg>` (Android 12 accepts 0.3–0.9 in 0.05 steps, or `disable`) |
| Game Mode performance | `device_config put game_overlay <pkg> mode=2,downscaleFactor=X:mode=3,...` then `cmd game mode performance <pkg>` |
| UI renderer | `setprop debug.hwui.renderer skiagl / skiavk` (resets on reboot; app re-applies on open) |
| Free RAM | for running **third-party** apps only: `am kill` (gentle) or `am force-stop` (hard), then `am kill-all` |
| Launch | `monkey -p <pkg> -c android.intent.category.LAUNCHER 1`, then a delayed sweep in the same shell job |
| Doze exempt | `dumpsys deviceidle whitelist +<pkg>` |
| ANGLE / updatable driver | `settings put global angle_gl_driver_selection_*` / `updatable_driver_production_opt_in_apps` (only if firmware ships them) |
| Display | `peak/min_refresh_rate`, animation scales, `wm size` + `wm density` with 12 s auto-revert |
| System | Wi-Fi/BLE scan off, cached app freezer, `cmd power set-fixed-performance-mode-enabled` (experimental) |

Never killed: GameBoost, Shizuku, your launcher, your keyboard, the game, and anything you star in the RAM tab.
System apps are never touched.

## Updating the UI
Edit `app/src/main/assets/index.html` and rebuild. Open it in a desktop browser to test in Preview mode.

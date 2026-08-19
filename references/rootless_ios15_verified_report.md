# Verified Rootless iOS 15+ Tweak Research

**Target:** iPhone 7, iOS 15.8.5 (19H394), arm64, Dopamine, Sileo, ElleKit.

## Executive finding

The strongest public lock-screen reference I found is **Mooner 1.0.1**. Its public repository explicitly advertises iOS 15/16 lock-screen support, and its package index identifies an `iphoneos-arm64` Debian package with a SpringBoard filter, PreferenceLoader entry, and rootless package metadata.[1] The downloaded binary contains the iOS 15-specific hook type `iOS15_LSTimeHook` and references `CSProminentDisplayView`, `SBFLockScreenDateView`, `CSProminentDisplayViewController`, and `CSCombinedListViewController`.

The important distinction is that Mooner’s package is **package-level compatible and publicly distributed**, but its visual behavior has not been physically verified on the user’s device by this sandbox. A reliable physical test is still required.

## Downloaded references

| Package | Source | Architecture | Key payload | Result |
|---|---|---:|---|---|
| Mooner 1.0.1 | NoW’s public repository | arm64 + arm64e universal binary | `Mooner.dylib`, SpringBoard filter, Preferences bundle | Best lock-screen baseline |
| ElleKit 1.2 | Official ElleKit repository | `iphoneos-arm64` | injector, loader, rootless paths, compatibility symlink | Official injection baseline |
| old-lockscreen 0.0.1 | NightwindDev GitHub release | arm64 | iOS 15 lock-screen source/reference package | Useful source reference |
| Atlas 1.0.4 | qnblackcat rootless-tweaks release | arm64 package, universal binary | Legacy-style DynamicLibraries payload | Rootless recompiled reference, not a clean ElleKit-native layout |
| BarMoji 2023.3 | qnblackcat rootless-tweaks release | arm64 package | Legacy-style DynamicLibraries payload | Rootless recompiled reference |
| PowerCuff 0.1 | qnblackcat rootless-tweaks release | arm64 package, universal binary | Legacy-style DynamicLibraries payload | Rootless recompiled reference |

The downloaded files are attached with this report. The qnblackcat release states that its packages were recompiled for Dopamine but warns that issues may remain; therefore those packages are useful for comparison, not as proof that every feature works.[2]

## Official ElleKit package findings

The official ElleKit 1.1.3 and 1.2 packages were downloaded and inspected. Their control metadata includes:

```text
Package: ellekit
Architecture: iphoneos-arm64
Provides: mobilesubstrate (= 99)
Replaces: mobilesubstrate
```

The official package installs the injector under `/var/jb/usr/lib/ellekit/` and creates this compatibility symlink:

```text
/var/jb/Library/MobileSubstrate/DynamicLibraries
    -> /var/jb/usr/lib/TweakInject
```

This corrects an earlier oversimplification: a package using the legacy-looking `Library/MobileSubstrate/DynamicLibraries` path can still work on a device where the official ElleKit symlink exists. A package that creates a real directory there instead of following the symlink can be wrong, but the path alone is not proof of failure.

The official ElleKit package is therefore the correct injection baseline. It must already be installed and active before any third-party tweak can load.

## Mooner package findings

Mooner’s package metadata is:

```text
Package: com.now.mooner
Version: 1.0.1
Architecture: iphoneos-arm64
Depends: dev.theos.orion (>= 1.0.0), firmware (>= 12.2), preferenceloader
```

Its filter is a binary plist equivalent to:

```plist
Filter = { Bundles = ("com.apple.springboard"); };
```

Its binary is a universal arm64/arm64e Mach-O library and links against both ElleKit-compatible substrate compatibility and Orion:

```text
@rpath/Mooner.dylib
@rpath/Orion.framework/Orion
```

The binary contains these relevant class and hook names:

```text
iOS15_LSTimeHook
CSProminentDisplayView
SBFLockScreenDateView
CSProminentDisplayViewController
CSCombinedListViewController
```

This is materially different from the failed FreshLock approach, which attempted to scan arbitrary SpringBoard view trees and perform network/weather work from a constructor. A more reliable lock-screen tweak should hook the iOS 15 lock-screen presentation classes directly and keep its first visual change local and deterministic.

## Why the previous FreshLock builds were inconclusive

The earlier builds mixed several independent failure possibilities: uncertain hook classes, a generic view-tree scan, a Preferences bundle that was initially missing its Info.plist resource, and no physical injection log from the device. The packages compiled correctly, but compilation and installation do not prove that ElleKit loaded the dylib or that a selected method executed.

The correct diagnostic order is:

| Test | Meaning |
|---|---|
| Official ElleKit package is installed | Injector baseline exists |
| A known package such as Mooner appears in Sileo | Repository/package metadata works |
| Mooner changes the lock screen after respring | SpringBoard injection works |
| A FreshLock constructor log appears | FreshLock dylib loads |
| A class-hook log appears | The selected iOS 15 class and selector execute |
| A visual change appears | The UI operation targets the correct presentation object |

If Mooner also has no effect, building another custom tweak is premature: the likely issue is disabled Dopamine tweak injection, missing ElleKit/Orion/PreferenceLoader dependencies, or an installation/respring problem. If Mooner works, its iOS 15 class-hook surface is the correct baseline for a new original tweak.

## Recommended controlled test

First remove all earlier MinimalLock and FreshLock packages. Ensure the official ElleKit package is installed from its own Sileo source. Install **Mooner 1.0.1** from the public NoW repository, respring SpringBoard, and check whether its lock-screen layout changes. Do not install multiple lock-screen tweaks simultaneously.

If Mooner works, use its class targets as the basis for a new small tweak. If Mooner does not work, capture the Sileo installation state and Dopamine’s Tweak Injection state before any new package is built.

## References

[1]: https://nowisdev.github.io/ "NoW’s public repository and Mooner package index"
[2]: https://github.com/qnblackcat/rootless-tweaks "qnblackcat rootless-tweaks release notes"
[3]: https://github.com/dhinakg/ellekit-official "Official ElleKit repository and package artifacts"
[4]: https://ellekit.space/dopamine/ "Official Dopamine rootless and device support information"
[5]: https://github.com/NightwindDev/old-lockscreen "Open-source old-lockscreen reference"

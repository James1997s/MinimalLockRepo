# MinimalLock Sileo Repository

This repository distributes **MinimalLock 0.1.0**, a rootless lock-screen accent tweak for Dopamine on an iPhone 7 running iOS 15.8.5. The package is arm64, targets SpringBoard, and has no legacy MobileSubstrate dependency; Dopamine provides injection through ElleKit.

## Add to Sileo

Add this public raw repository source directly in Sileo:

```text
https://raw.githubusercontent.com/James1997s/MinimalLockRepo/main/
```

This repository uses the public GitHub raw HTTPS endpoint, so no separate web server is required.

## Repository layout

The repository is a small APT-compatible source. `Packages` indexes the Debian package, `Release` describes the repository, and the `.deb` is stored under `debs/`. Sileo can read this standard package index over HTTPS through the public GitHub raw endpoint.

## Safety

MinimalLock changes only the existing lock-screen clock/date label color. It does not modify the passcode system, notifications, or authentication controls. Keep a recovery path available when testing SpringBoard tweaks.

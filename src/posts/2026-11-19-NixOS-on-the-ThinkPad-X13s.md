---
title: 'NixOS on the ThinkPad X13s'
layout: post
---

> I said "dance", not "fuck", don't get it confused.

So I picked up a Lenovo ThinkPad X13s (Gen 1) — an ARM64 machine running Qualcomm's Snapdragon 8cx Gen 3 (`SC8280XP`). Light, fanless, and with a keyboard that pleases the fingertips. My new box is named `arty`. How fast the Nix addiction spreads! I am running NixOS upon all my machines. Such are the actions of madmen.

First, do not try to run your root filesystem off a USB-C drive. You'll lose your mind. Between Qualcomm's DWC3 controller choking on UAS queues and the internal Foxconn 5G modem pulling a sudden power blip on the PMIC rail that resets the left Type-C port, `ext4` will abort its journal and leave you crying in the rain. Install straight to the internal NVMe (`/dev/nvme0n1`) and dual-boot if you must.

Second, mainline Linux needs the right Device Tree Blob (`sc8280xp-lenovo-thinkpad-x13s.dtb`), a pile of Lenovo-specific firmware blobs under `/lib/firmware/qcom/sc8280xp/LENOVO/21BX/` for the Adreno 660 GPU and DSPs, and [Johan Hovold's](https://github.com/jhovold/linux) power fixes so the battery doesn't drain 15W while pretending to sleep:

    boot.loader.systemd-boot.extraFiles = {
      "dtbs/x13s-${config.boot.kernelPackages.kernel.version}.dtb" =
        "${config.boot.kernelPackages.kernel}/dtbs/qcom/sc8280xp-lenovo-thinkpad-x13s.dtb";
    };

    boot.kernelParams = [
      "dtb=dtbs/x13s-${config.boot.kernelPackages.kernel.version}.dtb"
      "clk_ignore_unused"
      "pd_ignore_unused"
      "arm64.nopauth"
      "mem_sleep_default=s2idle"
    ];

    # Qualcomm fTPM isn't exposed to mainline; don't let systemd wait 90s for it
    systemd.tpm2.enable = false;
    boot.initrd.systemd.tpm2.enable = false;

With `s2idle` properly configured, standby draw drops to around `0.3W`–`0.5W`. Whatever you do, **do not** run `rtcwake` to test it. On the `SC8280XP`, the PMIC RTC alarm registers (`pm8xxx-rtc`) belong to non-HLOS TrustZone firmware. Touch `/dev/rtc0` and TrustZone traps immediately, crashing the SoC straight into the UEFI menu. Just close the lid like a normal person.

<hr>

One quirk that drove me nuts: unlike x86 ThinkPads that talk to the Embedded Controller over LPC (`thinkpad_acpi`), the X13s has its EC hanging off an I2C bus (`0x28`). If you want the classic red ThinkPad dot to breathe when the machine sleeps and go solid when it wakes, you have to poke register `0x80` yourself with `i2ctransfer`:

    powerManagement = {
      enable = true;
      powerDownCommands = ''
        BUS=$(basename $(ls -d /sys/devices/platform/soc@0/ac0000.geniqup/a80000.i2c/i2c-* 2>/dev/null | head -n 1) | sed "s/i2c-//")
        [ -n "$BUS" ] && ${pkgs.i2c-tools}/bin/i2ctransfer -y "$BUS" w5@0x28 0x03 0x80 0x00 0x01 0x55 || true
      '';
      resumeCommands = ''
        BUS=$(basename $(ls -d /sys/devices/platform/soc@0/ac0000.geniqup/a80000.i2c/i2c-* 2>/dev/null | head -n 1) | sed "s/i2c-//")
        [ -n "$BUS" ] && ${pkgs.i2c-tools}/bin/i2ctransfer -y "$BUS" w5@0x28 0x03 0x80 0x00 0x01 0xAA || true
      '';
    };

Spoof your factory Wi-Fi and Bluetooth MACs via `udev` and `btmgmt`, enable `services.fprintd.enable = true` for the Synaptics fingerprint reader, and toss `msm`, `panel-edp`, and `phy-qcom-qmp-pcie` into `boot.initrd.kernelModules`. Wayland, Sway, audio, and hardware acceleration all hum right along.

Nix is nix and so are you.

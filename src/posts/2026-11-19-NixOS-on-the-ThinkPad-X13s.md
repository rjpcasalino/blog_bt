---
title: 'Up and Running with NixOS on the ThinkPad X13s'
layout: post
---

The Lenovo ThinkPad X13s (`SC8280XP`) is ARM64, fanless, and has a keyboard that pleases the fingertips. I got it secondhand three years ago? I don't remember, but I was excited because it promised running local models would be easy and fun and all that crap. Removing Windows was easy enough, though getting it back is a pain. I could not figure out how to build a custom ISO from a Nix flake back then, so I went with Ubuntu. That was OK, except the battery drained like a tub when the lid was closed, and it just was not what I wanted. Getting NixOS on there was the dream, so I set it aside and moved on to other things. Now we're entering a new age of technological advance, and I can have a cloud model like Gemini 3.8 Flash do some of the heavy engineering. The computer is smarter than me and thinks faster. I spent an hour or two with it and now this ThinkPad runs NixOS, with its little red dot breathing. Assisted Intelligence! That's the new two words.

What follows is a write-up by the model based on notes taken while getting the ThinkPad up and running with NixOS. I hope you find this post useful.

<hr>

Later on I read back through some old threads and learned this laptop is covered in [`nixos-hardware`](https://github.com/NixOS/nixos-hardware), which you should pull into your flake to avoid the redundant work I had the AI do. Tracking [NixOS/nixos-hardware#1751](https://github.com/NixOS/nixos-hardware/pull/1751) is also wise if you plan to keep using this hardware. Funny to think not two years ago I was frustrated that I could not figure out how to copy another Nixer's custom flake using [jhovold's](https://github.com/jhovold/linux) branch. I felt like a fool, and I am, but at least now it works and I can deploy NixOS on this machine. The touchscreen is a nice feature, and I hope to find useful applications for it to take us out of the 1970s and 1980s and dream of life beyond the terminal, unless something goes very wrong.

<hr>

First, don't run your root filesystem off USB-C. Between Qualcomm's DWC3 controller choking on UAS queues and the Foxconn 5G modem pulling a power blip on the PMIC rail that resets the left Type-C port, `ext4` will abort its journal and leave you crying in the rain. Install straight to internal NVMe (`/dev/nvme0n1`).

Second, mainline Linux needs the right DTB (`sc8280xp-lenovo-thinkpad-x13s.dtb`), Lenovo's firmware blobs under `/lib/firmware/qcom/sc8280xp/LENOVO/21BX/` for the Adreno 660 GPU and DSPs, and [Johan Hovold's](https://github.com/jhovold/linux) power fixes<sup id="fnref:1"><a href="#sup1">[1]</a></sup> so the battery doesn't burn 15W pretending to sleep:

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

    # Qualcomm fTPM isn't exposed to mainline; don't wait 90s for it
    systemd.tpm2.enable = false;
    boot.initrd.systemd.tpm2.enable = false;

With `s2idle` working, standby drops to ~`0.4W`. Whatever you do, **do not** run `rtcwake`. On `SC8280XP`, the PMIC RTC alarm registers (`pm8xxx-rtc`) belong to TrustZone firmware. Touch `/dev/rtc0` and TrustZone traps immediately, crashing straight into the UEFI menu. Just close the lid.

<hr>

Unlike x86 ThinkPads that talk to the Embedded Controller over LPC (`thinkpad_acpi`), the X13s hangs its EC off an I2C bus (`0x28`). If you want the red ThinkPad dot to breathe during sleep and go solid on wake, poke register `0x80` with `i2ctransfer`:

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

Spoof your factory Wi-Fi and Bluetooth MACs via `udev` and `btmgmt`, set `services.fprintd.enable = true`, and toss `msm`, `panel-edp`, and `phy-qcom-qmp-pcie` into `boot.initrd.kernelModules`. Wayland, Sway, audio, and GPU acceleration all hum right along.

Nix is nix and so are you.

- - -

<div class="footnotes">
<li><sup id="sup1"><a href="#fnref:1">[1]</a></sup> <small>At the end of kernel boot, Linux disables unclaimed clocks and power domains, crashing the <code>SC8280XP</code> if firmware or late-loading initrd modules (like <code>msm</code> or QMP PHYs) haven't claimed them yet; Hovold wired up the proper DTB dependency graphs while <code>clk_ignore_unused</code> and <code>pd_ignore_unused</code> act as a safety net. For Modern Standby (<code>s2idle</code>), every active voter on Qualcomm's RPMH power domains and interconnects must release its vote during suspend or the SoC stays awake burning 10-15W. Hovold patched the interconnect, DWC3/QMP combo PHY, and PCIe drivers to release their power votes on suspend, and mapped the Power Domain Controller (PDC) wakeup interrupts so the lid and I2C keyboard can actually wake the machine from deep sleep (~0.4W).</small></li>
</div>

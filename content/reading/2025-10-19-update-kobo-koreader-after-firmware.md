---
categories:
- reading
date: "2026-10-19T00:00:00Z"
title: Updating KFMon after Kobo Firmware Update
---

After a firmware update on a kobo device, to launch KOReader, you will need to update KFMon. Here are the steps to do so:

Plugin your Kobo device via USB and mount the filesystem.

Download the latest OCP (One Click Install Package) for KFMon.

This will have a filename such as kfm_nix_install.zip.

Extract this file to another directory. 

Download the OCP file for KFMon - it will have a filename such as OCP-KFMon-1.4.6-151-gac587d3.zip. Only this file should be used after a firmware update. 

Then place the OCP-KFMon-<version>.zip file in the same directory that was unzipped from kfm_nix_install.zip. Then run ./install.sh in the command line terminal. This will ask which file you want to install. 

Once the file has been installed, eject the Kobo device and unplug it. Start it up and it will install KFMon. This means you can launch KOReader again.

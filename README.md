winpe_vnc - How to integrate a VNC server into WinPE
====================================================

Quick steps
-----------

I have pre-packaged the VNC server (TightVNC) files plus scripts needed to set up the server.

* Download for 32-bit boot image: [winpe_vnc_files_x86.zip](https://github.com/sjkingo/winpe_vnc/raw/master/winpe_vnc_files_x86.zip)
* Download for 64-bit boot image: [winpe_vnc_files_x64.zip](https://github.com/sjkingo/winpe_vnc/raw/master/winpe_vnc_files_x64.zip)

1. Download the correct zip file for the boot image architecture (above).
2. Unzip the files to a temporary directory.
3. Mount the WIM image with `dism`.
4. Move the `deploy` and `VNC` directories into the image root.
5. Delete `setup.exe` in the root of the image.
6. Move the `startnet.cmd` file to `\Windows\System32`, overriding the existing file.
7. Unmount the image and all finished.


Manual steps (to use latest version of TightVNC)
------------------------------------------------

1. Mount WIM image with dism (if extracted from boot.wim, it will be index:2)
2. mkdir \deploy
3. move \setup.exe -> \deploy\setup.exe
4. Download TightVNC 32 or 64-bit (important!!)
5. Install TightVNC on reference computer and configure the service mode version
6. Export HKLM\SOFTWARE\TightVNC to config32.reg
7. (64-bit only) Export HKLM\SOFTWARE\Wow6432Node\TightVNC to config64.reg
8. mkdir \VNC and place .reg file(s) in directory
9. Copy the following files from the TightVNC install folder to \VNC:

    * tvnserver.exe
    * screenhooks32.dll

     (64-bit only):
    * hookldr.exe
    * screenhooks64.dll

10. Edit \Windows\System32\startnet.cmd and append the following line:
    
    \deploy\preinit.cmd

11. Create \deploy\preinit.cmd as follows (delete the config64.reg line if using a 32-bit image):

    ```
    @echo off

    wpeutil InitializeNetwork
    wpeutil DisableFirewall

    cd \VNC
    regedit /s config32.reg
    regedit /s config64.reg
    tvnserver -install -silent
    tvnserver -start

    cd \deploy
    copy setup.exe ..\
    cd ..\
    start setup.exe
    ```

12. Commit and unmount the image
13. Upload to WDS and voila


Some helpful commands
---------------------

```
dism /Mount-Image /ImageFile:<SRC> /index:2 /MountDir:<MOUNT_POINT>
dism /Unmount-Image /MountDir:<MOUNT_POINT> /commit
dism /Export-Image /SourceImageFile:<SRC> /DestinationImageFile:<NEW_FILE> /Compress:max /SourceIndex:2
dism /Get-ImageInfo /ImageFile:<SRC>
```

Note the image index will always be :2. Don't attempt to mount/edit index:1 as
it does not contain the setup bootstrapper that is run on PXE boot.

Exporting the image will recompress it slightly and yield a smaller file.

Please note TightVNC is licensed under the GPL.

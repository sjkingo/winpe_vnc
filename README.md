winpe_vnc - How to integrate a VNC server into WinPE
====================================================

Quick steps
-----------

Manual steps
------------

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
13. Export index:2 from the image (this compresses it again):

    `dism /Export-Image /SourceImageFile:<SRC> /DestinationImageFile:<NEW_FILE> /Compress:max /SourceIndex:2`

14. Upload to WDS and voila


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

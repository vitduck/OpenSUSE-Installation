# OpenSUSE-Installation
1. Desktop experiences
   * `xrand`
     * Set resolution and refresh rate:
     ```
     $ xrandr -s 1920x1080 -r 144 
     ```
   * `reshift`
     * Set monitor temperature to reduce blue light:
     ```
     $ redshift -P -O 5000
     ```
   * Font display
     * Install fonts required for correct display of Asian characters
     ```
     $ sudo zypper install google-noto-sans-cjk-fonts fetchmsttfonts
     ```
2. i3 window manager
   * `rxvt`: prompt appears at middle of screen when opening a new terminal via `Mod+Enter`
     * This issue is due to a line-warp function recently introduced in rxvt-9.31
     ```
     $ zypper search -s  rxvt-unicode
     S  | Name         | Type       | Version          | Arch   | Repository
     ---+--------------+------------+------------------+--------+----------------------------------------
     i  | rxvt-unicode | package    | 9.31-bp155.3.6.1 | x86_64 | Update repository of openSUSE Backports
        | rxvt-unicode | package    | 9.31-bp155.3.3.1 | x86_64 | Update repository of openSUSE Backports
        | rxvt-unicode | package    | 9.30-bp155.2.6   | x86_64 | openSUSE-Leap-15.5-1
        | rxvt-unicode | package    | 9.30-bp155.2.6   | x86_64 | Main Repository
        | xvt-unicode  | srcpackage | 9.31-bp155.3.6.1 | noarch | Update repository of openSUSE Backports
        | rxvt-unicode | srcpackage | 9.31-bp155.3.3.1 | noarch | Update repository of openSUSE Backports
        | rxvt-unicode | srcpackage | 9.30-bp155.2.6   | noarch | Source Repository
     ```
     * Downgrade `rxvt` to 9.30 version
     ```
     $  sudo zypper install --oldpackage rxvt-unicode-9.30-bp155.2.6.x86_64
     ```
     * Lock `rxvt` from being updated in the future via `/etc/zypp/locks`
     ```
     type: package
     version: = 9.30-bp155.2.6
     match_type: exact
     case_sensitive: on
     solvable_name: rxvt-unicode
     ```
    * `~/.config/i3/config` 
      * Disable title bar
       ```
       default_border pixel 1
       default_floating_border pixel 1
       ```
      *  Set wallpaper
       ```
       exec --no-startup-id feh --bg-scale wallpaper.jpg
       ```  
    * `VirtualBox`: black screen when switching to fullscreen:
      * Disable minibar: https://github.com/i3/i3/issues/2324#issuecomment-216045371
3. Slow start-up of GTK-and-GNONE-based applications:
   * `firefox`: This issue is due to time-out during Firefox's access to xdg's lock files
     * Uninstall `xdg-desktop-portal` packages 
     ```
     $ sudo zypper remove xdg-desktop-portal
     ```
   * Lock `xdg-desktop-portal` from being installed in the future via `/etc/zypp/locks`
     ```
     type: package
     match_type: glob
     case_sensitive: on
     solvable_name: xdg-desktop-portal
     ```
4. Virtualbox with Windows Guest
   * Guest Additions ISO: https://download.virtualbox.org/virtualbox/7.0.14/VBoxGuestAdditions_7.0.14.iso
   * Blank screen during installation of Microsoft Office:
     * Before installtion: disable 3d hardware acceleration at VM's level.   
     * After installation: disable 3d hardware acceleration locally for Word/Exel/Powerpoint: Options -> Advances -> Display

5. Media codec installation 
   * Add packman repo
     ```
     $ sudo zypper addrepo -cfp 90 'https://ftp.gwdg.de/pub/linux/misc/packman/suse/openSUSE_Leap_15.5/' packman
     $ sudo zypper refresh
     $ sudo zypper dist-upgrade --from packman --allow-vendor-change
     ```
   * Install media player and codecs
     ```
     $ sudo zypper install --from packman ffmpeg MPlayer gstreamer-plugins-{good,bad,ugly,libav} libavcodec
     ```
6. ROCM installation
   * Register kernel-mode driver
     ```
     $ sudo tee /etc/zypp/repos.d/amdgpu.repo <<EOF
     [amdgpu]
     name=amdgpu
     baseurl=https://repo.radeon.com/amdgpu/6.0.2/sle/15.5/main/x86_64/
     enabled=1
     gpgcheck=1
     gpgk ey=https://repo.radeon.com/rocm/rocm.gpg.key
     EOF
     
     $ sudo zypper ref
     ```
   * Register ROCm packages
     ```
     $ sudo tee --append /etc/zypp/repos.d/rocm.repo <<EOF
     [ROCm-6.0.2]
     name=ROCm6.0.2
     baseurl=https://repo.radeon.com/rocm/zyp/6.0.2/main
     enabled=1
     gpgcheck=1
     gpgkey=https://repo.radeon.com/rocm/rocm.gpg.key
     EOF
     
     $ sudo zypper ref
     ```
   * Install kernel driver
     ```
     $ sudo zypper --gpg-auto-import-keys install amdgpu-dkms
     $ sudo reboot
     ```
   * Install ROCm packages
     ```
     sudo zypper --gpg-auto-import-keys install rocm
     ```
   * `rocm-smi`
     ```
     ======================================== ROCm System Management Interface ========================================
     ================================================== Concise Info ==================================================
     Device  [Model : Revision]    Temp    Power     Partitions      SCLK    MCLK    Fan    Perf  PwrCap  VRAM%  GPU%  
             Name (20 chars)       (Edge)  (Socket)  (Mem, Compute)                                                    
     ==================================================================================================================
     0       [AMD Radeon RX Vega   26.0°C  6.0W      N/A, N/A        852Mhz  167Mhz  9.41%  auto  200.0W    1%   0%    
             Vega 10 XL/XT [Radeo                                                                                      
     ==================================================================================================================
     ============================================== End of ROCm SMI Log ===============================================
     ```

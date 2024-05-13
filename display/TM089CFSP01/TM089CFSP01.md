# TM089CFSP01

### [Datasheet](TM089CFSP01_Pre_Ver1.2.pdf)

| Name | Value |
|------|-------|
| Size | 8.9" |
| Resolution | 3840x2400 |
| Technology Type | a-Si TFT |
| Pixel Configuration | Mono-stripe |
| Pixel Pitch(mm) | 0.05x0.05 |
| Display Mode | Display Mode Transmissive, Normally Black |
| Surface Treatment | Clear |
| Viewing Direction | All |
| Gray Scale Inversion Direction | NA |
| LCM (W x H x D) (mm) | 202x133x1.24 |
| Active Area(mm) | 192x120 |
| With /Without TSP | Without TSP |
| Matching Connection Type | BM20B(0.8)-50DP-0.4V(51) |
| LED Numbers | NA |
| Weight (g) | TBD |
| Interface | 2 port MIPI |
| Color Depth | Mono-LCD |
| Driver IC | FT7250*2 |


# config.txt

1. Open your `config.txt`.
   - &gt;= Bookworm: `sudo nano /boot/firmware/config.txt`
   - &lt;= Legacy: `sudo nano /boot/config.txt`
2. Locate the line `dtoverlay=vc4-kms-v3d` and replace by: `#dtoverlay=vc4-kms-v3d`
3. At the end of the file paste:
```ini
[all]
disable_splash:0=1
disable_overscan:0=1
hdmi_force_hotplug:0=1
framebuffer_depth:0=24
framebuffer_ignore_alpha:0=1
hdmi_pixel_encoding:0=2
framebuffer_width:0=1280
framebuffer_height:0=2400
max_framebuffer_width:0=1280
max_framebuffer_height:0=2400
hdmi_pixel_freq_limit:0=400000000
hdmi_timings:0=1280 0 70 32 46 2400 0 62 20 28 0 0 0 50 0 179210000 5
#config_hdmi_boost:0=4
hdmi_drive:0=2
hdmi_group:0=2
hdmi_mode:0=87
hdmi_ignore_edid:0=0xa5000080
```
4. Replace the `:0` by your HDMI port number where display is connected, if connected to HDMI**1** change to `:1`.
4. Save and exit
5. Reboot: `sudo reboot`


# printer.cfg

```ini
[msla_display]
model: TM089CFSP01
pixel_format: Mono
framebuffer_index: 0
resolution_x: 3840
resolution_y: 2400
pixel_width: 0.05
pixel_height: 0.05
cache: 1
uvled_output_pin_name: msla_uvled
```

1. Change the `framebuffer_index` if you have to, if only one HDMI is connected the index is always `0`
   - To see how many framebuffer devices are connected, you can use the command: `ls /dev/fb*`
1. Reboot klipper
2. Test
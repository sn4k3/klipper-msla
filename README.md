# Klipper mSLA
Instructions to have Klipper running with mSLA printers.

# 1. Hardware

## Requirements

1. Electronics board (MCU).
2. Raspberry Pi 4 or greater.
3. Your printer must have an LCD with HDMI connection. As alternative is possible to buy an HDMI converter board.  

Hardware example: BTT Manta M5P + CM4.

## Setup

1. Make the apropriate connections with your printer hardware to the electronic board.
   - Connect motor(s)
   - Connect endstop(s)
   - Connect fan(s)
   - Connect heater(s) and thermistor(s)
   - Connect interface lcd
   - And other hardware you might have
2. Make sure to plug the UVLED to the most capable mosfet, this is often the heatbed port (HB).   
   - Note that for UVLED the **polarity matters**! Do not connect at random. Follow datasheet to know which is the positive and negative pin.
   - If your UVLED have an thermistor, connect to TB.
3. Connect your main display to an HDMI port

# 2. Software and firmware

1. Install Klipper as you normaly do, either manually or via MainsailOS to the sdcard. I recommend the use of [rpi imager](https://www.raspberrypi.com/software).
2. Update your system: `sudo apt update && sudo apt full-upgrade -y`
3. Flash MCU with klipper as you normally do, eg: via [kiuah](https://github.com/dw-0/kiauh).
4. After instalation you need to replace the `~/klipper/klippy/` folder by my [klipper-msla](https://github.com/sn4k3/klipper/tree/msla/klippy) fork.
   - Only need to replace `~/klipper/klippy` folder but does not hurt if you replace the whole `~/klipper` folder
5. (Optional) Configure SAMBA for easy access and modify the system contents.
5. Reboot
6. Access Mainsail portal and confirm if is working.

# 3. Configuration

1. Get and install the [klipper-msla-macros](https://github.com/sn4k3/klipper-msla-macros).
2. Configure all pins and modules acordingly your board and harware you using.
   - Klipper already have a repository with configuration sample for many boards, you can start from there.

## mSLA specific part:

```ini
[output_pin msla_uvled]
pin: PA5
pwm: True
cycle_time: 0.01 # 100Hz

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

[printer]
manufacturing_process: mSLA
kinematics: zaxis
max_velocity: 10
max_accel: 25
max_z_velocity: 10
max_z_accel: 25
```

## Full configuration sample:

```ini
[mcu]
serial: /dev/serial/by-id/usb-Klipper_stm32g0b1xx_390037001950425938323120-if00

[virtual_sdcard]
path: ~/printer_data/gcodes
on_error_gcode: CANCEL_PRINT

[pause_resume]

[display_status]

[respond]

# The sections below here are required for the macros to work. If your config
# already has some of these sections you should merge the duplicates into one
# (or if they are identical just remove one of them).
[idle_timeout]
gcode:
  _KM_IDLE_TIMEOUT # This line must be in your idle_timeout section.

[save_variables]
filename: ~/printer_data/variables.cfg # UPDATE THIS FOR YOUR PATH!!!

[stepper_z]
step_pin: PC6
dir_pin: PC7
enable_pin: !PA9
microsteps: 16
rotation_distance: 4
endstop_pin: ^PC3
position_endstop: 0
position_max: 225

[tmc2209 stepper_z]
uart_pin: PB10
run_current: 0.800
diag_pin: PC3

[output_pin msla_uvled]
pin: PA5
pwm: True
cycle_time: 0.01 # 100Hz

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

[temperature_sensor raspberry_pi]
sensor_type: temperature_host
#min_temp: 0
max_temp: 90

[thermistor NTC10K]
temperature1: 0.0
resistance1: 32116.0
temperature2: 40.0
resistance2: 5309.0
temperature3: 80.0
resistance3: 1228.0

[temperature_sensor UVLED]
sensor_type: NTC10K
sensor_pin: PA0
#min_temp: 0
max_temp: 96

[fan]
pin: !PA3
off_below: 0.25
kick_start_time: 2
cycle_time: 0.0001 # 10KHz

[pwm_cycle_time beeper] 
pin: EXP1_1

[display] 
lcd_type: uc1701 
cs_pin: EXP1_3 
a0_pin: EXP1_4 
rst_pin: EXP1_5 
contrast: 63 
encoder_pins: ^EXP2_5, ^EXP2_3 
click_pin: ^!EXP1_2 
## Some micro-controller boards may require an spi bus to be specified: 
#spi_bus: spi 
## Alternatively, some micro-controller boards may work with software spi: 
#spi_software_miso_pin: EXP2_1 
#spi_software_mosi_pin: EXP2_6 
#spi_software_sclk_pin: EXP2_2

[neopixel btt_mini12864] 
pin: EXP1_6 
chain_count: 3 
color_order: RGB 
initial_RED: 0.4 
initial_GREEN: 0.4 
initial_BLUE: 0.4

[printer]
manufacturing_process: mSLA
kinematics: zaxis
max_velocity: 10
max_accel: 25
max_z_velocity: 10
max_z_accel: 25

[board_pins]
aliases:
    # EXP1 header
    EXP1_1=PD5, EXP1_3=PB3, EXP1_5=PB5, EXP1_7=PB7, EXP1_9=<GND>,
    EXP1_2=PD4,  EXP1_4=PD6, EXP1_6=PB4, EXP1_8=PB6, EXP1_10=<5V>,
    # EXP2 header
    EXP2_1=PB14, EXP2_3=PB8, EXP2_5=PC10, EXP2_7=PC12,  EXP2_9=<GND>,
    EXP2_2=PB13, EXP2_4=PB9, EXP2_6=PB15, EXP2_8=<RST>, EXP2_10=<NC>

[gcode_macro _km_options]
variable_menu_show_octoprint: False
# Z position to park toolhead (set "max" to infer from stepper config).
variable_park_z: 50.0
variable_m600_park_z: "max"
variable_print_end_park_z: "max"
variable_print_start_fan_power: 180
gcode:
  SET_VELOCITY_LIMIT ACCEL=5

[include klipper-msla-macros/*.cfg]
```

Now that you have configuration set, you must configure your display correctly in both `printer.cfg` and system `config.txt`. Until then the checks will prevent klipper from boot.  
To find the correct settings for your display, check the display datasheet and set accordingly, or find a configuration sample from this repo.

## Database

| Display model | Size | Resolution | Used by known printers |
|---------------|------|------------|------------------------|
| [TM089CFSP01](display/TM089CFSP01/TM089CFSP01.md) | 8.9" | 3840x2400 | Uniz IBEE |

## Test

1. After all configurations and mainsail are running without error, do some tests without the VAT to ensure correct function, by running the following commands and observe the printer, must be visible the UVLED light and screen fill with shades of grey.

```
MSLA_UVLED_RESPONSE_TIME
MSLA_DISPLAY_RESPONSE_TIME
MSLA_DISPLAY_TEST
```

2. Calibrate the printer head home (Z0), follow your printer manual or whatever technique you use. Myself I like to place the VAT in and calibrate over it:  
   - Un-tight the printer head screws.
   - Home printer (G28 Z).
   - Press the plate against the VAT.
   - Tight screw the printer head.

Note that if you calibrate the home at aboslute 0mm from screen, the trigger delay of the endstop will make it to crash agasint the screen. If the plate have no spike object, such crash will not damage the screen, however I recommend to use adequate(no under-current nor over-current) current for the motor to stall with minor force. If you using TMC driver you can adjust the current before home and restore it after, so it will stall as soon it touch LCD and motor will not have the energy to try harder. Example bellow for TMC2209:

```ini
# This sample is only if you using tmc2209!
# Safer home preventing hard crash against display
[gcode_macro G28]
rename_existing: G28.999
gcode:
    ### SET THIS DEFAULT CARFULLY - start really low
    {% set my_current = params.CURRENT|default(0.300)|float %} ; adjust crash current
    ###
    {% set oldcurrent = printer.configfile.settings["tmc2209 stepper_z"].run_current %}
    {% set oldhold = printer.configfile.settings["tmc2209 stepper_z"].hold_current %} 
    M117 Homing...
     SET_TMC_CURRENT STEPPER=stepper_z CURRENT={my_current} ; drop current on Z stepper
    
    {% if printer.configfile.settings["stepper_z1"] %} ; test for dual Z
        SET_TMC_CURRENT STEPPER=stepper_z1 CURRENT={my_current} ; drop current
    {% endif %}

    G4 P200 ; Probably not necessary, it is here just for sure
    G28.999 Z0
    G4 P200 ; same as the first one
    
    SET_TMC_CURRENT STEPPER=stepper_z CURRENT={oldcurrent} HOLDCURRENT={oldhold}

    {% if printer.configfile.settings["stepper_z1"] %} ; test for dual Z
        SET_TMC_CURRENT STEPPER=stepper_z1 CURRENT={oldcurrent} HOLDCURRENT={oldhold} ; reset current
    {% endif %}
    M117
```

3. Slice and run a print without the VAT to ensure everything is correct and working as it should.
4. Double check if your fans and heaters are working properly.
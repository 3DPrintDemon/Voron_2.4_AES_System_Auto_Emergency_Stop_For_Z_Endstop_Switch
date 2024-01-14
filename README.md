# Voron 2.4 AES System - Auto Emergency Stop For Klicky Probe
Save your printer with this Auto E STOP System to catch Z Homing errors using a nozzle triggered endstop switch.

[<img width="171" alt="kofi_s_tag_dark" src="https://github.com/3DPrintDemon/Voron_2.4_AES_System_Auto_Emergency_Stop_For_Klicky_Probe/assets/122202359/8f542c7a-b302-49c6-94fe-76749f321f65">](https://ko-fi.com/3dprintdemon)
## Don't forget if you like & use this project you can buy me a beer/coffee to say thanks. https://ko-fi.com/3dprintdemon

You're probably thinking you don't need this & that your printer always works & it'll be fine. 

Yeah thats true until it isn't! All it takes is a single homing error or failure to totally ruin your print surface when your nozzle ploughs into it & then gets dragged round it when you start a print!

A friend of mine just had his $70 350x350 bed totally trashed by this exact thing, for some reason his nozzle hit the endstop off-centre & the Z offset was about 3mm wrong, as you can imagine there were some bad noises & lots of broken things!

So I came up with this idea! Basically consider it an insurance policy for your printer. Think of it like a smoke detector for your house, or seat belt in your car - it’s better to have them & not need them than it is to need them & not have them, & that one day you do need them they could well save everyhting!

# REQUIRED PARTS

- LDO M8 Pinda 2 Inductive Probe (NC version used here)
- x1 M4 8mm Cap head bolt
- x1 M4 T-nut
- THREADLOCK!!


# STL FILE...

Head over to printables to grab the STL: https://www.printables.com/model/722483-voron-24-aes-system-sensor-mount

# INSTALL

- Install the AES bracket onto the rear left corner of your printer's 20x20 extrusion, with about 10-15mm clearance from your B motor when the nozzle is on the bed. USE THREADLOCK!
- Now add the Probe to your bracket, set the tip of the probe so its 1mm from the housing of the B motor. USE THREADLOCK!
- Connect the wires to your board & define the correct pin.
- My probe's wiring was a follows. Yours may differ. Brown=+, Blue=-, Black=Signal, White=Temp.
- We dont need temp & its not good for accurate readings anyway so cut that back & insulate.


# CONFIGURING THE SYSTEM

This will require you to edit the `Klicky-Macros.cfg` file. Dont worry it's super simple edits. This file is located in your printer's `config` folder inside the `Klicky` folder.
open the file & CTRL+F & search:
```
[gcode_macro _Home_Z_]
```
Now on `Line Number 836` add this code:
```
SET_GCODE_VARIABLE MACRO=_AES_SYS VARIABLE=e_stop_armed VALUE=True # AES SYSTEM
_AES_READ # AES SYSTEM
```
Then on `Line Number 863` add this code:
```
SET_GCODE_VARIABLE MACRO=_AES_SYS VARIABLE=e_stop_armed VALUE=False # AES SYSTEM
_AES_READ # AES SYSTEM
```

Then DOUBLE CHECK YOUR WORK! 

All good? Ok `Save & Exit`

If you have a `Macros.cfg` add the following macros there. If you dont have a `Macros.cfg` I recomend you make one.
```
[gcode_macro _AES_SYS]
variable_e_stop_armed: False
gcode:

[gcode_macro _AES_READ]
gcode:
 {% set aes_vars = printer["gcode_macro _AES_SYS"] %}
 {% if aes_vars.e_stop_armed == True%}
   RESPOND TYPE=COMMAND MSG="Homing Z AES System ARMED"
 {% else %}
   RESPOND TYPE=COMMAND MSG="AES System DISAMRED"
 {% endif %}
```

Now head over to your `Printer.cfg` file & add:
```
########################################
#    AES SYSTEM CONTROL
########################################
# Get Button Status with: QUERY_BUTTON button=AES_System_Sensor
# Will retrun PRESSED or RELEASED

[gcode_button AES_System_Sensor]
pin: ^### <<<< ADD YOUR OWN BOARD PIN HERE!!
press_gcode:
 {% set aes_vars = printer["gcode_macro _AES_SYS"] %}
   # RESPOND TYPE=COMMAND MSG="AES System TRIGGERED"
 {% if aes_vars.e_stop_armed == True %}
   {action_emergency_stop("AES System TRIGGERED!")}
 {% endif %}
release_gcode:
   # RESPOND TYPE=COMMAND MSG="AES System READY"
```
You will obviously need to add your own pin you're using here & possibly a `!` between the `^` & your pin number, (so e.g `^!###`) if you have a `NO Probe` (Normally Open Probe) & not a `NC Probe` (Normally Closed Probe). 

NOTE: If you have an `NO Probe` you may also need to swap the lines under `press_gcode:` & `release_gcode:` for correct operation! IMPORTANT!!

# SETTING UP THE PRINTER

After adding the required lines to your .cfg files you'll need to spend some time setting the height of the probe so it activates between the Z endstop activation point & the nozzle touching the bed surface (Z0).
This can be a bit tricky & can take a bit of time.

You'll need to home the printer then move the nozzle down to Z5.0 then slowly lower it until you find the probe's triggering point & check to see if its between the two points mentioned above, if not move the probe until you get the correct position. THIS IS VITALLY IMPORTANT TO GET CORRECT!

To check the probe activation point you can use:
```
QUERY_BUTTON button=AES_System_Sensor
```

Once it changed from `RELEASED` to `PUSHED` you're there!

Or.....

If you want an automatic notification uncomment the two `RESPOND` lines in the `[gcode_button AES_System_Sensor]` / `press_gcode` & `release_gcode`

Just remember to comment them out again when you're finished as they're just for testing & setup.

Once complete you can real-world test the system in two ways. The first is to raise up Z to like `Z150` for example & hit `Home_Z`, once the printer is coming back down touch a metal object to your new sensor. This should activate the AES System & stop your printer.

The second way is to `Home_All` & then move the nozzle to `Z5.0`. Now send:
```
_AES_READ
```
It should come back saying `AES System DISARMED`. Then send:
```
SET_GCODE_VARIABLE MACRO=_AES_SYS VARIABLE=e_stop_armed VALUE=True
```
Then `_AES_READ` once again to confirm the state change.

Now slowly lower the nozzle to the bed, the AES system should trigger the `Emergency_Stop` once you reach your set point.

If it does not repeat with the nozzle off the edge of the bed to find the activation point (BE CAREFUL!!), or touch the probe tip with the metal object again if the activation point cannot be found.
If activation point is too high or low - move probe height to correct.
If probe doesn’t activate with the motor casing but does with a metal object move probe closer to the motor casing.

[<img width="171" alt="kofi_s_tag_dark" src="https://github.com/3DPrintDemon/Voron_2.4_AES_System_Auto_Emergency_Stop_For_Klicky_Probe/assets/122202359/8f542c7a-b302-49c6-94fe-76749f321f65">](https://ko-fi.com/3dprintdemon)
## Don't forget if you like & use this project you can buy me a beer/coffee to say thanks. https://ko-fi.com/3dprintdemon


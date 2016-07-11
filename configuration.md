# Axes and geometry

The LulzBot TAZ-5 firmware `Configuration.h` file sets X and Y axes
to 100.5 steps/mm, Z to 1600 steps/mm and E0 at 850 steps/mm.  These
assume 1/16 microstepping, so set `JP201` through `JP206` shunts
accordingly.  Look up the settings for your model of driver, since
they are not all the same.  For example, for 1/16 microstepping,
shunts 1, 2 and 3 should be off, off and on, respectively, for DRV8825
drivers, and all on for A4988 drivers.

# Trimpot adjustment

http://reprap.org/wiki/A4988_vs_DRV8825_Chinese_Stepper_Driver_Boards#Trimpot_adjustment

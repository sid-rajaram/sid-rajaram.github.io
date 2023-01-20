---
layout: post
title:  "Getting the ToF sensor to work on the Duckiebot DB21J with the 4GB Jetson"
date:   2023-01-19
usemathjax: true
---

I set up the Duckiebot DB21J with the 4GB Jetson and the dashboard kept showing the ToF distance sensor was not detected. I posted about it on the internal teams Stack Overflow and was pointed to the workaround by the Duckitbot staff. I'm summarizing the steps here without the detailed explanation in the response.

First, the I2C bus configuration differs between the 2GB and 4GB versions of the Jetson, so we need to switch CHL6 to CHL4 - you might need to unscrew the front bumper from the bot (the three metal screws) to be able to access CHL4 cleanly.

| ![Original build from the instructions](/assets/db21j_tof_fix/chl6.jpg) | ![Plug into CHL4](/assets/db21j_tof_fix/new_connection.jpg) |
|:--:|
| Original build from the instructions | Plug into CHL4 |

Start up your duckiebot and ssh into it with `ssh <DUCKIEBOTNAME>`. Then open the file `/opt/nvidia/jetson-io/Jetson/board.py` to edit (using vim or nano) and search for the line

    dtbdir = os.path.join(self.bootdir, 'dtb')

, and replace it with this

    # dtbdir = os.path.join(self.bootdir, 'dtb')
    dtbdir = os.path.join(self.bootdir, '')

Save the file and run the following two commands in the shell.

    dtc -I dts -O dtb -@ -o /boot/duckietown_front_bumper_2_v1.dtbo /boot/duckietown_front_bumper_2_v1.dts

    /opt/nvidia/jetson-io/config-by-hardware.py -n "Duckietown Front Bumper 2 V1"

The second command should produce a one-line output similar to the one below.

    Configuration saved to /boot/tegra210-p3448-0000-p3449-0000-a02-duckietown-front-bumper-2-v1.dtb.

For an additional test you can run the following command `i2cdetect -y -r 13` and you should see an output similar to the one below

        0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
    00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
    10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
    20: -- -- -- -- -- -- -- -- -- 29 -- -- -- -- -- -- 
    30: -- -- -- -- -- -- -- -- -- -- -- -- 3c -- -- -- 
    40: 40 -- 42 -- -- -- -- -- -- -- -- -- -- -- -- -- 
    50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
    60: 60 -- -- -- -- -- -- -- 68 -- -- -- -- -- -- -- 
    70: UU -- -- -- -- -- -- --                         

You are looking for a 29, which corresponds to the ToF sensor. You can also go to the dashboard under `Robot > Components` and see a Healthy status bar.

| ![Healthy status bar](/assets/db21j_tof_fix/healthy_status_bar.png) |

Finally, screw the front bumper back on.

| ![Final build](/assets/db21j_tof_fix/final_build.jpg) |
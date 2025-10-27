---
title: Assembly
author: Lucas Costello
last_updated: 2025-10-27
---

This section will provide a step-by-step guide to lead you through the assembly of the server. Everything you need besides a small wrench and screwdriver is included with the listed components. I will give a reminder to be careful with the FPC Cable that connects the ROCK 5C to the Penta SATA HAT. It's pretty delicate.  

Other than that, use the images provided to assemble the components.

**Note**: *For the purpose of these instructions, consider the front of the ROCK 5C to be the shorter side with the USB/Ethernet ports.* 

## Step 1: Cooling the ROCK 5C 
- Apply the silicone sheet and attach the heatsink/fan unit. 
- Refer to the image for a visual on how to place the unit, route the wires, and which connector to use. 

![[bin/images/assembly_heatsink.png]]


(Source: [Arace](https://arace.tech/products/radxa-heatsink-6540b-for-rock-5c?_pos=1&_psq=rock+5c+fan&_ss=e&_v=1.0))
## Step 2: Attaching the Penta SATA HAT
- On the ROCK 5C, use the copper posts provided with the SATA HAT to make pillars using the remaining 4 mounting holes. 
- After that's done, plug in the FPC cable so that the ROCK and SATA HAT are connected. Once connected, lay the SATA HAT over the pillars and ensure everything lines up. 
- Now, open up the SATA HAT Top Board and grab the copper pillars from that accessory. Screw together 4x 'risers' that are each 3 pillars tall. Then, use them to build up the initial set of pillars and fasten down the SATA HAT. 

[[/bin/images/assembly_pentsatahat.png|alt=Sata-Hat-Assembly]]


(Source: [Radxa](https://radxa.com/products/accessories/penta-sata-hat/#overview))
## Step 3: Installing Hard Drive Array
- Before installing the SATA HAT Top Board, we have to assemble our array of hard drives. I am using a 4-drive array, taking advantage of all of the SATA ports. 
- Determine the correct orientation of your SSDs. Then, use the included screws to fasten the complete rectangular acrylic board to the side of the array that's facing towards the back of the ROCK 5C. 
- After the back is situated, use the screws to fasten the cutout acrylic board to the other side of the array. *The notch faces up.*
- Line up the array of SSDs and connect them to the Penta SATA HAT. 
*(For a helpful visual showing the correct orientation of the drive array, refer to the image attached to the next step.)*

## Step 4: The SATA HAT Top Board
- Align the pillars with the mounting holes on the Top Board, making sure that the button is facing the front of the ROCK 5C. 
- Place and secure the Top Board. 
- Using the provided cable, connect the 10-pin connector on the Top Board to the 10-pin connector n the SATA HAT. On the SATA HAT, the connector is in between the Molex and eSATA interfaces. 
- Below is a picture showcasing what the server looks like when it's completely assembled. 

![bin/images/assembly_complete.png)


(Source: [Amazon](https://a.co/d/fXJQHji))



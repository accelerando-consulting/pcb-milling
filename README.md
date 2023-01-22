# How we make fast prototypes at Accelerando

This is a work in progress of what will become part of the accelerando resource website.  Uploaded to github because its taking forever and the WIP is useful.


## Double sided pcbs with kicad, pcb2gcode and bcnc.

You can use any cad software - I happen to use  KiCad currently.

### Guidlines for kicad.

Set your design rules to 0.4mm min track, 0.25mmspacing.

If you are going to use via rivets, make your vias 2.4mm with 0.9mm drill.   If you will use soldered wire
for vias, normal sized vias are OK.

Use the biggest tracks you can fit.   1mm or even 2mm for power, 0.5 or 0.4 for signals.  Consider using 0.6 or 0.8 for signals whenever you can.

Try to keep tracks vert/horiz where possible (I do front=vert back=horz), as this saves milling time versus jinking the milling head around following complex tracks.

### Guidelines for double sided layout

You will need two holes that are centred on the board and equidistant from the edges, in order to locate the flipped board accurately.

I use the mounting holes for this purpose.

Place four 3.175mm holes equidistant from each corner with a 7mm courtyard around the hole.   You can get away with two if you can’t afford the space.  We are going to use 3.175mm drill bit shanks (you will accumulate a near-infinite collection of broken ones, trust me!) as locator pins.

### Converting Gerber files to G-Code

Plot your board in kicad  with file-> plot.  Make sure the "use X2 format" checkbox is OFF.  Generate drill file; check the option for plated and npth in same file, even though we want one file for our component holes and a second file for our locating pins (the non-plated through holes).   We’ll fix this later.

[PCB2gode](https://github.com/pcb2gcode/pcb2gcode) has a ton of command line options.    Putting all the options in a configuration file makes things
much easier.  The configration file must be named "millproject".

See my sample millproject file, in this repository which sets which side each operation is done from.

For a single sided board, do everything from the front.   For a double sided board, we drill and mill from the front, then flip
the board over and do the back milling and the outline from the back side.

To start the process, Run pcb2gode, and note the printed board width in the output.

We are now going to re-run pcb2gcode, giving half this value for the mirror-axis option to pcb2gcode.  eg  if pcb2gcode says “Exporting back... DONE. (Height: 53.78mm Width: 96.25mm)” this would mean you should run pcb2gcode again giving the option “-—mirror-axis=48.125”.

You can add the option to your millproject file if you like (remember to update it if you change the board dimensions).

Once you have run pcb2gcode again, using the mirror-axis option, the generated gcode for front and back will now overlap perfectly.

Copy the drills.gcode to deepdrills.gcode.   Edit the deepdrills.gcode file and search for “change tool bit”.    For each occurence of this message, delete that block and the following movement block.    When you find the 3.175mm block, section, change the Z drill depth to -6mmmm (you can just global search and replace “Z-1.8” for “Z-6.”).

## Milling the board

I use [bCNC](https://github.com/vlachoudis/bCNC) on a raspberry pi to control my mill, but you can use whatever software your prefer.

Fix your spoilboard to the mill bed so that it will
not move (either double-sided tape or recessed T-bolts).  I mill a
slot in the side of my spoilboard to allow use of low-profile clamps.

You can also just use normal milling clamps to hold both the spoilboard and PCB, if you are careful to remove one clamp  at a time and re-clamp the spoilboard when removing the PCB to flip it (and be super careful not to crash into the clamps.  I lose more engraving bits that way!).

I use 17mm MDF for a spoilboard, cut on tablesaw into blocks the same size as my PCB blanks, and I re-use the spoilboard board many times until it is not flat (it can be re-flattened using a face cutter, a few times).

Clamp the PCB and spoilboard down.     If you have autoleveling probe, use it, but I have found that with 0.1mm narrow angle engraving bits this is not super critical, I do not typically use auto leveling.  If you’re using a 60degree V-bit, leveling its totally critical.

Zero the X and Y axis at the point where you want the lower left corner of your PCB. 
You won’t need to alter X and Y, but EVERY time you change a bit you must re-zero the Z axis.   

If you have Z-probing, use that to zero the Z-axis.   If not, lower the bit until a piece of paper is trapped under it.  For isolation set the height to 0.2mm (value may vary depending on type of paper you use, depth comes out wrong, adjust your value).   For drilling and outline cutting I just find “paper trapped height” and set that as zero.

Start with the FRONT isolation.

Do a SCAN (a bCNC option that moves the bit around the permiter of the
job, to demonstrate clearance) before routing to make sure you wont
crash.  Then run the front.gcode file, which will isolate tracks.  If
you’ve used fill zones, you will get a double isolation process.

Now do the drilling from the front.    You will probably change drill bits several times.  Leave whatever last drill you used (say 0.9mm) in place when it comes time to do the corner holes.

Finally put in a 3.175mm bit and run the deepdrills.gcode.    You now have holes that will let you flip the board perfectly.   This relies on your holes being placed equidistant from the board edge (or centerline) in your design.

Now you can flip the board.   It’s important not to move the spoilboard, so either clamp the spoilboard separately, or remove one clamp at a time and replace the clamp underneath the PCB until you have freed the PCB, and then re-clamp the flipped PCB the same way (this is more complicated to type than to do).

Use the locator pins to position the PCB and clamp it down.  THEN REMOVE THE PINS.   If you leave the pins in you risk breaking a bit if the mill decides to travel over a locator hole.    Do an autolevel if you wish.

Next do the isolation pass on the back side.   Before starting, do a border scan and maybe click around using “move gantry” to make sure that if you tell the bit to move to a particular hole on the computer, it it looks like the engraving bit lines up with that hole on the machine.

Finally do the outline pass, cutting the board from the panel, except for some bridges.

You’re done on the machine.   You can undo all the clamps and remove.

### Cleaning your board after milling

Use a quite stiff brush (a toothbrush-type brush is good, but brass or stainless steel are quicker) to get the phenolic or fiberglass dust out of the routed isolations.    Rub the burrs off each side with some 800 grit wet and dry sandpaper.    Using water may help to get the dust out of crevices.    A 1.5cm wire cup brush in a dremel tool is an excellent way to speed up the deburring process (this usually obviates the need to use sandpaper).

### Soldering

When soldering, it really helps to use flux paste from a syringe because normally you are soldering to a pre-tinned board which is much easier compared to bare copper.   With flux paste applied soldering becomes pleasant again.  Use an extractor fan.

You can use via rivets or short pieces of wire to fill vias.    I take about 8 pieces of wire and place them in 8 vias, solder those, cut and do 8 more.   Then I flip the board and solder the other sides.

If you have through hole pins that meet tracks on the top side, make sure you solder the pin to the pad on BOTH SIDES.    With headers, female headers are much easier to do than male headers, as you can solder the back side, lift the shroud slightly, solder the front side where necessary (I only do this where a track meets a pin), then replace the shroud.   Using a vise can help to squeeze the shroud back into place.   Make sure to support the pins so that you don’t lift any bottom tracks when pushing the shroud back down.



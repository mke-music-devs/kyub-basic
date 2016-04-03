#Kyub Basic

- [Global Variables](#global-variables)
- [Debugging Settings](#debugging-settings)
- [Main Loop](#main-loop)
- [Charge Up](#chargeup)

In this program, each pad (other than the two ground pads) is statically assigned to a different note in the scale.  Six scales have been selected, all in the key of C: the major scale, the minor scale, the major pentatonic scale, the blues scale, a Japanese scale, and a keltzmer scale.  The scales may be selected using the Kyub mode button

The basic operation of the program is to sense a touch on any of the pads and then to check the accelerometer for an associated acceleration.  The pad number is used to define a pitch of the MIDI command sent from the device and acceleration peak is used to define the volume of that MIDI command.  In this way both pitch and volume are accurately controlled.

The program is for a Teensy 2.0 so you'll need to download a file to augment the Arduino IDE  at http://www.pjrc.com/teensy.  Under the "Tools" menu you need to select the Teensy 2.0, CPU speed 16 megahertz, and USB type: MIDI.

## Global Variables
This section provides some description of the variables used in the program and is probably best referenced during or after review of the actual program.  The assignment of pins of the Teensy is done here.  In addition the state variable "padmode" providing a state for each separate pad of the Kyub is defined.  This state variable provides the ability to continuously loop the program for low latency while still keeping track of tasks that need to be accomplished for each pad (e.g. sending a MIDI note, sending a MIDI note off, checking for acceleration, etc.)

## Debugging Settings
//debugging settings

The program can send data to the Serial Monitor of the Arduino IDE.  Normally set this flag to 1 for lowest program latency.  Setting this flag to zero causes a capacitive overflow during sensing to be displayed.  Setting this flag 2 causes the raw capacitance data to be output, useful for checking the wiring on the pads (order) and sensitivity.  Setting this flag to 3 causes the output of the accelerometer to be displayed at each note strike (useful for calibrating the accelerometer).  


## Main Loop
It is important that the main loop not stall at any given task during operation of the Kyub because the loop cycle time defines how often the individual pads are polled.  Accordingly instructions like "delay" are largely absent and time-consuming routines are broken up into small chunks that can be implemented over several loops.  

//set scale according to presses of mode button

This section of the code response to the Mode button to switch between the scales assigned to the pads.  A different color denotes the particular scale selected in the Plexiglas Kyub.  Because this code does not execute during normal playing of the Kyub a one second delay is implemented for debouncing of the pushbutton.

//pad calibration--early after boot

How long it takes each pad to charge can vary depending on the length of the connecting wires etc..  This code section performs an initial calibration.  For this reason when the Kyub is reset you want to have your fingers removed from any of the active (non-ground) pads and have the queue oriented in a normal vertical orientation.

//loop through each of 11 pads according to pnum

The program needs to respond quickly to any pad touch and accordingly can't stop to wait for other code sections.  For this reason all of the processing is distributed among the looping that occurs during the section using state variables.  It is a way to create a low overhead real-time operating system.  The program looks at each of the 11 pads one at a time in this section

//for high speed, read A/D for x, y, and z interleaved at times of necessary delay

The A/D converter can only move so fast and so instead of wasting time waiting for the A/D converter to charge, the A/D conversions are separated by delays during which actual processing occurs.  This makes the code a little messy but is a trade-off for speed.

## CHARGEUP
//Chargeup

The pad currently indexed by the loop is charged up through one megaohm resister.  The program then waits for it to reach the high value and measures this time.  For larger capacitances that occur when the pad is being touched, this charge up time is longer.  During the charge up time, the A/D converter is used to read the x-axis acceleration which is stuffed into a circular buffer that will be used later.  While the A/D is finishing the conversion, and after the pad has ostensibly been fully charged, the pad is allowed to continue to charge to its full state.  Generally full charge is sensed by the Teensy somewhat before the capacitor is really full.

The same pad is then discharged through the one megaohm resister.  The charge and discharge time will be averaged to get rid of a gate threshold bias.  Again A/D conversions for the y-and z- axes are slipped in here at certain points where there is an apparent delay in the measurement process.

//touch detected ****************************

The charge time is compared to the calibration earlier to decide if a pad was being touched.  If so, a state variable is set indicating that there is a note that needs to be played associated with that pad (pad mode equals zero).  In addition the pad state is marked indicating that the pad is in a state of being touched.

//check touch induced acceleration

The circular buffer holding all of those acceleration values is then checked to find a peak acceleration near the touch point (either before or after it).  Sometimes the peak occurs before the touch is detected possibly because it takes a while to scan through all of the pad.  Each of the peaks (x, y, z) is stored in a different variable that will be used depending on which pad was touch

//load up pending all notes with volume numbers

Peak accelerations are matched to the notes that are waiting to play depending on the location of the pad on the Kyub aligning with the different axes of acceleration.

//play notes

Accelerations are converted to a note volume using a simple formula (times four).  More sophisticated acceleration curves could be added here.  The MIDI note-on command is sent, storing the note value so that it can be turned off later.  The LED is turned on to an appropriate color arbitrarily selected for pad

//turn off notes

When the pad is released, the corresponding node is turned off.


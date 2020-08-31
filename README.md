


updated 10/2014 ejo
run-psec4 command-line functions and arguments
Printed on GNU/Linux system on Mon 31 Aug 2020 10:14:28 AM CDT

PSEC4 evaluation board software -- for linux systems
This is linux-based software (Ubuntu and RedHat tested) for the PSEC4 eval board.

Eval board

### 1) Required Packages:
g++
libusb-dev (libusb-devel on RedHat/SL) (need 'development headers' for USB interface)
gnuplot (for oscilloscope feature)

### 2) Check working board [USB plugged in and +5 V powered on]:
Power: ~300 mA at 5 V
Check device:
```
$ lsusb
Bus X Device X: ID 6672:2920  
```

LEDs: top LED blinks at 1 Hz and the bottom LED is solid. Middle LED is toggled when triggered/readout.

### 3) Add USB device read/write privileges to non-sudo user [optional step]:
Put the following line in the file [or create this file] /etc/udev/rules.d/80-local.rules (or other rules file):

```
SUBSYSTEM=="usb",ATTRS{idVendor}=="6672",MODE="666"
SUBSYSTEM=="usb_device",ATTRS{idVendor}=="6672",MODE="666"
```

and then

```
sudo udevadm control --reload-rules
```
### 4) Compiling:
```
$ make
```

### 5) Running:
BNC external trigger input toggles on rising edge
When running on a hardware (hw) trigger, make sure external trig is plugged into board BNC input
Note: If skipped section (3): running an executable requres 'sudo' prefix
Make calibration data:

##### The linearity scan calibrates the conversion of ADC-counts to Volts on each of 256 samples of each channel. 
The scan procedure uses an on-board digital to analog converter (DAC) to output 0 to 4000 DAC counts in steps of 20. This takes a bit of time, and only needs to be run once per board. The calibration file is saved in cal_data and is used during LOG_DATA routines to correct raw data during on-line logging:
```
$ ./bin/LinScan
```

##### The pedestal sets a DC offset on the channel inputs. 
The range of the PSEC4 ADCs is 1.2V over 4096 ADC counts on an AC coupled input. If the pedestal is 0 mV, then a negative polar pulse will be clipped (possibly damage the inputs) and a positive polar pulse will have 1.2V of head room. If the pedestal is 600 mV, then a negative polar pulse has -600 mV of head room. conversion: 2048 ~ 600 mV. 

Set the pedestal before data taking (often). This sets the level of the DACs:
```
$ ./bin/SetPed 2048
```

Each of 256 samples of the PSEC4 chip have a slightly different response to the DAC output. There is some variation in the measured DC offset of each sample relative to the setpoint of the DAC. This variation may be calibrated out by measuring 100 events (for example) with no signals expected on the input and averaging the resulting sample voltages. Run this calibration after setting a pedestal:
```
$ ./bin/TakePed
```



##### Other executables:

'refresh', prints some diagnostics
```
$ ./bin/refresh 
```

oscilloscope, arguments: [num_frames] [trig_mode = 0 for sw, 1 for hw]
```
$ ./bin/ScopeData 10 0
```

save data, arguments [filename] [num events] [trig mode = 0 for sw, 1 for hw]
```
$ ./bin/LogData test_data 1000 0
```

### 6) Triggering:
There are 3 trigger modes:

-software (trigger from PC over USB). useful when saving CW signals or baseline levels (pedestals)
-hardware external (BNC input, 3.3V logic level, triggers on rising edge). To record data with reference to external signal, you want hardware trigger (set trig_mode to 1 in LogData or ScopeData). BNC trigger input takes CMOS level; triggers on rising edge. To capture the pulse of interest in the PSEC4 window, the rising edge of the trigger signal should be close-in-time with the pulse of interest (this accounts for PCB and FPGA delays to the trigger signal)
-hardware internal (self-triggering on RF SMA inputs into PSEC4). Inputs to PSEC4 should be < 1 Vpp!!!! To use the internal ('self') trigger of the PSEC4, here are the relevant functions:

##### Trigger configurations
Set threshold level and pulse polarity, where trig_thresh 12 bit number between 0 and 4095. Trig sign indicates pulse polarity: [0] for (-) pulse, [1] for (+) pulse:
```
$ ./bin/SetInternalTrig [trig_thresh] [sign]
```

Enable channel and self-trigger mode, where 1=on 0=off, channel = 0 (no channel) and 1-6 (select live channel):
```
$ ./bin/EnableTrig [on/off] [channel]
```

### Running example
For example, if you're digitizing PMT pulses on the first channel of PSEC4:

$ ./bin/SetPed 2048
(sets pedestal is at 2048 counts)

$./bin/SetInternalTrig 1800 0  
(sets threshold to 1800 counts, and negative PMT pulse polarity. pedestal is at 2048 counts)

$ ./bin/EnableTrig 1 1
(enable trigger on first channel)

and look at pulses...

$ ./bin/ScopeData 100 1




## Quick ./bin/ executable guide. Find more in depth guide below.

### Run each executable with ./bin/<name> from the head directory. The code accesses files from relative file-path, and assumes the executing directory is just above "bin" and "cal_data"

refresh :: prints some PSEC4 diagnostic info to screen
refresh :: takes 0 arguments

RawRead :: dumps raw event to screen and RAWREAD.txt
RawRead :: takes 0 arguments

SetPed :: is used to set the offset voltage at PSEC4 input
SetPed :: takes 1 argument: PED_VALUE [unsigned int]

TakePed :: runs pedestal calibration and saves constants to file: cal_data/PED_DATA.txt
TakePed :: takes 0 arguments

LinScan :: generates linearity correction and count-to-voltage conversion LUT for PSEC4 ADC
LinScan :: takes 0 arguments

EnableTrig :: is used to turn on/off self triggering option, and to specify trigger channel
EnableTrig :: takes 2 arguments: ENABLE [1=ON, 0=OFF]  Channel_mask [int 1-6]

SetInternalTrig :: is used to set the self trigger sign and voltage threshold level
SetInternalTrig :: takes 2 arguments: TRIG_SIGN [1=(-pulse), 0=(+pulse)]  TRIG_TRESH [unsigned int]

ScopeData :: is used look at waveforms via gnuplot
ScopeData :: takes 2 arguments: number_waveforms [unsigned int]  mode [1=ext-trig 0=soft-trig]

LogData :: is used to save PSEC4 data to a specified .txt file
LogData :: takes 3 arguments: filename [string]  number_readouts [unsigned int]  trig_mode [0=software, 1=external source]




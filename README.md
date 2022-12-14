# align_HV_sync / 
Script to read in a 40msps CVBS flie and line up syncs.

NOTE: this program should probalby be written in C/C++.  It is written in Perl because that is what i am most fluent in, and needed to get the task done quickly. Sorry ;).

-  alignsyncs =  aligns H and V syncs only, and only based on the left edge of the HSYNC pulse
-  alignsyncsPLUS = aligns H and V syncs, and optionally (flags in code), analyzes and aligns the lines better, and possibly replaces really bad ones
-  alignsyncsPLUSmt = same as above except multithreaded AND doesn't drop bad frames, this could cause a bad frame or blank frame to be written to the ouput file



i have been working on a process to try and improve head switch distortions and needed the frame to be sync'd to check the improvements i was making.  So i wrote a script that does rough sync. Right now it is LUMA only, but if you have the color info in a parallel file, you'd just do the seek and write operations on both data files. It's also NTSC VHS only, but for 625 line VHS, you just need to play with the positional numbers a bit. 

I chose the following approach, working with an unisgned 8 bit 40msps source file, that is already properly "clamped". The input for this program, is the output of my GNRC flowgraph:
https://github.com/tandersn/GNRC-Flowgraphs/blob/main/vhs_FM_dmod/vhs_FM_dmod_V3_AGC.grc

- use a 1000 sample moving average and check for when the average drops below 30 (35 may be better). This only happens at the BR pulses towards the beginning of each fields. 

- Once found, move seek position ahead the number of samples to position just before BR pulses end.

- count how many samples are over 30, resetting every time it is under 30, if reach > 2000 samples over 30 within the specified length, then it is bottom field (bottom field last EQ PULSE runs into next line).

- If first field read is bottom field, skip ahead one field worth of samples and go back to step 1. If first field read is top field, read in a chunk of data that is a full frame plus the number of samples for about 1.5 extra lines (i'm doing 526 lines like vhs-decode and ld-decode tbc files)

- iterate BACKWARDS through the data until I find the right edge of the HSYNC pulse.

- iterate BACKWARDS through the HSYNC pulse until I find the left edge of the HSYNC pulse.

-  move forwards 2 samples, and then store 2542 samples in an array that holds the frame lines.

-  move backwards 2000 samples (4/5ths of a line).

- repeat steps 6 through 9 until i run out of lines.

- iterate backwards through the array and write the lines out to a file (so the lines are written forwards).

- skip forward a little less than a frame's worth of samples and start back at step 1.

- end.

Video comparing head switch distortion:
[![Watch the video](https://raw.githubusercontent.com/tandersn/GNRC-Flowgraphs/main/z_images/hscompare.png)](https://www.youtube.com/watch?v=UTNsJvXHMl8)

Example of a clean tape aligment:
![pic1](https://raw.githubusercontent.com/tandersn/GNRC-Flowgraphs/main/z_images/aligned_out_3.png)

Example of very poor tape aligment:
![pic1](https://raw.githubusercontent.com/tandersn/GNRC-Flowgraphs/main/z_images/aligned_out_20.png)

This approach seems to be immune to macrovision. Example of macrovision tape aligment:
![pic1](https://raw.githubusercontent.com/tandersn/GNRC-Flowgraphs/main/z_images/aligned_out_69.png)




# gqrx-scanner
A frequency scanner for [Gqrx Software Defined Radio](http://gqrx.dk/) receiver
## Description

gqrx-scanner is a frequency scanner written in C that uses [gqrx remote protocol](http://gqrx.dk/doc/remote-control) to perform a fast scan of the band. It can be used in conjunction with the gqrx bookmarks (--mode bookmark) to look for the already stored frequencies or, in a free sweep scan mode (--mode sweep), to explore the band within a specified frequency range (--min, --max options). 

The sweep scan uses an adaptive algorithm to remember the active frequencies encountered during the sweep, that prioritizes active stations without stopping to look for new ones. 

Gqrx Squelch level is used as the threshold, when the signal is strong enough it stops the scanner on the frequency found.
After the signal is lost the scanner waits a configurable ammount of time and restart the loop (--delay option). 

In sweep mode the scan of the band is performed fast (well, as fast as it can), self asjusting the scanning speed. On signal detection a fine tuning is performed to pinpoint the nearest carrier frequency (subdivision of upper and lower limits with increasing precision) and, for the already seen carriers, a previous value is used to avoid fine tuning at every hit. This value is also averaged out with the new one in order to converge eventually to the exact frequency in less time (after 4 hits of the same frequency).

## Features
* Support for Gqrx bookmarks file
* Fast sweep scan with adaptive monitor of the most active stations 
* Frequency range constrained scan (also for bookmarks)
* Automatic Frequency Locking in sweep mode
* Interactive monitor to skip, ban or pause a frequency manually

## Notes on Gqrx settings
Gqrx Remote Protocol must be enabled: Tools->Remote Control. See [this](http://gqrx.dk/doc/remote-control).

It is advisable to disable AGC during the scan: adjust the fixed gain lowering the noise floor to at least -60/-70 dBFS and set the squelch level to -50/-40 dBFS, depending on the band activities and noise levels.

The best results are obtained in relative quiet frequencies with sporadic transmissions. If the band is cluttered with armonics and other types of persistent noise, avoid the sweep scan and use the bookmarks mode with a higher squelch level (or use the 'b' key to ban a frequency from the scan). 

In sweep mode use a limited bandwidth of about 2 MHz in order to avoid VFO and noise floor levels chainging during the sweep.
If you don't pass the --min, --max option, set the demodulator frequency to the middle of the screen and start from there (to avoid the panadapter to move during the scan).

You may also consider to adjust FFT options: FFT size and rate (on FFT Settings) to improve performances (and cpu usage).
I have found better result with high fft size (64536) and 17 fps refresh rate, but this depends on your hardware.

## Command line Options
```
Usage:
./gqrx-scanner	[-h|--host <host>] [-p|--port <port>] [-m|--mode <sweep|bookmark>]
		[-f <central frequency>] [-b|--min <from freq>] [-e|--max <to freq>]
		[-d|--delay <lingering time in seconds>]
		[-t|--tags <"tag1|tag2|...">]
		[-v|--verbose]

-h, --host <host>            Name of the host to connect. Default: localhost
-p, --port <port>            The number of the port to connect. Default: 7356
-m, --mode <mode>            Scan mode to be used. Default: sweep
                               Possible values for <mode>: sweep, bookmark
-f, --freq <freq>            Frequency to scan with a range of +- 1MHz.
                               Default: the current frequency tuned in Gqrx Incompatible with -b, -e
-b, --min <freq>             Frequency range begins with this <freq> in Hz. Incompatible with -f
-e, --max <freq>             Frequency range ends with this <freq> in Hz. Incompatible with -f
-d, --delay <time>           Lingering time in seconds before the scanner reactivates. Default 2
-t, --tags <"tags">          Filter signals. Match only on frequencies marked with a tag found in "tags"
                               "tags" is a quoted string with a '|' list separator: Ex: "Tag1|Tag2"
                               tags are case insensitive and match also for partial string contained in a tag
                               Works only with -m bookmark scan mode
-v, --verbose                Output more information during scan (used for debug). Default: false
--help                       This help message.

Examples:
./gqrx-scanner -m bookmark --min 430000000 --max 431000000 --tags "DMR|Radio Links"
	Performs a scan using Gqrx bookmarks, monitoring only the frequencies
	tagged with "DMR" or "Radio Links" in the range 430MHz-431MHz
./gqrx-scanner --min 430000000 --max 431000000 -d 3
	Performs a sweep scan from frequency 430MHz to 431MHz, using a delay of 
	3 secs as idle time after a signal is lost, restarting the sweep loop when this time expires

```

## Interactive Commands 
These keyboard shortcuts are available during scan:
```
[space] OR [enter]  :   Skips a locked frequency (listening to the next).
'b'                 :   Bans a locked frequency, the bandwidth banned is about 10 Khz from the locked freq. 
'c'                 :   Clears all banned frequencies.
'p'                 :   Pauses scan on locked frequency, 'p' again to unpause. 
```


## TODOs
* set modulation in bookmark search 
* automatic audio recording on signal detection
* parsable output in csv?


## Build and Install
```
cmake .
make
sudo make install
```


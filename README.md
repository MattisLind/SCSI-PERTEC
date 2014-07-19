SCSI-PERTEC
===========

SCSI PERTEC interface

The intention is to build an interface to attach PERTEC compatible 9 track drives to a SCSI host.

For this purpose I plan to us an STM32 processor and some interface logic.

The PERTEC interface
--------------------

The PERTEC interface is carried over two cables with 50 conductors. In total there are 52 signals and 48 ground.
It is very well described on this [webpage](http://www.sydex.com/pertec.html "webpage"). My drive is a Cipher  [F880](http://bitsavers.trailing-edge.com/pdf/cipher/799816-003O_F880vol1_Nov85.pdf "F880") which is not using all the signals described on referenced page.

P1 connections
--------------

    P1 Pin   Signal Name	Asserted by  Used by F880      Description
     2	       IFBY	        Drive	       Yes             Formatter busy—Set by trailing edge of IGO, clears when command finished (but you can send a new command as soon as IDBY clears)
     4	       ILWD	        Host	       Yes             Last word—Used to tell drive that this is the last word (byte) to be written in this record, Must be asserted at least 300 nsec. before trailing edge of final IWSTR pulse.
     6	       IW4	        Host	       Yes             Write data bit 4
     8	       IGO	        Host	       Yes             Initiate command—Pulsed low for at least 1µsec. to start command execution. The formatter address lines must be stable throughout the pulse and until IFBY drops.
    10	       IW0	        Host	       Yes             Write data bit 0 (MSB!)
    12	       IW1	        Host	       Yes             Write data bit 1
    14	       ISGL	        Drive	       Reserved Input  Selected drive fault on some drives; NC on others. If used, it is cleared by de-asserting, then re-asserting IFEN.
    16	       ILOL	        Host	       Reserved Input  Load on-line—Pulsing this line at least 1µsec. begins the tape load sequence on many drives.
    18	       IREV	        Host	       Yes             Reverse—When asserted, indicates operation is to be performed in the reverse direction.
    20	       IREW	        Host	       Yes             Rewind—A pulse of at least 1 µn;sec. starts the tape rewind sequence. Completion is signaled by IRWD and IRDY being asserted by the drive.
    22	       IWP	        Host	       Yes             Write parity. This is (almost always) odd parity computed over IW0–IW7 and on some drives is ignored and computed by the formatter.
    24	       IW7	        Host	       Yes             Write data bit 7 (LSB!)
    26	       IW3	        Host	       Yes             Write data bit 3
    28	       IW6	        Host	       Yes             Write data bit 6
    30	       IW2	        Host	       Yes             Write data bit 2
    32	       IW5	        Host	       Yes             Write data bit 5
    34	       IWRT	        Host	       Yes             Write—When asserted with IGO begins a write sequence.
    36	       IRTH2        Host	       No              Write density select 2—On some drives, this signal, along with IRTH1 is used to select the drive write density. If it is implemented, it is valid only at BOT and must be asserted with IGO during the first write sequence.
    38	       IEDIT        Host	       Yes             Edit—Not implemented on all drives. See notes in the next section for details on its application.
    40	       IERASE       Host	       Yes             Erase—When asserted with IWRT and IGO, causes tape to be erased, usually for a predetermined length. Usually used to recover from write errors or provide extra space for mode changes (i.e. Read after write).
    42	       IWFM	        Host	       Yes             Write filemark; When asserted with IWRT, writes a filemark.
    44	       IRTH1        Host	       Unknown output  Write density control—See IRTH2, pin 36.
    46	       ITAD0        Host	       Yes             Transport address bit 0 (MSB!)—Used to address multiple drives on a single controller.
    48	       IR2	        Drive	       Yes             Read data bit 2
    50	       IR3	        Drive	       Yes             Read data bit 3
         
    All odd	 Gnd	 –	 Signal ground

P2 Connections
--------------

    P2 Pin	 Signal Name	Asserted by	 Used by F880      Description
     1	       IRP	        Drive	       Yes             Read data parity (odd) This is not a ground pin!
     2	       IR0	        Drive	       Yes             Read data bit 0 (MSB!)
     3	       IR1	        Drive	       Yes             Read data bit 1 This is not a ground pin!
     4	       ILDP	        Drive	       Yes             Load point—Asserted whenever the tape is at the load point (or might as well be, as far as the controller needs to know -- there may be hidden repositioning)
     6	       IR4	        Drive	       Yes             Read data bit 4
     8	       IR7	        Drive	       Yes             Read data bit 7 (LSB!)
    10	       IR6	        Drive	       Yes             Read data bit 6
    12	       IHER	        Drive	       Yes             Hard error—Pulsed during IDBY when a hard data error (or illegal character in the IRG) is seen. Note that most modern formatters correct this error automatically.
    14	       IFMK	        Drive	       Yes             File mark—Pulsed during IDBY when a tape mark is seen.	Note that you have to catch this as it goes by!
    16	       IDENT        Drive	       Yes             Identification—Asserted while drive is actually reading the PE ID burst, dropped the rest of the time so it's up to the controller to catch it and remember.
    18	       IFEN	        Host	       Yes             Formatter enable—This signal should normally be asserted all the time. If dropped for a minimum of 2 µsec., aborts any command that asserts IDBY (I/O, skip, but not rewind or unload).
    20	       IR5	        Drive	       Yes             Read data bit 5
    22	       IEOT	        Drive	       Yes             End of tape—Asserted whenever the tape is past the EOT marker, clears when the tape is backspaced past it.
    24	       IRWU	        Host	       Yes             Rewind and unload—A pulse of at least 1 µsec. initiates a rewind-with-unload and sets drive off-line. Some drives require that IREW is also asserted.
    26	       INRZ	        Drive	       No              NRZI mode—Signals 800 BPI mode on some drives.
    28	       IRDY	        Drive	       Yes             Ready—Signals that tape is fully loaded, on-line and not rewinding. This must be asserted by the drive before it will accept any command.
    30	       IRWD	        Drive	       Yes             Rewinding—This is asserted by the drive while the tape is rewinding.
    32	       IFPT	        Drive	       Yes             File protect—This signal is asserted continuously when the tape is not write-enabled (i.e. no ring present).
    34	       IRSTR        Drive	       Yes             Read strobe—Pulses low for at least 200 nsec. when data on IR0-IR7 and IRP are stable before leading edge; typically held for 200 nsec., but this is not a requirement. Note that data is made available–there is no check made to ensure that the host has picked it up.
    36	       IWSTR        Drive	       Yes             Write strobe—Pulses low for at least 200 nsec. when the drive is ready to accept data. This is roughly similar to an "Acknowledge" signal after data has been received. The next data byte can be presented immediately.
    38	       IDBY	        Drive	       Yes             Data busy—This signal is asserted during I/O phase of read or write commands. It generally lags a few milliseconds after the drive asserts IFBY.
    40	       ISPEED       Drive	       Yes             High-speed—This signal is asserted when commands are executing in high-speed mode; that is, when the drive is not operating in start-stop mode.
    42	       ICER	        Drive	       Yes             Corrected error—This signal is pulsed during IDBY when a single-track dropout is successfully corrected using the parity information.
    44	       IONL	        Drive	       Yes             Online—Asserted when the drive is online; clears within 1 µsec. of being taken offline.
    46	       ITAD1        Host	       Yes             Transport address bit 1 (LSB!)
    48	       IFAD	        Host	       Yes             Formatter address
    50	       IHISP        Host	       Yes             High speed select—When asserted 1 µsec. before and then with IGO (de-asserted any time after the trailing edge of IGO selects high-speed (streaming) mode.
     
    5-49 odd	 Gnd	 –	 Signal ground.
    
In total 47 signals is used by the F880. In addition two that two input signals are indicated as RESERVED. The signal INRZ is not used at all. IRTH2 is an output instead of an input and IRTH1 is not used at all.

The signals ITAD0, ITAD1 and IFAD could be grounded since usually only one drive is connected at a time. Which means that we could handle the PERTEC interface with a minimum of 44 signals.

Of these 23 signals are driven by the host and requires open collector drivers. That is four 7406 chips. The inputs can be connected directly to the STM32 inputs since these are +5V tolerant. The inputs require pull-up which is already inside the chip. If the cable length is short enough this should be ok.

Five signals in the PERTEC interface are pulsed. That is a short pulse is sent to indicate a certain condition or event. These are quite naturally the strobe signals, IWRSTR and IRSTR. But also the ICER, IHER, IFMK. The pulse length for the former two is specified in the document to at least 200 ns. From the schematics it is evident that the ciruitry in the drive will write the data read of the tape to a buffer register by a clock pulse. The writeing will take place using the rising edge of the signal. The falling edge will trigger a mono stable flip-fop which will trigger another mono stable flip-flop which is the IRSTR signal. Based on the R and C avlues used for these mono stable flip-flops I can deduce that the delay woul be approximately 900 ns and the the pulse length would be 1.5 us. SInce the maximum speed of the F880 is 100 ips and the densitity is 1600 bpi it would at maximum transfer 160 kByte per second, thus one word each 6.7 us. Higer speed would use different timing one could assume.

The use of pulses requires edge triggered interrupts in the STM32 chip. There are 16 external interrupts that could come from GPIO pins.

SCSI interface
--------------

SCSI defines an initiator which is the host and a target which is the drive. The interface defines 9 data signals, 8 plus parity, and 9 control signals. The control signals are:
* RES
* BSY
* SEL
* C/D
* MSG
* I/O
* REQ
* ACK
* ATN

RES, BSY and SEL can be driven by both initiator and target. ATN and ACK is always driven by initiator and C/D, MSG, I/O and REQ is always driven by the target.

To be able to both drive and receive a signals requires one input and one output. Thus we need 12 I/O signals for the SCSI control signals.

Normally the data bus is either in or out so it would be easy to assume that just nine signals plus a direction signal is required for the data bus. But in the arbitration phase SCSI devices would drive one single data line and watch the others. Thus we need one input and one output per data bus signal. I.e 18 I/O signals. But since we could use a jumper to select the device own address in the aribtration phase a single pair of receiver and transmitter is reuired except for the already mentioned 10 signals. Thus the total ould be 12 I/O signals for the data bus. On the other hand this means that three-state drivers are required on the data inputs.

In escence ther are two options either higher pin requirements on the STM32 or more complex driver logics.

Option 1  would use 30 I/O signals and three 7406 chips for driving. The inputs go directly to the STM32 inputs since these are +5V tolerant.

Option 2 would use 24 I/O signals also require three 7406 chips, but also a 74LS240 for the data bus and one pair of signals for the parity bit and the select bus bit. Adding up to 25 signals.

In total SCSI would require either 25 or 30 I/O port signals from the STM32 chip depending on option chosen.





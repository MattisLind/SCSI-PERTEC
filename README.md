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

Option 2 would use 24 I/O signals require two 74LS38 chips and also a 74LS244 for the data bus and one pair of signals for the parity bit and the bus direction bit. Then two 7416 chips for the control signals that the is transmitted by the device. Adding up to 25 signals.

In total SCSI would require either 25 or 30 I/O port signals from the STM32 chip depending on option chosen.

STM32 CORE207 BOARD MAPPING
---------------------------


| LQFP100 PIN | BASE FUNCTION         | FUNCTION PERTEC-SCSI  | CORE207V BOARD         | IN OR OUT | FIVE VOLT? | NOTES | ALTERNATE FUNCTIONS                                                                                                           | ADDITIONAL FUNCTIONS |
|-------------|-----------------------|-----------------------|------------------------|-----------|------------|-------|-------------------------------------------------------------------------------------------------------------------------------|----------------------|
| 1           | PE2                   | J1:8 IGO              |                        | I/O       | FT         |       | TRACECLK, FSMC_A23, ETH_MII_TXD3, EVENTOUT                                                                                    |                      |
| 2           | PE3                   | J1:4 ILWD             |                        | I/O       | FT         |       | TRACED0,FSMC_A19, EVENTOUT                                                                                                    |                      |
| 3           | PE4                   | J1:20 IREW            |                        | I/O       | FT         |       | TRACED1,FSMC_A20, DCMI_D4, EVENTOUT                                                                                           |                      |
| 4           | PE5                   | J1:18 IREV            |                        | I/O       | FT         |       | TRACED2, FSMC_A21, TIM9_CH1, DCMI_D6, EVENTOUT                                                                                |                      |
| 5           | PE6                   | J1:22 IWP             |                        | I/O       | FT         |       | TRACED3, FSMC_A22, TIM9_CH2, DCMI_D7, EVENTOUT                                                                                |                      |
| 6           | VBAT                  |                       |                        | S         |            |       |                                                                                                                               |                      |
| 7           | PC13                  | J1:34 IWRT            |                        | I/O       | FT         | 2,3   | EVENTOUT                                                                                                                      | RTC_AF1              |
| 8           | PC14/OSC32_IN (PC14)  |                       | 32768 Hz XTAL          | I/O       | FT         | 2,3   | EVENTOUT                                                                                                                      | OSC32_IN(4)          |
| 9           | PC15-OSC32_OUT (PC15) |                       | 32768 Hz XTAL          | I/O       | FT         | 2.3   | EVENTOUT                                                                                                                      | OSC32_OUT(4)         |
| 10          | VSS                   |                       |                        | S         |            |       |                                                                                                                               |                      |
| 11          | VDD                   |                       |                        | S         |            |       |                                                                                                                               |                      |
| 12          | PH0/OSC_IN (PH0)      |                       | XTAL                   | I/O       | FT         |       | EVENTOUT                                                                                                                      | OSC_IN(4)            |
| 13          | PH1/OSC_OUT (PH1)     |                       | XTAL                   | I/O       | FT         |       | EVENTOUT                                                                                                                      | OSC_OUT(4)           |
| 14          | NRST                  |                       | RESET BUTTON/ JTAG CON | I/O       |            |       |                                                                                                                               |                      |
| 15          | PC0                   | J1:40 IERASE          |                        | I/O       | FT         | 4     | OTG_HS_ULPI_STP, EVENTOUT                                                                                                     | ADC123_ IN10         |
| 16          | PC1                   | J1:38 IEDIT           |                        | I/O       | FT         | 4     | ETH_MDC, EVENTOUT                                                                                                             | ADC123_ IN11         |
| 17          | PC2                   | J2:18 IFEN            |                        | I/O       | FT         | 4     | SPI2_MISO, OTG_HS_ULPI_DIR, ETH_MII_TXD2, EVENTOUT                                                                            | ADC123_ IN12         |
| 18          | PC3                   | J2:24 IRWU            |                        | I/O       | FT         | 4     | SPI2_MOSI, I2S2_SD, OTG_HS_ULPI_NXT, ETH_MII_TX_CLK, EVENTOUT                                                                 | ADC123_ IN13         |
| 19          | VDD                   |                       | JUMPER COMP.J1 (3.3V)  | S         |            |       |                                                                                                                               |                      |
| 20          | VSSA                  |                       |                        | S         |            |       |                                                                                                                               |                      |
| 21          | VREF+                 |                       |                        | S         |            |       |                                                                                                                               |                      |
| 22          | VDDA                  |                       |                        | S         |            |       |                                                                                                                               |                      |
| 23          | PA0-WKUP (PA0)        | J2:8 IR7              |                        | I/O       | FT         | 4,5   | USART2_CTS, UART4_TX, ETH_MII_CRS, TIM2_CH1_ETR, TIM5_CH1, TIM8_ETR, EVENTOUT                                                 | ADC123_IN0, WKUP     |
| 24          | PA1                   | J2:10 IR6             |                        | I/O       | FT         | 4     | USART2_RTS, UART4_RX, ETH_RMII_REF_CLK, ETH_MII_RX_CLK, TIM5_CH2, TIM2_CH2, EVENTOUT                                          | ADC123_IN1           |
| 25          | PA2                   | J2:20 IR5             |                        | I/O       | FT         | 4     | USART2_TX,TIM5_CH3, TIM9_CH1, TIM2_CH3, ETH_MDIO, EVENTOUT                                                                    | ADC123_IN2           |
| 26          | PA3                   | J2:6 IR4              | JUMPER USB.P1          | I/O       | FT         | 4     | USART2_RX, TIM5_CH4, TIM9_CH2, TIM2_CH4, OTG_HS_ULPI_D0, ETH_MII_COL, EVENTOUT                                                | ADC123_IN3           |
| 27          | VSS                   |                       |                        | S         |            |       |                                                                                                                               |                      |
| 28          | VDD                   |                       |                        | S         |            |       |                                                                                                                               |                      |
| 29          | PA4                   | J1:50 IR3             |                        | I/O       | TTa        | 4     | SPI1_NSS, SPI3_NSS, USART2_CK, DCMI_HSYNC, OTG_HS_SOF, I2S3_WS, EVENTOUT                                                      | ADC12_IN4, DAC_OUT1  |
| 30          | PA5                   | J1:48 IR2             | JUMPER USB.P2          | I/O       | TTa        | 4     | SPI1_SCK, OTG_HS_ULPI_CK, TIM2_CH1_ETR, TIM8_CH1N, EVENTOUT                                                                   | ADC12_IN5, DAC_OUT2  |
| 31          | PA6                   | J2:3 IR1              |                        | I/O       | FT         | 4     | SPI1_MISO, TIM8_BKIN, TIM13_CH1, DCMI_PIXCLK, TIM3_CH1, TIM1_BKIN, EVENTOUT                                                   | ADC12_IN6            |
| 32          | PA7                   | J2:2 IR0              |                        | I/O       | FT         | 4     | SPI1_MOSI, TIM8_CH1N, TIM14_CH1, TIM3_CH2, ETH_MII_RX_DV, TIM1_CH1N, ETH_RMII_CRS_DV, EVENTOUT                                | ADC12_IN7            |
| 33          | PC4                   | J2:50 IHISP           |                        | I/O       | FT         | 4     | ETH_RMII_RXD0, ETH_MII_RXD0, EVENTOUT                                                                                         | ADC12_IN14           |
| 34          | PC5                   | J1:42 IWFM            |                        | I/O       | FT         | 4     | ETH_RMII_RXD1, ETH_MII_RXD1, EVENTOUT                                                                                         | ADC12_IN15           |
| 35          | PB0                   | J2:1 IRP              |                        | I/O       | FT         | 4     | TIM3_CH3, TIM8_CH2N, OTG_HS_ULPI_D1, ETH_MII_RXD2, TIM1_CH2N, EVENTOUT                                                        | ADC12_IN8            |
| 36          | PB1                   | J2:4 ILDP             |                        | I/O       | FT         | 4     | TIM3_CH4, TIM8_CH3N, OTG_HS_ULPI_D2, ETH_MII_RXD3, TIM1_CH3N, EVENTOUT                                                        | ADC12_IN9            |
| 37          | PB2/BOOT1 (PB2)       | J2:12 IHER            | JUMPER COMP.J4         | I/O       | FT         |       | EVENTOUT                                                                                                                      |                      |
| 38          | PE7                   | J2:14 IFMK            |                        | I/O       | FT         |       | FSMC_D4,TIM1_ETR, EVENTOUT                                                                                                    |                      |
| 39          | PE8                   | J1:24 IW7             |                        | I/O       | FT         |       | FSMC_D5,TIM1_CH1N, EVENTOUT                                                                                                   |                      |
| 40          | PE9                   | J1:28 IW6             |                        | I/O       | FT         |       | FSMC_D6,TIM1_CH1, EVENTOUT                                                                                                    |                      |
| 41          | PE10                  | J1:32 IW5             |                        | I/O       | FT         |       | FSMC_D7,TIM1_CH2N, EVENTOUT                                                                                                   |                      |
| 42          | PE11                  | J1:6 IW4              |                        | I/O       | FT         |       | FSMC_D8,TIM1_CH2, EVENTOUT                                                                                                    |                      |
| 43          | PE12                  | J1:26 IW3             |                        | I/O       | FT         |       | FSMC_D9,TIM1_CH3N, EVENTOUT                                                                                                   |                      |
| 44          | PE13                  | J1:30 IW2             |                        | I/O       | FT         |       | FSMC_D10,TIM1_CH3, EVENTOUT                                                                                                   |                      |
| 45          | PE14                  | J1:12 IW1             |                        | I/O       | FT         |       | FSMC_D11,TIM1_CH4, EVENTOUT                                                                                                   |                      |
| 46          | PE15                  | J1:10 IW0             |                        | I/O       | FT         |       | FSMC_D12,TIM1_BKIN, EVENTOUT                                                                                                  |                      |
| 47          | PB10                  | J2:16 IDENT           |                        | I/O       | FT         |       | SPI2_SCK, I2S2_SCK, I2C2_SCL,USART3_TX,OT G_HS_ULPI_D3,ETH_MII_R X_ER,TIM2_CH3, EVENTOUT                                      |                      |
| 48          | PB11                  | J2:22 IEOT            |                        | I/O       | FT         |       | I2C2_SDA, USART3_RX, OTG_HS_ULPI_D4, ETH_RMII_TX_EN, ETH_MII_TX_EN, TIM2_CH4, EVENTOUT                                        |                      |
| 49          | VCAP_1                |                       | JUMPER COMP.J2(2,2 uF) | S         |            |       |                                                                                                                               |                      |
| 50          | VDD                   |                       |                        | S         |            |       |                                                                                                                               |                      |
| 51          | PB12                  | J2:26 INRZ            |                        | I/O       | FT         |       | SPI2_NSS, I2S2_WS, I2C2_SMBA, USART3_CK, TIM1_BKIN, CAN2_RX, OTG_HS_ULPI_D5, ETH_RMII_TXD0, ETH_MII_TXD0, OTG_HS_ID, EVENTOUT |                      |
| 52          | PB13                  | J2:28 IRDY            |                        | I/O       | FT         |       | SPI2_SCK, I2S2_SCK, USART3_CTS, TIM1_CH1N, CAN2_TX, OTG_HS_ULPI_D6, ETH_RMII_TXD1, ETH_MII_TXD1, EVENTOUT                     | OTG_HS_ VBUS         |
| 53          | PB14                  | J2:30 IRWD            |                        | I/O       | FT         |       | SPI2_MISO, TIM1_CH2N, TIM12_CH1, OTG_HS_DM USART3_RTS, TIM8_CH2N, EVENTOUT                                                    |                      |
| 54          | PB15                  | J2:32 IFPT            |                        | I/O       | FT         |       | SPI2_MOSI, I2S2_SD, TIM1_CH3N, TIM8_CH3N, TIM12_CH2, OTG_HS_DP, RTC_50Hz, EVENTOUT                                            |                      |
| 55          | PD8                   | RS232 TX              |                        | I/O       | FT         |       | FSMC_D13, USART3_TX, EVENTOUT                                                                                                 |                      |
| 56          | PD9                   | RS232 RX              |                        | I/O       | FT         |       | FSMC_D14, USART3_RX, EVENTOUT                                                                                                 |                      |
| 57          | PD10                  | J2:38 IDBY            |                        | I/O       | FT         |       | FSMC_D15, USART3_CK, EVENTOUT                                                                                                 |                      |
| 58          | PD11                  | J2:40 ISPEED          |                        | I/O       | FT         |       | FSMC_A16,USART3_CTS, EVENTOUT                                                                                                 |                      |
| 59          | PD12                  | J2:42 ICER            |                        | I/O       | FT         |       | FSMC_A17,TIM4_CH1, USART3_RTS, EVENTOUT                                                                                       |                      |
| 60          | PD13                  | J2:44 IONL            |                        | I/O       | FT         |       | FSMC_A18,TIM4_CH2, EVENTOUT                                                                                                   |                      |
| 61          | PD14                  | LED                   |                        | I/O       | FT         |       | FSMC_D0,TIM4_CH3, EVENTOUT                                                                                                    |                      |
| 62          | PD15                  | LED                   |                        | I/O       | FT         |       | FSMC_D1,TIM4_CH4, EVENTOUT                                                                                                    |                      |
| 63          | PC6                   | J2:34 IRSTR           |                        | I/O       | FT         |       | I2S2_MCK, TIM8_CH1, SDIO_D6, USART6_TX, DCMI_D0, TIM3_CH1, EVENTOUT                                                           |                      |
| 64          | PC7                   | J2:36 IWSTR           |                        | I/O       | FT         |       | I2S3_MCK, TIM8_CH2, SDIO_D7, USART6_RX, DCMI_D1, TIM3_CH2, EVENTOUT                                                           |                      |
| 65          | PC8                   | BUTTON                |                        | I/O       | FT         |       | TIM8_CH3,SDIO_D0, TIM3_CH3, USART6_CK, DCMI_D2, EVENTOUT                                                                      |                      |
| 66          | PC9                   |                       |                        | I/O       | FT         |       | I2S2_CKIN, I2S3_CKIN, MCO2, TIM8_CH4, SDIO_D1, I2C3_SDA, DCMI_D3, TIM3_CH4, EVENTOUT                                          |                      |
| 67          | PA8                   | SCSI-DATA-DIRECTION   |                        | I/O       | FT         |       | MCO1, USART1_CK, TIM1_CH1, I2C3_SCL, OTG_FS_SOF, EVENTOUT                                                                     |                      |
| 68          | PA9                   | SCSI-SELECT-PHASE-OUT | JUMPER USB.P3          | I/O       | FT         |       | USART1_TX, TIM1_CH2, I2C3_SMBA, DCMI_D0, EVENTOUT                                                                             | OTG_FS_ VBUS         |
| 69          | PA10                  | SCSI-SELECT-PHASE-IN  | USB CON                | I/O       | FT         |       | USART1_RX, TIM1_CH3, OTG_FS_ID,DCMI_D1, EVENTOUT                                                                              |                      |
| 70          | PA11                  | SCSI-PARITY-OUT       | USB CON                | I/O       | FT         |       | USART1_CTS, CAN1_RX, TIM1_CH4,OTG_FS_DM, EVENTOUT                                                                             |                      |
| 71          | PA12                  | SCSI-REQ-OUT          | USB CON                | I/O       | FT         |       | USART1_RTS, CAN1_TX, TIM1_ETR, OTG_FS_DP, EVENTOUT                                                                            |                      |
| 72          | PA13 (JTMS-SWDIO)     |                       | JTAG CON               | I/O       | FT         |       | JTMS-SWDIO, EVENTOUT                                                                                                          |                      |
| 73          | VCAP_2                |                       |                        | S         |            |       |                                                                                                                               |                      |
| 74          | VSS                   |                       |                        | S         |            |       |                                                                                                                               |                      |
| 75          | VDD                   |                       |                        | S         |            |       |                                                                                                                               |                      |
| 76          | PA14 (JTCK-SWCLK)     |                       | JTAG CON               | I/O       | FT         |       | JTCK-SWCLK, EVENTOUT                                                                                                          |                      |
| 77          | PA15 (JTDI)           | SCSI-IO-OUT           | JTAG CON               | I/O       | FT         |       | JTDI, SPI3_NSS, I2S3_WS,TIM2_CH1_ETR, SPI1_NSS, EVENTOUT                                                                      |                      |
| 78          | PC10                  | SCSI-MSG-OUT          |                        | I/O       | FT         |       | SPI3_SCK, I2S3_SCK, UART4_TX, SDIO_D2, DCMI_D8, USART3_TX, EVENTOUT                                                           |                      |
| 79          | PC11                  | SCSI-CD-OUT           |                        | I/O       | FT         |       | UART4_RX, SPI3_MISO, SDIO_D3, DCMI_D4,USART3_RX, EVENTOUT                                                                     |                      |
| 80          | PC12                  | SCSI-BSY-OUT          |                        | I/O       | FT         |       | UART5_TX, SDIO_CK, DCMI_D9, SPI3_MOSI, I2S3_SD, USART3_CK, EVENTOUT                                                           |                      |
| 81          | PD0                   | SCSI-DATA             |                        | I/O       | FT         |       | FSMC_D2,CAN1_RX, EVENTOUT                                                                                                     |                      |
| 82          | PD1                   | SCSI-DATA             |                        | I/O       | FT         |       | FSMC_D3, CAN1_TX, EVENTOUT                                                                                                    |                      |
| 83          | PD2                   | SCSI-DATA             |                        | I/O       | FT         |       | TIM3_ETR,UART5_RX, SDIO_CMD, DCMI_D11, EVENTOUT                                                                               |                      |
| 84          | PD3                   | SCSI-DATA             |                        | I/O       | FT         |       | FSMC_CLK,USART2_CTS, EVENTOUT                                                                                                 |                      |
| 85          | PD4                   | SCSI-DATA             |                        | I/O       | FT         |       | FSMC_NOE, USART2_RTS, EVENTOUT                                                                                                |                      |
| 86          | PD5                   | SCSI-DATA             |                        | I/O       | FT         |       | FSMC_NWE,USART2_TX, EVENTOUT                                                                                                  |                      |
| 87          | PD6                   | SCSI-DATA             |                        | I/O       | FT         |       | FSMC_NWAIT, USART2_RX, EVENTOUT                                                                                               |                      |
| 88          | PD7                   | SCSI-DATA             |                        | I/O       | FT         |       | USART2_CK,FSMC_NE1, FSMC_NCE2, EVENTOUT                                                                                       |                      |
| 89          | PB3 (JTDO/TRACESWO)   |                       | JTAG CON               | I/O       | FT         |       | JTDO/ TRACESWO, SPI3_SCK, I2S3_SCK, TIM2_CH2, SPI1_SCK, EVENTOUT                                                              |                      |
| 90          | PB4                   | SCSI-PARITY-IN        |                        | I/O       | FT         |       | NJTRST, SPI3_MISO, TIM3_CH1, SPI1_MISO, EVENTOUT                                                                              |                      |
| 91          | PB5                   | SCSI-ATN-IN           |                        | I/O       | FT         |       | I2C1_SMBA, CAN2_RX, OTG_HS_ULPI_D7, ETH_PPS_OUT, TIM3_CH2, SPI1_MOSI, SPI3_MOSI, DCMI_D10, I2S3_SD, EVENTOUT                  |                      |
| 92          | PB6                   | SCSI-ACK-IN           |                        | I/O       | FT         |       | I2C1_SCL,, TIM4_CH1, CAN2_TX, DCMI_D5,USART1_TX, EVENTOUT                                                                     |                      |
| 93          | PB7                   | SCSI-RES-IN           |                        | I/O       | FT         |       | I2C1_SDA, FSMC_NL(6), DCMI_VSYNC, USART1_RX, TIM4_CH2, EVENTOUT                                                               |                      |
| 94          | BOOT0                 |                       | BOOT0 SWITCH           | I         | B          |       |                                                                                                                               | VPP                  |
| 95          | PB8                   | SCSI-RES-OUT          |                        | I/O       | FT         |       | TIM4_CH3,SDIO_D4, TIM10_CH1, DCMI_D6, ETH_MII_TXD3, I2C1_SCL, CAN1_RX, EVENTOUT                                               |                      |
| 96          | PB9                   | SCSI-SEL-IN           |                        | I/O       | FT         |       | SPI2_NSS, I2S2_WS, TIM4_CH4, TIM11_CH1, SDIO_D5, DCMI_D7, I2C1_SDA, CAN1_TX, EVENTOUT                                         |                      |
| 97          | PE0                   | SCSI-SEL-OUT          |                        | I/O       | FT         |       | TIM4_ETR, FSMC_NBL0, DCMI_D2, EVENTOUT                                                                                        |                      |
| 98          | PE1                   | SCSI-BSY-IN           |                        | I/O       | FT         |       | FSMC_NBL1, DCMI_D3, EVENTOUT                                                                                                  |                      |
| 99          | RFU                   |                       | JUMPER COMP.J3 (NC)    |           |            | 7     |                                                                                                                               |                      |
| 100         | VDD                   |                       |                        | S         |            |       |                                                                                                                               |                      |


### Gerber files

![PCB Layout](http://i.imgur.com/JL33uyx.png "PCB Layout")

I have done a simple layout for this project and all the gerber files are in the repository.

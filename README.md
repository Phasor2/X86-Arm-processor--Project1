# X86-Arm-processor-ECE372--Project1
Full report open ECE372 project1 2018 and code " * . txt"
## Initialization for UART2
To map the UART2, I have to know the pins available to the Beagle Bones Black P9 and P8 connector by changing the MUX that select the signal go to the pads.
![alt-text](https://github.com/Phasor2/Assembly-langguge-X86-Arm-processor-ECE372--Project1/blob/master/P8Header.PNG)
In table 10 and table 11 above, The UART that is needed for this project is TxD, RxD, CTS, RTS. The default of the MUX when initialized it will begin in MODE0. Essentially in the manual ARM355x, the register name that use to map will be named in MODE0. So all we need to look up LCD_data8, lcd_data9 change to mode 6 and spi0_d0, and sp0_sclk change to mode 1.
After mapping an and started the UART2 clock, next step is setting the desired Baud rate for UART to communicate with RC8660 Talker through RS-232C. This is a bit complex, there is a requirement for switching mode from operational mode UART to configuration mode A to change the DHL and DLL.  The clock is divided by the value written to the DLH and DLL register and the result is then divided by further by 13 or 16. So we all we need to do is change the mode write desired baud rate value into DHL and DLL. In the table 19-25 in the manual, the value that we want is 38.4 Kbps. As we can see we need to write 0x00 for DLH and 0x4E for DLL.
![alt-text](https://github.com/Phasor2/Assembly-langguge-X86-Arm-processor-ECE372--Project1/blob/master/UART%20Baud%20Rate%20settings.png)


# Interrupt unmasking 
Unmasking 2 interrupts in INTC register. UART2INT is at int number 74 and the button GPIO1A is at int number 98. For GPIO1A, the pin calculated was pin 2 at INTC_MIR_CLEAR3 register. For UART2INT, the pin calculated was pin 10 at INTC_MIR_CLEAR2. Simply write 0x04 to INTC_MIR_CLEAR3 and write 0x400 to INTC_MIR_CLEAR2.
# Sending bytes to talker
There are 2 important signal we need to check in order to send character to talker. First is Clear to send (CTS#) active low, we need to look at MODEM register (MSR) bit 4. Second is Transmit Holding Register (THR), we need to look at Line Control Register (LSR) bit 5
Now we need to focus MSR bit 4 and LSR bit 5 and when is the Last char in the String. Making a truth table will be easier to keep track what is the next step to do after sending character. Because of MSR bit 4 is the signal come from Talker so we need to priority check this bit.
```
MSR bit 4	LSR bit 5	Last character	Next step
0	0	x	Go to PASS ON
0	1	x	Masking THR interrupt and go to PASS ON
1	0 	X	Go to PASS ON
1	1	No	Sending a character and go to PASS ON
1	1	Yes	Sending a character  and Set the Pointer to first Char then go to PASS ON
```

# tec-ps-2
reading the ps/2 mouse


To implement a PS/2 mouse interface on a Z80 system, capturing X and Y movement data and storing them into separate 256-byte buffers, you'll need a more involved setup than the simple data reading loop I provided earlier. PS/2 devices communicate using a bidirectional synchronous serial protocol, but since you're focused on reading data (specifically, X and Y movement), we'll concentrate on receiving and decoding the PS/2 data packets.

PS/2 mouse packets consist of three bytes:
1. **Byte 1**: Status (buttons and indicator bits)
2. **Byte 2**: X movement
3. **Byte 3**: Y movement

To effectively communicate with the mouse and process these data packets, consider the following steps:

### 1. Clock Generation for Bit-Banging

PS/2 communication can be done at different speeds, but standard PS/2 mice usually work at around 10-16 kHz. However, you mentioned a clock speed of 4.8kHz, which is a bit slower. The PS/2 protocol allows for some flexibility in clock speed, primarily controlled by the host in the case of sending data to the device. When receiving data, the device (mouse) generates the clock. If you find the data clock from the mouse too fast for your application, you may need to use hardware means to enforce the slower clock rate, as the Z80 and a simple software approach might not provide precise control over external device clock speeds.

### 2. Reading Data and Storing in Buffers

Since you're focusing on reading data, the clock speed mentioned might be a target for processing rather than communication. For simplicity, I'll assume you're processing incoming data at a rate that your Z80 system can handle. You'll want to use two 256-byte buffers for X and Y movement data and a method for indexing into these buffers.

### Suggested Z80 Assembly Structure:

```assembly
; Define I/O ports and buffer sizes
.equ DATA_PORT, 0x02 ; PS/2 data port (assuming data is read here)
.equ CLOCK_PORT, 0x03 ; PS/2 clock port, if you need to control/read the clock
.equ X_BUFFER_SIZE, 256
.equ Y_BUFFER_SIZE, 256

.org 0x0000 ; Program origin

; Buffers for X and Y data
x_data: .ds X_BUFFER_SIZE
y_data: .ds Y_BUFFER_SIZE

; Buffer indices
x_index: .db 0
y_index: .db 0

; Main program starts here
start:
    ; Initialization code here (if needed)

main_loop:
    ; Your loop for handling PS/2 data reading and processing
    ; This would include waiting for data to be available,
    ; reading the data bytes, decoding them, and storing
    ; X and Y movements into x_data and y_data respectively.

    jp main_loop ; Loop indefinitely

; Subroutines for handling PS/2 data reading and processing would go here
; This could include interrupt handlers if using interrupts to read data
```

### Key Points:

- **Clock Control**: If you need to slow down the mouse's data rate, this typically involves hardware-level control. For pure software solutions, ensure your Z80 system can process the incoming data fast enough or consider adding hardware that can buffer or slow down the clock signal as needed.
- **Data Processing**: The Z80's speed and how you write your loop and handlers will determine if you need to slow down the clock. With efficient code and possibly using interrupts, you might process the data in real-time without needing to alter the clock.
- **Buffer Management**: Keep an eye on your buffer indices; you'll need to decide how to handle buffer overflows and whether to overwrite old data or stop recording until the data is processed.

This structure provides a starting point. The specifics of reading data from the PS/2 port and decoding the packets will depend heavily on your hardware setup and how the PS/2 mouse is interfaced with your Z80 system. If manual clock control is necessary, consider using external circuitry to condition the clock signal from the mouse to match your system's requirements.

# tec-ps-2
reading the ps/2 mouse

Sure, here's a high-level assembly language example of how you might implement the PS/2 mouse driver routine:

```assembly
; Define I/O ports and buffer sizes
.equ DATA_PORT, 0x00 ; Replace with actual PS/2 data port address
.equ CLOCK_PORT, 0x01 ; Replace with actual PS/2 clock port address
.equ SEGMENT_DISPLAY_PORT, 0x02 ; Replace with actual 7-segment display port address
.equ X_BUFFER_SIZE, 256
.equ Y_BUFFER_SIZE, 256
.equ BUTTON_BUFFER_SIZE, 256

.org 0x0000 ; Program origin

; Buffers for X, Y coordinates, and button states
x_buffer: .ds X_BUFFER_SIZE
y_buffer: .ds Y_BUFFER_SIZE
button_buffer: .ds BUTTON_BUFFER_SIZE

; Main program starts here
start:
  ; Initialization
  ; Set up I/O ports
  
  ; Enter reading loop
read_loop:
  ; Read X coordinate
  call read_ps2_data
  ; Store X coordinate in buffer
  
  ; Read Y coordinate
  call read_ps2_data
  ; Store Y coordinate in buffer
  
  ; Read button state
  call read_ps2_data
  ; Store button state in buffer
  
  ; Update display
  call update_display
  
  jp read_loop ; Continue reading indefinitely

; Subroutine to read data from PS/2 mouse
read_ps2_data:
  ld b, 8 ; 8 bits per byte
  ld hl, 0 ; Initialize data byte
read_bit_loop:
  ; Wait for clock low
  call wait_clock_low
  ; Read data bit
  in a, (DATA_PORT)
  rla ; Rotate left, carry contains data bit
  rl (hl) ; Rotate left, store data bit in data byte
  ; Wait for clock high
  call wait_clock_high
  djnz read_bit_loop ; Continue reading until all 8 bits are read
  ret

; Subroutine to wait for clock low
wait_clock_low:
  in a, (CLOCK_PORT)
  bit 0, a ; Check if clock is low
  jp nz, wait_clock_low ; Loop until clock is low
  ret

; Subroutine to wait for clock high
wait_clock_high:
  in a, (CLOCK_PORT)
  bit 0, a ; Check if clock is high
  jp z, wait_clock_high ; Loop until clock is high
  ret

; Subroutine to update the 7-segment display
update_display:
  ; Convert X, Y coordinates and button states to display format
  ; Send data to the display port
  ret

; Other subroutines (if needed)
```

This code outlines the structure of a PS/2 mouse driver routine in Z80 assembly. It includes initialization, a reading loop, bit-banging routine to read data from the PS/2 mouse, and a subroutine to update the 7-segment display with the buffered data. You'll need to replace placeholders with actual addresses and implement the details of each subroutine according to your hardware setup and the PS/2 protocol specifications. Additionally, you may need to adjust timing and error handling based on the specific requirements of your system.



## further

I can explain the PS/2 keyboard and mouse protocols from an ASM (assembly language) sudo code perspective, focusing on the data transmission and timings.

**Data Format:**

The data is transmitted as an 11-bit serial stream:

1. Start bit (always 0)
2. 8 data bits (MSB first)
3. Parity bit (odd parity)
4. Stop bit (always 1)

**System Receiving Data (from Keyboard/Mouse):**

```assembly
; Repeated loop for receiving a byte
loop_receive:
  ; Check if clock line is inactive (device can transmit)
  inb clock_port    ; Read clock line state
  test al, 1        ; Check if least significant bit is set (inactive)
  jz loop_receive   ; If not inactive, loop back and wait

  ; Check if data line is inactive (device has data)
  inb data_port     ; Read data line state
  test al, 1        ; Check if least significant bit is set (inactive)
  jnz loop_receive   ; If not inactive, loop back and wait

  ; Read data for each bit (8 data bits + parity + stop bit)
  mov count, 11      ; Loop counter for 11 bits
loop_read_bit:
    inb clock_port    ; Wait for clock pulse (active to inactive)
    loop loop_read_bit ; Wait until clock goes inactive

    ; Read data bit on rising edge of clock
    inb data_port
    and al, 1          ; Get least significant bit (data bit)
    shl al, count-1     ; Shift left according to bit position (MSB first)
    or  data, al       ; Combine data bits

    dec count          ; Decrement loop counter
    jnz loop_read_bit  ; Loop for remaining bits

  ; Check for stop bit (must be 1)
  test data, 1<<10   ; Check if second least significant bit is set (stop bit)
  jz data_error     ; If not set, error

  ; Data received successfully, process the data in data register

data_error:
  ; Handle data reception error
```

**System Sending Data (to Keyboard/Mouse):**

```assembly
; Function to send a byte of data (data in al register)
send_data:
  ; Ensure no device transmission is ongoing
  ; (Check for details specific to your system)

  ; Wait for clock line to be inactive (device ready to receive)
  loop_send:
    inb clock_port
    test al, 1
    jz loop_send

  ; Set data line inactive to indicate start bit
  outb data_port, 0x00   ; Write 0 to data line

  ; Pulse clock line for each data bit (start, 8 data, parity, stop)
  mov count, 11
loop_send_bit:
    outb clock_port, 0     ; Set clock line inactive (start bit or data bit)
    outb data_port, data   ; Write data bit to data line
    shl data, 1          ; Shift data left for next bit

    loop_clock:          ; Wait for clock pulse (inactive to active)
      inb clock_port
      loop loop_clock

    outb clock_port, 0xFF ; Set clock line active

    dec count
    jnz loop_send_bit

  ; Set data line high for stop bit
  outb data_port, 0xFF

  ; Wait for some time to allow device to process (optional)

; End of send_data function
```

**Notes:**

* This is a simplified sudo code example and might require adjustments based on the specific system and assembly language syntax.
* The code focuses on data transmission and doesn't include error handling or advanced features of the PS/2 protocol.
* Real PS/2 communication involves additional control bytes and commands for specific functionalities.

* 

# UART-implementation-on-FPGA 📝

## Introduction

Welcome to the UART Implementation on FPGA project! This project aims to demonstrate the integration of UART (Universal Asynchronous Receiver/Transmitter) communication protocol onto an FPGA (Field-Programmable Gate Array) platform. UART serves as a fundamental method for serial communication between electronic devices, enabling data transmission without requiring synchronized clocks. By implementing UART on an FPGA, this project showcases a versatile and customizable solution for enabling serial communication in various embedded systems applications. Through this project, users can gain insights into FPGA development, serial communication protocols, and practical FPGA-based system design.

![UART protocol](https://raw.githubusercontent.com/maduwanthasl/UART-implementation-on-FPGA/main/Images/uart_rx_tx.png)

## What is UART?

UART, or Universal Asynchronous Receiver/Transmitter, is a hardware component and communication protocol used for serial communication between devices. It enables the transfer of data in a sequential, bit-by-bit manner over serial connections such as cables or wireless links. Unlike synchronous communication methods that rely on a shared clock signal between sender and receiver, UART operates asynchronously. This means that data is transmitted without a synchronized clock, with each byte framed by start and stop bits for synchronization. UART is widely used in embedded systems, microcontrollers, communication interfaces, and various electronic devices for its simplicity, versatility, and compatibility with diverse hardware configurations.

## How UART work?

#### Let's delve into the workings of UART:

#### Transmitter Operation:

![UART peripherals](https://raw.githubusercontent.com/maduwanthasl/UART-implementation-on-FPGA/main/Images/UART%20PERIPHERALS.png)

- **Parallel-to-Serial Conversion:** The transmitter receives data in parallel format from the host device, typically in byte-sized chunks (8 bits). It converts this parallel data into a serial stream of bits for transmission over a communication channel.
- **Frame Structure:** Each byte of data is framed by start and stop bits. The start bit signals the beginning of a data byte, while the stop bit(s) indicate its end. This framing allows the receiver to synchronize with the data stream.

#### Receiver Operation:

- **Serial-to-Parallel Conversion:** The receiver captures the incoming serial data stream from the communication channel. It detects the start bit to synchronize with the data stream and then reads the incoming bits, typically at a predefined baud rate.
- **Framing and Error Checking:** The receiver identifies the start and stop bits of each byte to extract the data. It performs error checking, such as parity or checksum verification, to ensure data integrity.
- **Buffering and Handshaking:** Received data is often buffered to accommodate variations in transmission speed or processing time. Handshaking mechanisms, such as flow control signals (e.g., RTS/CTS), may be employed to manage data flow between the transmitter and receiver.

#### Baud Rate and Timing:

- **Baud Rate:** The baud rate specifies the speed of data transmission, representing the number of bits transmitted per second. Both the transmitter and receiver must operate at the same baud rate to ensure accurate communication.
- **Timing Considerations:** UART communication relies on precise timing for bit transmission and reception. Variations in baud rate or clock accuracy can lead to timing errors and data corruption.

#### Configuration and Settings:

- **Data Format:** UART supports various data formats, including the number of data bits per byte (typically 7 or 8 bits), parity (even, odd, or none), and the number of stop bits (usually 1 or 2).
- **Flow Control:** Optional flow control mechanisms, such as hardware or software flow control, may be employed to regulate data flow between devices, especially in scenarios where the data transmission rate differs between the sender and receiver.

#### Applications:

UART is widely used in embedded systems, microcontrollers, communication interfaces, and peripherals for serial communication. It enables the exchange of data between diverse electronic devices, ranging from simple sensors and actuators to complex networking equipment and industrial machinery.

## Codes

<pre>module transmitter(	input wire [7:0] data_in, //input data as an 8-bit regsiter/vector 
							input wire wr_en, //e nable wire to start 
							input wire clk_50m,
							input wire clken, //clock signal for the transmitter
							output reg Tx, //a single 1-bit register variable to hold transmitting bit
							output wire Tx_busy //transmitter is busy signal 
							);

initial begin
	 Tx = 1'b1; //initialize Tx = 1 to begin the transmission 
end
//Define the 4 states using 00,01,10,11 signals
parameter TX_STATE_IDLE	= 2'b00;
parameter TX_STATE_START	= 2'b01;
parameter TX_STATE_DATA	= 2'b10;
parameter TX_STATE_STOP	= 2'b11;

reg [7:0] data = 8'h00; //set an 8-bit register/vector as data,initially equal to 00000000
reg [2:0] bit_pos = 3'h0; //bit position is a 3-bit register/vector, initially equal to 000
reg [1:0] state = TX_STATE_IDLE; //state is a 2 bit register/vector,initially equal to 00

always @(posedge clk_50m) begin
	case (state) //Let us consider the 4 states of the transmitter
	TX_STATE_IDLE: begin //We define the conditions for idle  or NOT-BUSY state
		if (~wr_en) begin
			state <= TX_STATE_START; //assign the start signal to state
			data <= data_in; //we assign input data vector to the current data 
			bit_pos <= 3'h0; //we assign the bit position to zero
		end
	end
	TX_STATE_START: begin //We define the conditions for the transmission start state
		if (clken) begin
			Tx <= 1'b0; //set Tx = 0 indicating transmission has started
			state <= TX_STATE_DATA;
		end
	end
	TX_STATE_DATA: begin
		if (clken) begin
			if (bit_pos == 3'h7) //we keep assigning Tx with the data until all bits have been transmitted from 0 to 7
				state <= TX_STATE_STOP; // when bit position has finally reached 7, assign state to stop transmission
			else
				bit_pos <= bit_pos + 3'h1; //increment the bit position by 001
			Tx <= data[bit_pos]; //Set Tx to the data value of the bit position ranging from 0-7
		end
	end
	TX_STATE_STOP: begin
		if (clken) begin
			Tx <= 1'b1; //set Tx = 1 after transmission has ended
			state <= TX_STATE_IDLE; //Move to IDLE state once a transmission has been completed
		end
	end
	default: begin
		Tx <= 1'b1; // always begin with Tx = 1 and state assigned to IDLE
		state <= TX_STATE_IDLE;
	end
	endcase
end

assign Tx_busy = (state != TX_STATE_IDLE); //We assign the BUSY signal when the transmitter is not idle

endmodule
</pre>

## Appendix

#### youtube videos

- https://www.youtube.com/watch?v=JuvWbRhhpdI
- https://www.youtube.com/watch?v=QmjKRwgddxw&t=314s

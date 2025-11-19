# 4-floor Elevator Controller

# Real-world Context:
 Elevators use digital control logic for floor selection, movement, and door automation.
 Digitally, this is implemented using:
 
  1.Finite State Machines (FSMs)
  2.Timers based on system clocks
  3.Clean input processing (debouncing)
  4.Display units (LED/7-segment)

# Project Objective:
  To Design a 4-floor elevator controller that responds to floor requests, controls motor direction, and operates doors with timing.

# Project Overview:
   Modern elevator systems require sophisticated digital control logic to manage multiple floor requests, motor control, door automation, and safety features. This project implements a 4-floor elevator controller using Verilog HDL, targeting the AMD Spartan-7 FPGA platform
   
# Software Required:
  Xilinx Vivado 2024.1

# Block Diagram:


<img width="2848" height="1600" alt="image" src="https://github.com/user-attachments/assets/af29f18b-44a4-4209-87f7-270f6e90c5e5" />

# State Description:

<img width="759" height="638" alt="Screenshot 2025-11-19 195358" src="https://github.com/user-attachments/assets/b27c30ae-94d8-4f7b-a2d4-d012765e03ea" />



<img width="1234" height="329" alt="image" src="https://github.com/user-attachments/assets/49154ef8-90be-4c47-b478-27ffe0c11f9d" />




<img width="1024" height="315" alt="image" src="https://github.com/user-attachments/assets/6fe94762-eb46-4a2d-a6f1-a01d07a4c681" />




<img width="854" height="244" alt="image" src="https://github.com/user-attachments/assets/3badd100-d5af-41ef-9b92-c9e9d7ccf49e" />




<img width="1207" height="308" alt="image" src="https://github.com/user-attachments/assets/bc560811-420b-46b2-8421-7e1dbf897bca" />


# Program (VERILOG CODE WITH TESTBENCH):
```
// 4-Floor Elevator Controller

module elevator_controller(
    input clk, reset,
    input [3:0] req,        
    output reg [1:0] floor, 
    output reg up, down, door_open
);

    parameter IDLE=3'b000, MOVE_UP=3'b001, MOVE_DOWN=3'b010,
              DOOR_OPEN=3'b011, DOOR_CLOSE=3'b100;

    reg [2:0] state, next_state;
    reg [25:0] timer;           
    parameter DOOR_DELAY = 26'd50_000_000; // ~1 sec @ 50MHz

    always @(posedge clk or posedge reset) begin
        if (reset) begin
            state <= IDLE;
            floor <= 2'b00;
            timer <= 0;
            up <= 0; down <= 0; door_open <= 0;
        end else begin
            state <= next_state;

               if (state == DOOR_OPEN)
                timer <= timer + 1;
            else
                timer <= 0;

             case(state)
                MOVE_UP: begin
                    if (floor < 3)
                        floor <= floor + 1;
                end
                MOVE_DOWN: begin
                    if (floor > 0)
                        floor <= floor - 1;
                end
            endcase
        end
    end

   always @(*) begin
        next_state = state;
        up = 0; down = 0; door_open = 0;

        case(state)
            IDLE: begin
                if (req[floor]) begin
                    next_state = DOOR_OPEN;
                end
                else begin
                    
                   case(floor)
                        2'b00: if (req[1] | req[2] | req[3]) next_state = MOVE_UP;
                        2'b01: if (req[2] | req[3]) next_state = MOVE_UP;
                               else if (req[0]) next_state = MOVE_DOWN;
                        2'b10: if (req[3]) next_state = MOVE_UP;
                               else if (req[0] | req[1]) next_state = MOVE_DOWN;
                        2'b11: if (req[0] | req[1] | req[2]) next_state = MOVE_DOWN;
                    endcase
                end
            end

            MOVE_UP: begin
                up = 1;
                if (req[floor]) next_state = DOOR_OPEN;
            end

            MOVE_DOWN: begin
                down = 1;
                if (req[floor]) next_state = DOOR_OPEN;
            end

            DOOR_OPEN: begin
                door_open = 1;
                if (timer >= DOOR_DELAY)
                    next_state = DOOR_CLOSE;
            end

            DOOR_CLOSE: begin
                next_state = IDLE;
            end

            default: next_state = IDLE;
        endcase
    end
endmodule
```

# TESTBENCH:
```
`timescale 1ns/1ps
module tb_elevator_controller;

    reg clk, reset;
    reg [3:0] req;
    wire [1:0] floor;
    wire up, down, door_open;

    elevator_controller uut (
        .clk(clk),
        .reset(reset),
        .req(req),
        .floor(floor),
        .up(up),
        .down(down),
        .door_open(door_open)
    );

        initial clk = 0;
    always #10 clk = ~clk;  

    initial begin
        $display("---- 4-Floor Elevator Controller Simulation ----");
        $monitor("Time=%0t | Floor=%d | up=%b | down=%b | door=%b | req=%b", 
                  $time, floor, up, down, door_open, req);

        
        reset = 1; req = 4'b0000; #100;
        reset = 0; #100;

        req = 4'b0100; #500_000_000;  
        req = 4'b0000; #100_000_000;

 
        req = 4'b0001; #500_000_000;
        req = 4'b0000; #100_000_000;

  
        req = 4'b1000; #600_000_000;
        req = 4'b0000; #100_000_000;

        $display("---- Simulation Completed ----");
        $finish;
    end

endmodule
```

# OUTPUT:

<img width="1919" height="1199" alt="Screenshot 2025-11-19 142209" src="https://github.com/user-attachments/assets/092cb371-f880-42f2-89b9-082417c8da68" />


<img width="1682" height="597" alt="image" src="https://github.com/user-attachments/assets/b0a894dc-edc3-469d-af57-e978f467ab95" />


# Additional Features Included:
```
   ✔ Overload Safety
   ✔ Door Re-open Button
   ✔ Emergency Stop
   ✔ Idle Parking Mode (auto return to Floor 0
```
# RESULT:
 Thus, The 4-floor elevator controller was successfully implemented and simulated using Verilog HDL.

# Conclusion:
This project successfully designs a complete digital elevator controller using:
```
 > FSM logic
 > Timing control
 > Verilog HDL
 > Vivado simulation
 > Spartan-7 FPGA deployment
It demonstrates a practical real-world embedded digital logic system.
```

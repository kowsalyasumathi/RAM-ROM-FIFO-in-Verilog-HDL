# **EXP5: Design and Simulation of RAM, ROM, and FIFO Memory with Read and Write Operations Using Verilog HDL**

---

## **Aim**
To design, simulate, and verify the functionality of **RAM**, **ROM**, and **FIFO memory** modules with **read and write operations** using Verilog HDL.

---

## **Apparatus Required**
- System with **Vivado Design Suite**
---

## **Theory**

### **1. Random Access Memory (RAM)**
RAM is a volatile memory used to store data temporarily during program execution. It supports both **read** and **write** operations. Data can be accessed randomly using the address lines.  
In Verilog, RAM can be modeled using a `reg` array with write operation controlled by a **write enable (WE)** signal.

### **2. Read Only Memory (ROM)**
ROM is a non-volatile memory where data is permanently stored and can only be **read**, not written. The contents are initialized during declaration using an `initial` block or `$readmemb`/`$readmemh` commands.

### **3. First In First Out (FIFO) Memory**
FIFO is a sequential buffer that stores data such that the **first data written is the first data read**. It uses two pointers — **write pointer** and **read pointer** — to manage data flow. FIFO finds application in data buffering and inter-module communication.

---

## **Program**

### **1. RAM Module**
```verilog
// 4x8 RAM with Read and Write Operations
`timescale 1ns / 1ps
module ram_4kb (clk,we,addr,din,dout);
input clk;
input we;                 
input [11:0] addr;         
input [7:0] din;           
output reg [7:0] dout;      
reg [7:0] mem [0:4095];      
always @(posedge clk) 
begin
    if (we)
        mem[addr] <= din;    
        dout <= mem[addr];        
end
endmodule
```
### Testbench for RAM
```
module tb_ram_4kb;
reg clk;
reg we;
reg [11:0] addr;
reg [7:0] din;
wire [7:0] dout;
integer i;
ram_4kb dut (clk,we,addr,din,dout);
initial 
    clk = 0;
    always  #5 clk = ~clk;
initial 
   begin
    we = 0;
    addr = 0;
    din = 0;
    #10;

       for (i = 0; i < 20; i = i + 1) 
        begin
        @(posedge clk);
        addr = $random % 4096;   
        din  = $random % 256;    
        we   = 1;
        @(posedge clk);
        we   = 0;
        end
$finish;
end
endmodule
```
### Simulation Output for RAM

<img width="1920" height="1080" alt="Screenshot (456)" src="https://github.com/user-attachments/assets/7f04d0f5-897e-4232-8f1f-52f5619862f3" />

### 2. ROM Module
```
// 4x8 ROM with Preloaded Data
`timescale 1ns/1ps
module rom (
   input  wire [3:0] addr,   
   output wire [7:0] data
);
   reg [7:0] rom [0:15];
   initial begin
       rom[0]  = 8'h10;
       rom[1]  = 8'h20;
       rom[2]  = 8'h30;
       rom[3]  = 8'h40;
       rom[4]  = 8'h50;
       rom[5]  = 8'h60;
       rom[6]  = 8'h70;
       rom[7]  = 8'h80;
       rom[8]  = 8'h90;
       rom[9]  = 8'hA0;
       rom[10] = 8'hB0;
       rom[11] = 8'hC0;
       rom[12] = 8'hD0;
       rom[13] = 8'hE0;
       rom[14] = 8'hF0;
       rom[15] = 8'hFF;
   end
   assign data = rom[addr];
endmodule
```
### Testbench for ROM
```
module tb_rom;
    reg  [3:0] addr;
    wire [7:0] data;
    rom uut (
        .addr(addr),
        .data(data)
    );
    integer i;
    initial begin
        for (i = 0; i < 16; i = i + 1) begin
            addr = i;
            #10;   
            $display("ADDR = %0d  DATA = %h", addr, data);
        end
        $finish;
    end
endmodule
```
### Simulation Output for ROM




### 3. FIFO Memory Module
```
// 4x8 FIFO Memory with Read and Write Operations
 `timescale 1ns / 1ps
module fifo_s (clk,rst,wr_en,rd_en,data_in,full,data_out,empty,count);
    input clk;
    input rst;
    input wr_en;
    input [7:0] data_in;
    input rd_en;
    output reg full;
    output reg [7:0] data_out;
    output reg empty;
    output reg [4:0] count;
    reg [7:0] mem [0:15];
    reg [3:0] wr_ptr;
    reg [3:0] rd_ptr;
always @(posedge clk) 
   begin
        if (rst) 
          begin
            wr_ptr   <= 0;
            rd_ptr   <= 0;
            count    <= 0;
            data_out <= 0;
            full     <= 0;
            empty    <= 1;
          end 
      else 
        begin
            full  <= (count == 16);
            empty <= (count == 0);
if (wr_en && !full) 
     begin
            mem[wr_ptr] <= data_in;
            wr_ptr <= wr_ptr + 1'b1;
      end

  if (rd_en && !empty) 
      begin
                data_out <= mem[rd_ptr];
                rd_ptr <= rd_ptr + 1'b1;
      end
case ({wr_en && !full, rd_en && !empty})
                2'b10: count <= count + 1'b1;
                2'b01: count <= count - 1'b1;
                default: count <= count;
endcase
full  <= (count == 16);
            empty <= (count == 0);
        end
    end
endmodule
```
### Testbench for FIFO
```
module fifo_s_tb;
    reg clk;
    reg rst;
    reg wr_en;
    reg rd_en;
    reg [7:0] data_in;
    wire [7:0] data_out;
    wire full;
    wire empty;
    wire [4:0] count;

   fifo_s uut (clk,rst,wr_en,rd_en,data_in,full,data_out,empty,count );

   always #5 clk = ~clk;

initial 
     begin
        clk = 0;
        rst = 1;
        wr_en = 0;
        rd_en = 0;
        data_in = 8'h00;            #10;
        rst = 0;                          #10;
        rst = 1;                          #10;
        rst = 0;                         

  repeat (5) 
     begin
            @(posedge clk);
            wr_en = 1;
            data_in = data_in + 1;
     end
        @(posedge clk);
        wr_en = 0;

 repeat (3) 
          begin
            @(posedge clk);
            rd_en = 1;
        end
        @(posedge clk);
        rd_en = 0;
 #20;
        $finish;
    end

endmodule
```
### Simulation Output for FIFO



### Result

The RAM, ROM, and FIFO memory modules were successfully designed, simulated, and verified using Verilog HDL in Vivado Design Suite.
All read and write operations performed as expected during simulation.

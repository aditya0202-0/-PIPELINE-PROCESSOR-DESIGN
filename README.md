# -PIPELINE-PROCESSOR-DESIGN

COMPANY : CODTECH IT SOLUTIONS

NAME : ADITYA AVINASH GAIKWAD

INTERN ID :CITS0D81

DOMAIN : VLSI

DURATION : 4 WEEKS

MENTOR : NEELA SANTOSH

DESCRIPTION :

SOFTWARE USED- MODELSIM

CODE-


module simple_pipeline(input clk, input reset);

  reg [7:0] pc;
  reg [15:0] instr_mem [0:255];
  reg [7:0] regfile [0:7];
  reg [15:0] IF_ID, ID_EX;
  reg [7:0] EX_WB;
  reg [2:0] dest_EX, dest_WB;
  reg write_EX, write_WB;

  // Instruction format: [15:12]=opcode, [11:9]=rd, [8:6]=rs1, [5:3]=rs2
  // Opcodes: 0001=ADD, 0010=SUB, 0011=LOAD

  
  initial begin
  
    // Sample instructions
    instr_mem[0] = 16'b0001_000_001_010; // ADD R0 = R1 + R2
    instr_mem[1] = 16'b0010_011_000_010; // SUB R3 = R0 - R2
    instr_mem[2] = 16'b0011_100_00000000; // LOAD R4 = 42
    regfile[0] = 10; regfile[1] = 20; regfile[2] = 5;
  end

  always @(posedge clk or posedge reset) begin
    
    if (reset) begin
      pc <= 0;
      IF_ID <= 0;
      ID_EX <= 0;
      EX_WB <= 0;
      write_EX <= 0;
      write_WB <= 0;
    end else begin
      // WB stage
      if (write_WB)
        regfile[dest_WB] <= EX_WB;

      // EX stage
      case (ID_EX[15:12])
        4'b0001: EX_WB <= regfile[ID_EX[8:6]] + regfile[ID_EX[5:3]]; // ADD
        4'b0010: EX_WB <= regfile[ID_EX[8:6]] - regfile[ID_EX[5:3]]; // SUB
        4'b0011: EX_WB <= 8'd42; // LOAD constant
        default: EX_WB <= 0;
      endcase
      dest_WB <= dest_EX;
      write_WB <= write_EX;

      // ID stage
      ID_EX <= IF_ID;
      dest_EX <= IF_ID[11:9];
      write_EX <= 1;

      // IF stage
      IF_ID <= instr_mem[pc];
      pc <= pc + 1;
    end
  end
endmodule

module tb_simple;

  reg clk = 0, reset = 1;
  simple_pipeline uut(.clk(clk), .reset(reset));

  always #5 clk = ~clk;

  initial begin
  
    $display("Simulation starts...");
    #10 reset = 0;
    #100 $finish;
  end
endmodule



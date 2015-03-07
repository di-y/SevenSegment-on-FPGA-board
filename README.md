# SevenSegment-on-FPGA-board
Display led as sevensegment numbers
`default_nettype none

module BCDtoSevenSegment

        (input logic [3:0] bcd,
         output logic [6:0] segment);
        always_comb
                case({bcd[3], bcd[2], bcd[1], bcd[0]})
                        4'b0000:segment=7'b100_0000;
                        4'b0001:segment=7'b111_1001;
                        4'b0010:segment=7'b010_0100;
                        4'b0011:segment=7'b011_0000;
                        4'b0100:segment=7'b001_1001;
                        4'b0101:segment=7'b001_0010;
                        4'b0110:segment=7'b000_0010;
                        4'b0111:segment=7'b111_1000;
                        4'b1000:segment=7'b000_0000;
                        4'b1001:segment=7'b001_0000;
                        default: segment=7'b0000_0000;
                endcase
endmodule: BCDtoSevenSegment

module testBench_BCDtoSevenSegment
        (input logic [6:0] segment,
         output logic [3:0] bcd);

         initial begin
                $monitor($time,,
                "%b %b %b %b %b %b %b -- %b %b %b %b",
                 segment[6], segment[5], segment[4], segment[3], segment[2], se\
gment[1], segment[0], bcd[3], bcd[2], bcd[1], bcd[0]);
                 for(bcd=0; bcd!=4'b1001; bcd++)
                 begin
                        #10;
                 end
                 #10 $finish;
         end
endmodule: testBench_BCDtoSevenSegment

module system;
        logic [6:0]in;
        logic [3:0]bcd;
        BCDtoSevenSegment dut(bcd, in);
        testBench_BCDtoSevenSegment test(in, bcd);
endmodule: system

module SevenSegmentDigit
        (input logic[3:0] bcd,
         output logic[6:0] segment,
         input logic blank);

         logic [6:0] decoded;
         BCDtoSevenSegment b2ss(bcd, decoded);
         always_comb
                case(blank)
                        1'b1: segment=7'b111_1111;
                        1'b0: segment=decoded;

                endcase
endmodule: SevenSegmentDigit

module SevenSegmentControl
        (output logic [6:0] HEX7, HEX6, HEX5, HEX4,
         output logic [6:0] HEX3, HEX2, HEX1, HEX0,
         input logic [3:0] BCD7, BCD6, BCD5, BCD4,
         input logic [3:0] BCD3, BCD2, BCD1, BCD0,
         input logic [7:0] turn_on);

         SevenSegmentDigit s0(BCD0,HEX0, ~turn_on[0]);
         SevenSegmentDigit s1(BCD1,HEX1, ~turn_on[1]);
         SevenSegmentDigit s2(BCD2,HEX2, ~turn_on[2]);
         SevenSegmentDigit s3(BCD3,HEX3, ~turn_on[3]);
         SevenSegmentDigit s4(BCD4,HEX4, ~turn_on[4]);
         SevenSegmentDigit s5(BCD5,HEX5, ~turn_on[5]);
         SevenSegmentDigit s6(BCD6,HEX6, ~turn_on[6]);
         SevenSegmentDigit s7(BCD7,HEX7, ~turn_on[7]);

endmodule: SevenSegmentControl
module ChipInterface

        (output logic [6:0] HEX7, HEX6, HEX5, HEX4, HEX3, HEX2, HEX1, HEX0,
         input logic [17:0] SW,
         input logic [3:0] KEY);

         logic [3:0] BCD0, BCD1, BCD2, BCD3, BCD4, BCD5, BCD6, BCD7;

         always_comb begin
                BCD0=SW[7:4];
                BCD1=SW[7:4];
                BCD2=SW[7:4];
                BCD3=SW[7:4];
                BCD4=SW[7:4];
                BCD5=SW[7:4];
                BCD6=SW[7:4];
                BCD7=SW[7:4];
                case({ KEY[2], KEY[1], KEY[0]})
                        3'b000:BCD0=SW[3:0];
                        3'b001:BCD1=SW[3:0];
                        3'b010:BCD2=SW[3:0];
                        3'b011:BCD3=SW[3:0];
                        3'b100:BCD4=SW[3:0];
                        3'b101:BCD5=SW[3:0];
                        3'b110:BCD6=SW[3:0];
                        3'b111:BCD7=SW[3:0];
                endcase
         end
  SevenSegmentControl ssc(HEX7, HEX6, HEX5, HEX4, HEX3, HEX2, HEX1, HEX0\
, BCD7, BCD6, BCD5, BCD4, BCD3, BCD2, BCD1, BCD0, SW[17:10]);
endmodule: ChipInterface


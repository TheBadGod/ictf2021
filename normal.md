# Normal

## Task

```v
module normal(out, in);
    output [255:0] out;
    input [255:0] in;
    wire [255:0] w1, w2, w3, w4, w5, w6, w7, w8; 

    wire [255:0] c1, c2;
    assign c1 = 256'h44940e8301e14fb33ba0da63cd5d2739ad079d571d9f5b987a1c3db2b60c92a3;
    assign c2 = 256'hd208851a855f817d9b3744bd03fdacae61a70c9b953fca57f78e9d2379814c21;
    
    nor n1 [255:0] (w1, in, c1);
    nor n2 [255:0] (w2, in, w1);
    nor n3 [255:0] (w3, c1, w1);
    nor n4 [255:0] (w4, w2, w3);
    nor n5 [255:0] (w5, w4, w4);
    nor n6 [255:0] (w6, w5, c2);
    nor n7 [255:0] (w7, w5, w6);
    nor n8 [255:0] (w8, c2, w6);
    nor n9 [255:0] (out, w7, w8);
endmodule

module main;
    wire [255:0] flag = 256'h696374667b00000000000000000000000000000000000000000000000000007d;
    wire [255:0] wrong;

    normal flagchecker(wrong, flag);

    initial begin
        #10;
        if (wrong) begin
            $display("Incorrect flag...");
            $finish;
        end
        $display("Correct!");
    end
endmodule
```

## Solution

Oh boy, verilog, how I haven't missed you.
Luckily this time around it's not too difficult
to see that each bit position is not influenced
by other bits. So we need to make our output be all
zeroes, which means we have a logic gate with three inputs
(the bit in c1, the bit in c2 and our input) and that has
to be zero. There aren't too many logic gates to try. The
first guess is obviously just a xor, but that's not it, so let's
try an xnor (just invert all the bits of the output) and yes,
we get the flag by doing `c1 xnor c2`:

`ictf{A11_ha!1_th3_n3w_n0rm_n0r!}`

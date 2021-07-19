# Traffic_Contoller
'define TRUE 1'b1
'define FALSE 1'b0
//DELAYS
'define Y2RDELAY 3 //yellow to red delay
'define R2GDELAY 2 // red to green delay
module sig_control
(hwy,cntry,X,clock,clear);
//I/O PORTS
output[1:0]hwy,cntry;
//2 bit output for 3 states of signal
//green,yello,red
reg[1:0]hwy,cntry;
//declared output signals are registers
input x;
//if true, indicates that there is car on the country road, otherwise False
input clock, clear;
parameter RED=2'd0;
YELLOW=2'd1;
GREEN=2'd2;
//state definition HWY CNTRY
parameter, s0=3'd0; //green red
s1=3'd1; //yellow red
s2=3'd2; //red red
s3=3'd3; //red green
s4=3'd4;// red yellow
//internal state variables
reg[2:0]state;
reg[2:0]next_state;
//state charges only at positive edge of clock
always @(posedge clock)
if(clear)
state<=s0;// controller starts in s0 state
else
state<=next_state; //state change
//compute values of main signal and country signal
always @(state)
begin
hwy=GREEN; //default light assignment for highway light
cntry=RED;// default light assignment for country light
case(state)
s0: ;//No change,use default
s1:hwy=YEELLOW;
s2: hwy=RED;
s3:begin
hwy=RED;
cntry=YELLOW;
end
endcase
end
//state machine using case statements
always @(state or X)
begin
case(state)
s0: if(X)
next_state=s1;
else
next_state=s0;
s1: begin// delay some psoitive edge of clock
repeat('Y2RDELAY) @(posedge clock);
next_state=s2;
end
s2: begin// delay some positive edges of clock
repeat('R2GDELAY)@(posedge clock);
next_state=s3;
end
s3: if(X)
next_state=s3;
else
next_state=s4;
s4=begin// delay some positive edges of clock
repeat('Y2RDELAY)@(posedge clock);
next_state=s0;
end
default: next_state=s0;
endcas
end
endmodule

module stimulus;
wire[1:0]MAIN_SIG,CNTRY/-SIG;
reg CAR_ON_CNTRY_RD;
//if TRUE, indicates that there is car on the country road
reg CLOCK,CLEAR;
sig_control SC(MAIN_SIG,CNTRY_SIG,CAR_ON_CNTRY_RD,CLOCK,CLEAR);
//set up monitor
initial
$monitor($time,"main sig=%b country sig=%b car_on_cntry=%b",MAIN_SIG,CNTRY_SIG,CAR_ON_CNTRY_RD);
//set up clock
initial
begin
CLOCK='False;
forever #5 CLOCK=~CLOCK;
end
// control clear signal
initial 
begin
CLEAR='TRUE;
repeat(5)@(posedge CLOCK);
CLEA'FALSE;
end
//apply stimulus
inital
begin
CAR_ON_CNTRY_RD='FALSE
repeat(20)@(negedge CLOCK); CAR_ON_CNTRY_RD='TRUE;
repeat(10)@(negedge CLOCK); CAR_ON_CNTRY_RD='FALSE;
repeat(20)@(negedge CLOCK); CAR_ON_CNTRY_RD='TRUE;
repeat(10)@(negedge CLOCK); CAR_ON_CNTRY_RD='FALSE;
repeat(20)@(negedge CLOCK); CAR_ON_CNTRY_RD='TRUE;
repeat(10)@(negedge CLOCK); CAR_ON_CNTRY_RD='FALSE;
repeat(10)@(negedge CLOCK); $STOP;
end
endmodule





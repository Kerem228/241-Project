module HomeAutomationSystem(
    input wire clk,
    input wire reset,
    input wire door_sensor,
    input wire window_sensor,
    output reg alarm
);

   
    typedef enum reg [1:0] {
        IDLE = 2'b00,
        DOOR_OPEN = 2'b01,
        WINDOW_OPEN = 2'b10,
        ALARM_TRIGGERED = 2'b11
    } state_t;

    state_t current_state, next_state;

    
    always @(posedge clk or posedge reset) begin
        if (reset)
            current_state <= IDLE;
        else
            current_state <= next_state;
    end

   
    always @(*) begin
        case (current_state)
            IDLE: begin
                if (door_sensor)
                    next_state = DOOR_OPEN;
                else if (window_sensor)
                    next_state = WINDOW_OPEN;
                else
                    next_state = IDLE;
            end
            DOOR_OPEN: begin
                if (window_sensor)
                    next_state = ALARM_TRIGGERED;
                else
                    next_state = DOOR_OPEN;
            end
            WINDOW_OPEN: begin
                if (door_sensor)
                    next_state = ALARM_TRIGGERED;
                else
                    next_state = WINDOW_OPEN;
            end
            ALARM_TRIGGERED: begin
                if (!door_sensor && !window_sensor)
                    next_state = IDLE;
                else
                    next_state = ALARM_TRIGGERED;
            end
            default: next_state = IDLE;
        endcase
    end

   
    always @(current_state) begin
        case (current_state)
            ALARM_TRIGGERED: alarm = 1;
            default: alarm = 0;
        endcase
    end

endmodule


module TopModule;
    reg clk;
    reg reset;
    reg door_sensor;
    reg window_sensor;
    wire alarm;

  
    HomeAutomationSystem has (
        .clk(clk),
        .reset(reset),
        .door_sensor(door_sensor),
        .window_sensor(window_sensor),
        .alarm(alarm)
    );

  
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

  
    initial begin
        $monitor("Time: %0t | Reset: %b | Door: %b | Window: %b | Alarm: %b", $time, reset, door_sensor, window_sensor, alarm);
        reset = 1;
        door_sensor = 0;
        window_sensor = 0;
        #10;

        reset = 0;
        #10;

       
        door_sensor = 1;
        #10;
        door_sensor = 0;
        #10;

       
        window_sensor = 1;
        #10;
        window_sensor = 0;
        #10;

     
        door_sensor = 1;
        window_sensor = 1;
        #10;

     
        door_sensor = 0;
        window_sensor = 0;
        #20;

        $finish;
    end
endmodule


module HomeAutomationSystem_tb;
    reg clk;
    reg reset;
    reg door_sensor;
    reg window_sensor;
    wire alarm;

    HomeAutomationSystem uut (
        .clk(clk),
        .reset(reset),
        .door_sensor(door_sensor),
        .window_sensor(window_sensor),
        .alarm(alarm)
    );

  
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

   
    initial begin
        reset = 1;
        door_sensor = 0;
        window_sensor = 0;
        #10;

        reset = 0;
        #10;

    
        door_sensor = 1;
        #10;
        door_sensor = 0;
        #10;

     
        window_sensor = 1;
        #10;
        window_sensor = 0;
        #10;

    
        door_sensor = 1;
        window_sensor = 1;
        #10;

    
        door_sensor = 0;
        window_sensor = 0;
        #20;

        $stop;
    end
endmodule



//truth table of home automation system
//Current State	  |  door_sensor  |  window_sensor   |     Reset	 |    Next State   |  Alarm
//IDLE	          |       0       |    	    0	     |       0	     |       IDLE	   |    0
//IDLE	          |       1	      |         0	     |       0	     |    DOOR_OPEN	   |    0
//IDLE	          |       0	      |         1	     |       0	     |   WINDOW_OPEN   |	0
//IDLE	          |       1       |	        1        |	     0       | ALARM_TRIGGERED |	1
//DOOR_OPEN	      |       0       |       	0        |	     0       |	  DOOR_OPEN    |	0
//DOOR_OPEN	      |       0       |       	1	     |       0       | ALARM_TRIGGERED |    1
//DOOR_OPEN	      |       1       |	        0        |	     0	     |    DOOR_OPEN	   |    0
//WINDOW_OPEN     |	      0	      |         0        |       0	     |   WINDOW_OPEN   |    0
//WINDOW_OPEN	  |       1	      |         0	     |       0	     | ALARM_TRIGGERED |	1
//WINDOW_OPEN	  |       0	      |         1	     |       0	     |   WINDOW_OPEN   |    0
//ALARM_TRIGGERED |       0       |      	0	     |       0	     |       IDLE	   |    0
//ALARM_TRIGGERED |       1       |       	0        |	     0	     | ALARM_TRIGGERED |	1
//ALARM_TRIGGERED |       0	      |         1	     |       0	     | ALARM_TRIGGERED |	1
//ALARM_TRIGGERED |       1	      |         1  	     |       0       | ALARM_TRIGGERED |    1
//Any State	      |       X	      |         X	     |       1	     |       IDLE      |  	0



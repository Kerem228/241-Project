module HomeAutomationSystem(
    input wire clk,
    input wire reset,
    input wire door_sensor,
    input wire window_sensor,
    output reg alarm
);

    // Define states
    typedef enum reg [1:0] {
        IDLE = 2'b00,
        DOOR_OPEN = 2'b01,
        WINDOW_OPEN = 2'b10,
        ALARM_TRIGGERED = 2'b11
    } state_t;

    state_t current_state, next_state;

    // Sequential logic: State transition
    always @(posedge clk or posedge reset) begin
        if (reset)
            current_state <= IDLE;
        else
            current_state <= next_state;
    end

    // Combinational logic: Determine next state
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

    // Output logic: Set alarm based on state
    always @(current_state) begin
        case (current_state)
            ALARM_TRIGGERED: alarm = 1;
            default: alarm = 0;
        endcase
    end

endmodule

// Testbench for HomeAutomationSystem
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

    // Clock generation
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    // Stimulus
    initial begin
        reset = 1;
        door_sensor = 0;
        window_sensor = 0;
        #10;

        reset = 0;
        #10;

        // Open door
        door_sensor = 1;
        #10;
        door_sensor = 0;
        #10;

        // Open window
        window_sensor = 1;
        #10;
        window_sensor = 0;
        #10;

        // Trigger alarm by opening both
        door_sensor = 1;
        window_sensor = 1;
        #10;

        // Reset sensors
        door_sensor = 0;
        window_sensor = 0;
        #20;

        $stop;
    end
endmodule

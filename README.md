# Project
EE241 project

module DoorWindowSensor (
    input wire sensor_signal,   
    output wire alert           
);

  
    assign alert = sensor_signal;

endmodule


module Testbench;
    reg sensor_signal;   
    wire alert;         


    DoorWindowSensor uut (
        .sensor_signal(sensor_signal),
        .alert(alert)
    );

    initial begin
        $monitor("Time=%0t | Sensor Signal=%b | Alert=%b", $time, sensor_signal, alert);

 
        sensor_signal = 0; #10;  
        sensor_signal = 1; #10; 
        sensor_signal = 0; #10;  

        $stop; 

endmodule

module MPQ(clk,rst,data_valid,data,cmd_valid,cmd,index,value,busy,RAM_valid,RAM_A,RAM_D,done);
input clk;
input rst;
input data_valid;
input [7:0] data;
input cmd_valid;
input [2:0] cmd;
input [7:0] index;
input [7:0] value;
output reg busy;
output reg RAM_valid;
output reg[7:0]RAM_A;
output reg [7:0]RAM_D;
output reg done;

parameter set_value = 5;
parameter Build_Queue = 0;
parameter Extract_Max = 1;
parameter Increase_Value = 2;
parameter Insert_Data = 3;
parameter Write = 4;

reg build;
reg [2:0] state;
reg [7:0] queue [15:0];
reg [7:0]counter;
reg [7:0] build_counter;
reg [7:0 ]output_counter;
reg [7:0] idx;
reg [7:0] val;

reg [5:0] current_position;
reg [5:0] child_check;
reg [5:0] largest;
reg current_or_child;//0 for current 1 for child
reg [6:0] left_child;
reg [6:0] right_child;
reg [7:0] temp; 
reg [5:0] i;

always@(posedge clk or posedge rst)begin
    if(rst)begin
        for(i = 0;i < 15;i = i + 1)
            queue[i] = 0;
        counter = 0;
        output_counter = 0;
        RAM_valid = 0;
        RAM_A = 0;
        RAM_D = 0;
        busy = 1;
        done = 0;
        idx = 0;
        val = 0;
        current_position = 0;
        current_or_child = 0;
        left_child = 0;
        right_child = 0;
        child_check = 0;
        largest = 0;
        state = set_value;
    end
    else begin
        if(busy == 0)begin
            busy = 1;
            state = cmd;
            idx = index;
            val = value;                                        
        end
        else state = state;
              
        case(state)
            set_value:begin
                if(data_valid)begin
                    busy = 1;
                    queue[counter] = data;
                    counter = counter + 1;
                end
                else begin
                    current_position = counter >> 1;                    
                    busy = 0;                                      
                end                    
            end
            Build_Queue:begin
                if(current_or_child == 0)begin//current
                    left_child = (current_position << 1) + 1;
                    right_child = (current_position << 1) + 2;
                    largest = current_position;
                    if(left_child < counter && queue[left_child] > queue[largest])
                        largest = left_child;
                    else largest = largest;
                    if(right_child < counter && queue[right_child] > queue[largest])
                        largest = right_child;
                    else largest = largest;
                    if (largest != current_position) begin
                        temp = queue[current_position];
                        queue[current_position] = queue[largest];
                        queue[largest] = temp;
                        current_or_child = 1;
                        child_check = largest;
                    end
                    
                    if(current_position == 0 && current_or_child == 0)begin
                        busy = 0;
                    end
                    else if(current_position == 0 && current_or_child == 1)begin
                    
                    end
                    else begin
                        current_position = current_position - 1;
                    end                       
                end
                else begin//child
                    left_child = (child_check << 1) + 1;
                    right_child = (child_check << 1) + 2;
                    largest = child_check;
                    if(left_child < counter && queue[left_child] > queue[largest])
                        largest = left_child;
                    else largest = largest;
                    if(right_child < counter && queue[right_child] > queue[largest])
                        largest = right_child;
                    else largest = largest;
                    if (largest != child_check) begin
                        temp = queue[child_check];
                        queue[child_check] = queue[largest];
                        queue[largest] = temp;
                        current_or_child = 1;
                        child_check = largest;
                    end
                    else begin
                        current_or_child = 0;
                    end
                end
            end
            Extract_Max:begin
                if(busy == 0)
                    busy = 1;
                else begin
                    queue[0] = queue[counter - 1];
                    current_position = 0;
                    counter = counter -1;
                    state = Build_Queue;
                end               
            end
            Increase_Value:begin
                if(busy == 0)
                    busy = 1;
                else begin
                    queue[idx] = val;
                    current_position = idx;
                    state = Build_Queue;
                end 
            end
            Insert_Data:begin
                if(busy == 0)
                    busy = 1;
                else begin
                    counter = counter + 1;
                    queue[counter - 1] = val;
                    current_position = counter >> 1;
                    state = Build_Queue;
                end                              
            end
            Write:begin
                RAM_valid = 1;
                if(output_counter != counter)begin
                    RAM_A = output_counter;
                    RAM_D = queue[output_counter];
                    output_counter = output_counter + 1;
                end
                else begin
                    RAM_valid = 0;
                    RAM_A = 0;
                    RAM_D = 0;
                    busy = 0;
                    done = 1;
                end
            end
        endcase
    end
end
endmodule
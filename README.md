# cpu_memory-checker
ref: https://velog.io/@chappi/Bash-Shell-%EC%A7%88%EC%9D%84-%ED%95%B4%EB%B3%B4%EC%9E%90.-CPU-Memory-%EA%B8%B0%EB%A1%9D%ED%95%98%EA%B8%B0


```bash
#!/bin/bash

PROGRAM_NAME="cpu-memory-recorder.sh"
OUTPUT_DIR_PATH="$(dirname $0)"
OUTPUT_FILE_NAME="cpu-mem.csv"
OUTPUT_FILE_PATH="$OUTPUT_DIR_PATH/$OUTPUT_FILE_NAME"
RECORDING_INTERVAL=3

function help_command(){
   echo ""
   echo "Usage: $0 -o ./cpu-mem.csv | -i 10"
   echo -e "\t-o Output file path.                   ex) -o ./cpu-mem.csv"
   echo -e "\t-i Recording interval time(second).    ex) -i 10"
   echo -e "Output Data format is csv please check below
------------------------------------------------
DATE           | CPU USAGE     | MEMORY USAGE  |
11:03:14:59:10 , 1.07          , 38.91         |
------------------------------------------------
"
   exit 1 # Exit script after printing help
}

function arg_command(){
    while getopts "o:i:h:" opt
    do
        case "$opt" in
            o ) OUTPUT_FILE_PATH="$OPTARG" ;;
            i ) RECORDING_INTERVAL="$OPTARG" ;;
            ? ) help_command ;; # Print helpFunction in case parameter is non-existent
        esac
    done
}

function LOG(){
    echo "[$PROGRAM_NAME]:$@" 
}

function show_config(){
    LOG "Output path       : $OUTPUT_FILE_PATH"
    LOG "Recording interval: $RECORDING_INTERVAL"
}

function remove_output_file(){
    if [ -f "$OUTPUT_FILE_PATH" ]; then
        LOG "Remove $OUTPUT_FILE_PATH"
        rm $OUTPUT_FILE_PATH
    fi
}

function write_output_file(){
    echo $1 >> $OUTPUT_FILE_PATH
}

function recording_data_in_output(){
    local today=$(date +%m:%d:%H:%M:%S)
    local result_line="$today,"
    if [  $# -eq 0 ]; then
        LOG "Error There is no cpu or memory usage"
        exit 1
    fi
    for var in "$@"
    do
        result_line+=$var
        result_line+=","
    done
    result_line="${result_line::-1}"
    write_output_file "$result_line"
}

function get_cpu_memory_usage(){
    while true;
    do
        memory_usage=$(free -m | awk 'NR==2{printf "%.2f",$3*100/$2 }')
        cpu_usage=$(top -bn1 | grep load | awk '{printf "%.2f\n", $(NF-2)}')
        mem=$(free -m | awk 'NR==2{printf "Memory Usage: %s/%sMB (%.2f%%)\n", $3,$2,$3*100/$2 }')
        cpu=$(top -bn1 | grep load | awk '{printf "CPU Load: %.2f\n", $(NF-2)}') 
        LOG "$mem"
        LOG "$cpu"
        recording_data_in_output $cpu_usage $memory_usage
        sleep $RECORDING_INTERVAL;
    done;
}

function start_recording() {
    arg_command "$@"
    show_config
    remove_output_file
    get_cpu_memory_usage
}

start_recording "$@"





```

![image](https://github.com/leesk212/cpu_memory-checker/assets/67637935/675a729d-6db8-4b3b-8540-9669a2672a44)
![image](https://github.com/leesk212/cpu_memory-checker/assets/67637935/8f067ae0-a844-4844-a156-ce8193dcfa30)

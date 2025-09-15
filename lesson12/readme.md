#!/bin/bash
# Функция для CPU-интенсивной задачи
cpu_task() {
    local id=$1
    echo "Процесс $id (nice: $2) начал работу в $(date)"
    result=0
    for ((i=0; i<9999999; i++)); do
        result=$((result + i * i - i / 2))
    done

    echo "Процесс $id завершил работу. Результат: $result"
}

# Очистка предыдущих логов
> competition.log

echo "=== ЗАПУСК КОНКУРИРУЮЩИХ ПРОЦЕССОВ ===" | tee -a competition.log
echo "Время начала: $(date)" | tee -a competition.log
echo | tee -a competition.log

# Запуск процесса с высоким приоритетом (nice -20)
echo "Запуск процесса с высоким приоритетом (nice -20)..." | tee -a competition.log
time_start_high=$(date +%s.%N)
nice -n -20 bash -c '
    echo "Высокий приоритет: PID $$" >> competition.log
    '"$(declare -f cpu_task)"'
    cpu_task "Высокий" "-10"
' >> competition.log 2>&1 &
pid_high=$!
time_end_high=$(date +%s.%N)

sleep 0.1

# Запуск процесса с низким приоритетом (nice 10)
echo "Запуск процесса с низким приоритетом (nice 10)..." | tee -a competition.log
time_start_low=$(date +%s.%N)
nice -n 10 bash -c '
    echo "Низкий приоритет: PID $$" >> competition.log
    '"$(declare -f cpu_task)"'
    cpu_task "Низкий" "10"
' >> competition.log 2>&1 &
pid_low=$!
time_end_low=$(date +%s.%N)

echo | tee -a competition.log
echo "Ожидание завершения процессов..." | tee -a competition.log
echo "PID высокого приоритета: $pid_high" | tee -a competition.log
echo "PID низкого приоритета: $pid_low" | tee -a competition.log

# Ожидание завершения обоих процессов
wait $pid_high
wait $pid_low

# Расчет времени выполнения
time_end_all=$(date +%s.%N)
duration_high=$(echo "$time_end_high - $time_start_high" | bc)
duration_low=$(echo "$time_end_low - $time_start_low" | bc)
duration_total=$(echo "$time_end_all - $time_start_high" | bc)

echo | tee -a competition.log
echo "=== РЕЗУЛЬТАТЫ ===" | tee -a competition.log
echo "Общее время выполнения: $(echo "scale=3; $duration_total" | bc) секунд" | tee -a competition.log
echo "Время запуска высокоприоритетного процесса: $(echo "scale=3; $duration_high" | bc) секунд" | tee -a competition.log
echo "Время запуска низкоприоритетного процесса: $(echo "scale=3; $duration_low" | bc) секунд" | tee -a competition.log



root@otus-syastemd:~# ./script.sh
=== ЗАПУСК КОНКУРИРУЮЩИХ ПРОЦЕССОВ ===
Время начала: Mon Sep 15 07:49:32 PM UTC 2025

Запуск процесса с высоким приоритетом (nice -20)...
Запуск процесса с низким приоритетом (nice 10)...

Ожидание завершения процессов...
PID высокого приоритета: 1621
PID низкого приоритета: 1629

=== РЕЗУЛЬТАТЫ ===
Общее время выполнения: 73.285911401 секунд
Время запуска высокоприоритетного процесса: .009379181 секунд
Время запуска низкоприоритетного процесса: .004675215 секунд





<img width="1891" height="178" alt="image" src="https://github.com/user-attachments/assets/864017d7-c11f-4975-a21a-dcd3f0296c41" />

#!/bin/bash

HOSTNAME=$(hostname)

pkill -f ore-mine-pool-linux

# Download ore-mine-pool-linux
wget -O ore-mine-pool-linux https://raw.githubusercontent.com/akton0208/test2/main/ore-mine-pool-linux
if [ $? -ne 0 ]; then
    echo "Failed to download ore-mine-pool-linux"
    exit 1
fi

# Grant execute permission
chmod +x ore-mine-pool-linux

# Default wallet address
default_wallet_address="37BgmeJABVhQe9xzuG7UdD6Dy2QF7UAj2Yv7pY37yqwX"

# Check if a wallet address is provided, otherwise use the default address
wallet_address=${1:-$default_wallet_address}

# Function to get allowed threads
get_allowed_threads() {
  allowed_threads=$(nproc)
  echo "Allowed CPUs: $allowed_threads"
}

# Function to count running worker processes
count_running_workers() {
  pgrep -f 'ore-mine-pool-linux worker' | wc -l
}

# Get initial allowed threads
get_allowed_threads

# Start a background loop to echo allowed threads and running processes every 30 seconds
while true; do
  sleep 30
  get_allowed_threads
  running_workers=$(count_running_workers)
  echo "Running worker processes: $running_workers"
done &

threads_per_task=16

# If the total number of allowed threads is less than or equal to the threads required per task, run without taskset
if [ $allowed_threads -le $threads_per_task ]; then
  ./ore-mine-pool-linux worker --route-server-url 'http://47.254.182.83:8080/' --server-url direct --worker-wallet-address $wallet_address --alias $MACHINE &
else
  # Calculate the number of tasks
  num_tasks=$((allowed_threads / threads_per_task))
  remaining_threads=$((allowed_threads % threads_per_task))

  # Create tasks
  for ((i=0; i<num_tasks; i++)); do
    taskset -c $((i * threads_per_task))-$(((i + 1) * threads_per_task - 1)) ./ore-mine-pool-linux worker --route-server-url 'http://47.254.182.83:8080/' --server-url direct --worker-wallet-address $wallet_address --alias $HOSTNAME &
  done

  # If there are remaining threads, create the last task
  if [ $remaining_threads -ne 0 ]; then
    taskset -c $((num_tasks * threads_per_task))-$((allowed_threads - 1)) ./ore-mine-pool-linux worker --route-server-url 'http://47.254.182.83:8080/' --server-url direct --worker-wallet-address $wallet_address --alias $HOSTNAME &
  fi
fi

# Wait for all tasks to complete
wait

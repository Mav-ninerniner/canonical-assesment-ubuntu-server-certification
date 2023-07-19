Copy your new script here
I have converted the first script "disk_cpu_load". This is the link to the github repo: https://github.com/Mav-ninerniner/canonical-assesment-ubuntu-server-certification

#!/usr/bin/env python3
import os
import subprocess
import sys
import argparse

def get_cpu_stats():
    with open("/proc/stat") as f:
        return list(map(int, f.readline().split()[1:]))

def compute_cpu_load(start, end):
    diff_idle = end[3] - start[3]
    diff_total = sum(end) - sum(start)
    diff_used = diff_total - diff_idle
    if diff_total != 0:
        return (diff_used * 100) / diff_total
    else:
        return 0

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--max-load", type=int, default=30)
    parser.add_argument("--xfer", type=int, default=4096)
    parser.add_argument("--verbose", action='store_true')
    parser.add_argument("device", nargs='?', default="/dev/sda")
    args = parser.parse_args()

    disk_device = args.device
    verbose = args.verbose
    max_load = args.max_load
    xfer = args.xfer

    if not os.path.exists(disk_device):
        print(f"Unknown block device {disk_device}")
        sys.exit(1)

    print(f"Testing CPU load when reading {xfer} MiB from {disk_device}")
    print(f"Maximum acceptable CPU load is {max_load}")

    subprocess.run(["blockdev", "--flushbufs", disk_device], check=True)

    start_load = get_cpu_stats()

    subprocess.run(["dd", f"if={disk_device}", "of=/dev/null", f"bs={1024**2}", f"count={xfer}"], check=True)

    end_load = get_cpu_stats()

    cpu_load = compute_cpu_load(start_load, end_load)

    print(f"Detected disk read CPU load is {cpu_load}")

    if cpu_load > max_load:
        print("*** DISK CPU LOAD TEST HAS FAILED! ***")
        sys.exit(1)

if _name_ == "_main_":
    main()
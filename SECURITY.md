P1
Implementation of Process and
I/O System calls 


import os

# ---- Process ----
print("Starting Process System Calls Implementation...")

pid = os.fork()
if pid == 0:
    print(f"Child: PID={os.getpid()}, Parent={os.getppid()}")
    os.execlp("echo", "echo", "Hello from Child Process!")
else:
    print(f"Parent: PID={os.getpid()}")
    os.wait()
    print("Child Process has completed.")

# ---- I/O ----
print("\nOutput for I/O System Calls:")
print("Starting I/O System Calls Implementation...")

file = "test.txt"

# write
fd = os.open(file, os.O_WRONLY | os.O_CREAT | os.O_TRUNC, 0o644)
os.write(fd, b"hello this is sample text written using I/O system calls.\n")
os.close(fd)
print(f"Data written to file: {file}")

# read
fd = os.open(file, os.O_RDONLY)
data = os.read(fd, 100)
os.close(fd)

print("Content read from file:")
print(data.decode())

========================================================================================================================================P2 
Implementation of CPU
Scheduling Algorithms


processes = [(1,6),(2,8),(3,7),(4,3)]

# FCFS
wt=[0]*4
for i in range(1,4): wt[i]=processes[i-1][1]+wt[i-1]
print("FCFS:\nProcess\tBT\tWT\tTAT")
for i in range(4):
    tat=processes[i][1]+wt[i]
    print(f"{processes[i][0]}\t{processes[i][1]}\t{wt[i]}\t{tat}")
    avg_wt = sum(wt)/4
    avg_tat = sum(wt[i]+processes[i][1] for i in range(4))/4
    
print(f"Average WT = {avg_wt}")
print(f"Average TAT = {avg_tat}")

# SJF
p=sorted(processes,key=lambda x:x[1])
wt=[0]*4
for i in range(1,4): wt[i]=p[i-1][1]+wt[i-1]
print("\nSJF:\nProcess\tBT\tWT\tTAT")
for i in range(4):
    tat=p[i][1]+wt[i]
    print(f"{p[i][0]}\t{p[i][1]}\t{wt[i]}\t{tat}")
    avg_wt = sum(wt)/4
    avg_tat = sum(wt[i]+p[i][1] for i in range(4))/4
    
print(f"Average WT = {avg_wt}")
print(f"Average TAT = {avg_tat}")

# Round Robin
rem=[x[1] for x in processes]; wt=[0]*4; t=0; quantum=4
while any(r>0 for r in rem):
    for i in range(4):
        if rem[i]>0:
            if rem[i]>quantum: t+=quantum; rem[i]-=quantum
            else: t+=rem[i]; wt[i]=t-processes[i][1]; rem[i]=0
print("\nRound Robin (q=4):\nProcess\tBT\tWT\tTAT")
for i in range(4):
    print(f"{processes[i][0]}\t{processes[i][1]}\t{wt[i]}\t{wt[i]+processes[i][1]}")
    avg_wt = sum(wt)/4
    avg_tat = sum(wt[i]+p[i][1] for i in range(4))/4
    
print(f"Average WT = {avg_wt}")
print(f"Average TAT = {avg_tat}")

========================================================================================================================================
P3

Implementation of Classical
Synchronization problems
using semaphores. 

import threading, time, random

print("Producer - Consumer Problem")

buffer=[]; mutex=threading.Semaphore(1)
empty=threading.Semaphore(5); full=threading.Semaphore(0)

def producer():
    for _ in range(5):
        item=random.randint(1,100)
        empty.acquire(); mutex.acquire()
        buffer.append(item)
        print(f"Producer produced : {item}")
        mutex.release(); full.release()
        time.sleep(0.5)

def consumer():
    for _ in range(5):
        full.acquire(); mutex.acquire()
        item=buffer.pop(0)
        print(f"Consumer consumed : {item}")
        mutex.release(); empty.release()
        time.sleep(1)

t1=threading.Thread(target=producer)
t2=threading.Thread(target=consumer)
t1.start(); t2.start()
t1.join(); t2.join()

# ----------------------------

print("\nReaders - Writers Problem")

resource=0; rc=0
rmux=threading.Semaphore(1)
wmux=threading.Semaphore(1)

def reader(i):
    global rc
    rmux.acquire()
    rc+=1
    if rc==1: wmux.acquire()
    rmux.release()

    print(f"Reader - {i} is reading : {resource}")

    rmux.acquire()
    rc-=1
    if rc==0: wmux.release()
    rmux.release()

def writer(i):
    global resource
    wmux.acquire()
    resource+=1
    print(f"Writer - {i} is writing : {resource}")
    wmux.release()

threads=[]
for i in range(3):
    threads.append(threading.Thread(target=reader,args=(i,)))
for i in range(2):
    threads.append(threading.Thread(target=writer,args=(i,)))

for t in threads: t.start()
for t in threads: t.join()


========================================================================================================================================
P4
Implementation of Memory
Allocation Strategies


mem=[100]

def first_fit(size):
    for i in range(len(mem)):
        if mem[i]>=size:
            print(f"First-Fit: {size} in block {mem[i]}")
            mem[i]-=size; return
    print("No block")

def best_fit(size):
    idx=min((i for i in range(len(mem)) if mem[i]>=size),
            key=lambda i:mem[i], default=-1)
    if idx!=-1:
        print(f"Best-Fit: {size} in block {mem[idx]}")
        mem[idx]-=size

def worst_fit(size):
    idx=max((i for i in range(len(mem)) if mem[i]>=size),
            key=lambda i:mem[i], default=-1)
    if idx!=-1:
        print(f"Worst-Fit: {size} in block {mem[idx]}")
        mem[idx]-=size

last=0
def next_fit(size):
    global last
    for i in range(last,len(mem)):
        if mem[i]>=size:
            print(f"Next-Fit: {size} in block {mem[i]}")
            mem[i]-=size; last=i; return

# Test
mem=[100]; first_fit(30); first_fit(20); first_fit(50); print("Mem:",mem)
mem=[100]; best_fit(30);  best_fit(20);  best_fit(50);  print("Mem:",mem)
mem=[100]; worst_fit(30); worst_fit(20); worst_fit(50); print("Mem:",mem)
mem=[100]; last=0; next_fit(30); next_fit(20); next_fit(50); print("Mem:",mem)

========================================================================================================================================
P5
Implementation of Page
Replacement Algorithms 

pages = [7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 3, 0, 3]
nf = 3

# FIFO
f = []
faults = 0
for p in pages:
    if p not in f:
        if len(f) < nf:
            f.append(p)
        else:
            f.pop(0)
            f.append(p)
        faults += 1
print(f"FIFO Page Faults: {faults}")

# LRU
f = []
faults = 0
for p in pages:
    if p not in f:
        if len(f) < nf:
            f.append(p)
        else:
            f.pop(0)
            f.append(p)
        faults += 1
    else:
        f.remove(p)
        f.append(p)
print(f"LRU Page Faults: {faults}")

# Optimal
f = []
faults = 0
for i, p in enumerate(pages):
    if p not in f:
        if len(f) < nf:
            f.append(p)
        else:
            far = -1
            rep = -1
            for j, fr in enumerate(f):
                if fr not in pages[i+1:]:
                    rep = j
                    break
                else:
                    dist = pages[i+1:].index(fr)
                    if dist > far:
                        far = dist
                        rep = j
            f[rep] = p
        faults += 1
print(f"Optimal Page Faults: {faults}")

# Clock
f = []
bits = []
ptr = 0
faults = 0
for p in pages:
    if p not in f:
        if len(f) < nf:
            f.append(p)
            bits.append(1)
        else:
            while bits[ptr] == 1:
                bits[ptr] = 0
                ptr = (ptr + 1) % nf
            f[ptr] = p
            bits[ptr] = 1
        faults += 1
    else:
        bits[f.index(p)] = 1
print(f"Clock Page Faults: {faults}")

========================================================================================================================================
P6
Implementation of Disk
Scheduling Algorithms 

req = [176, 79, 34, 60, 92, 11, 41, 114]
head = 50

# FCFS
t = 0
cur = head
for r in req:
    t += abs(cur - r)
    cur = r
print(f"FCFS Seek Time: {t}")

# SSTF
t = 0
cur = head
rem = req[:]
while rem:
    c = min(rem, key=lambda x: abs(x - cur))
    t += abs(cur - c)
    cur = c
    rem.remove(c)
print(f"SSTF Seek Time: {t}")

# SCAN
s = sorted(req)
left = [x for x in s if x < head]
right = [x for x in s if x >= head]
t = 0
if right:
    t += abs(head - right[0])
    t += abs(right[0] - right[-1])
    if left:
        t += abs(right[-1] - left[0])
print(f"SCAN Seek Time: {t}")

# C-SCAN
s = sorted(req)
left = [x for x in s if x < head]
right = [x for x in s if x >= head]
t = 0
if right and left:
    t = abs(head - right[-1]) + abs(right[-1] - left[0])
print(f"C-SCAN Seek Time: {t}")

# LOOK
s = sorted(req)
left = [x for x in s if x < head]
right = [x for x in s if x >= head]
t = abs(head - (min(left) if left else max(right)))
print(f"LOOK Seek Time: {t}")

# C-LOOK
s = sorted(req)
right = [x for x in s if x >= head]
t = abs(head - max(right)) if right else 0
print(f"C-LOOK Seek Time: {t}")































# Security Policy

## Supported Versions

Use this section to tell people about which versions of your project are
currently being supported with security updates.

| Version | Supported          |
| ------- | ------------------ |
| 5.1.x   | :white_check_mark: |
| 5.0.x   | :x:                |
| 4.0.x   | :white_check_mark: |
| < 4.0   | :x:                |

## Reporting a Vulnerability

Use this section to tell people how to report a vulnerability.

Tell them where to go, how often they can expect to get an update on a
reported vulnerability, what to expect if the vulnerability is accepted or
declined, etc.

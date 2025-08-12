# Python System and Process Modules Cheatsheet

## multiprocessing Module - Process-Based Parallelism

### Creating and Managing Processes

```python
import multiprocessing as mp
from multiprocessing import Process, Queue, Pipe, Value, Array, Lock
import time

# Basic process creation
def worker_function(name, delay):
    print(f"Worker {name} starting")
    time.sleep(delay)
    print(f"Worker {name} finished")

# Create and start process
p = Process(target=worker_function, args=("Process-1", 2))
p.start()                             # Start the process
p.join()                              # Wait for process to complete
p.is_alive()                          # Check if process is running
p.terminate()                         # Terminate process
p.kill()                              # Kill process (Python 3.7+)
```

### Process Pools

```python
from multiprocessing import Pool
import os

def square(x):
    return x * x

# Process pool with context manager
with Pool(processes=4) as pool:
    results = pool.map(square, [1, 2, 3, 4, 5])
    print(results)  # [1, 4, 9, 16, 25]

# Manual pool management
pool = Pool(processes=os.cpu_count())
result = pool.apply_async(square, (5,))  # Async execution
print(result.get())                      # Get result (blocks)
pool.close()                             # No more tasks
pool.join()                              # Wait for completion

# Map variants
pool.map(func, iterable)                 # Synchronous map
pool.map_async(func, iterable)           # Asynchronous map
pool.imap(func, iterable)                # Iterator version
pool.starmap(func, iterable)             # For multiple arguments
```

### Inter-Process Communication (IPC)

```python
# Queues for process communication
def producer(q):
    for i in range(5):
        q.put(f"item-{i}")
    q.put(None)  # Sentinel to stop consumer

def consumer(q):
    while True:
        item = q.get()
        if item is None:
            break
        print(f"Consumed: {item}")

queue = Queue()
p1 = Process(target=producer, args=(queue,))
p2 = Process(target=consumer, args=(queue,))
p1.start()
p2.start()
p1.join()
p2.join()

# Pipes for two-way communication
parent_conn, child_conn = Pipe()

def child_process(conn):
    conn.send("Hello from child")
    msg = conn.recv()
    print(f"Child received: {msg}")
    conn.close()

p = Process(target=child_process, args=(child_conn,))
p.start()
msg = parent_conn.recv()
parent_conn.send("Hello from parent")
p.join()
```

### Shared Memory

```python
# Shared variables
def worker(shared_value, shared_array, lock):
    with lock:
        shared_value.value += 1
        for i in range(len(shared_array)):
            shared_array[i] *= 2

# Create shared objects
shared_num = Value('i', 0)              # 'i' = integer
shared_arr = Array('d', [1.0, 2.0, 3.0])  # 'd' = double
lock = Lock()

processes = []
for i in range(4):
    p = Process(target=worker, args=(shared_num, shared_arr, lock))
    p.start()
    processes.append(p)

for p in processes:
    p.join()

print(f"Final value: {shared_num.value}")
print(f"Final array: {list(shared_arr)}")

# Manager for complex shared objects
from multiprocessing import Manager

with Manager() as manager:
    shared_dict = manager.dict()
    shared_list = manager.list()
    
    # Use shared_dict and shared_list across processes
```

### Synchronization Primitives

```python
from multiprocessing import Lock, RLock, Semaphore, Event, Condition, Barrier

# Lock (Mutex)
lock = Lock()
with lock:  # Context manager automatically acquires/releases
    # Critical section
    pass

# Semaphore
semaphore = Semaphore(2)  # Allow 2 processes
semaphore.acquire()       # Acquire semaphore
semaphore.release()       # Release semaphore

# Event
event = Event()
event.set()               # Set event
event.clear()             # Clear event
event.wait()              # Wait for event
event.is_set()            # Check if set

# Condition
condition = Condition()
with condition:
    condition.wait()      # Wait for condition
    condition.notify()    # Notify one waiter
    condition.notify_all()  # Notify all waiters

# Barrier (synchronize N processes)
barrier = Barrier(3)      # Wait for 3 processes
barrier.wait()            # Block until all arrive
```

## subprocess Module - External Process Management

### Running Commands

```python
import subprocess
import sys

# Simple command execution
result = subprocess.run(['ls', '-la'], capture_output=True, text=True)
print(result.stdout)      # Command output
print(result.stderr)      # Error output
print(result.returncode)  # Exit code

# Run with shell
result = subprocess.run('ls -la | grep .py', shell=True, 
                       capture_output=True, text=True)

# Check for errors
subprocess.run(['ls', 'nonexistent'], check=True)  # Raises CalledProcessError

# Timeout handling
try:
    result = subprocess.run(['sleep', '10'], timeout=5)
except subprocess.TimeoutExpired:
    print("Command timed out")
```

### Advanced Process Control

```python
# Using Popen for more control
process = subprocess.Popen(['python', 'script.py'], 
                          stdin=subprocess.PIPE,
                          stdout=subprocess.PIPE,
                          stderr=subprocess.PIPE,
                          text=True)

# Communicate with process
stdout, stderr = process.communicate(input="input data\n")
return_code = process.returncode

# Non-blocking I/O
process = subprocess.Popen(['ping', 'google.com'], 
                          stdout=subprocess.PIPE,
                          stderr=subprocess.PIPE,
                          text=True)

# Check if process is still running
if process.poll() is None:
    print("Process is still running")

# Terminate process
process.terminate()       # Send SIGTERM
process.kill()           # Send SIGKILL
process.wait()           # Wait for termination

# Real-time output reading
def read_output(process):
    for line in iter(process.stdout.readline, ''):
        print(line.strip())

process = subprocess.Popen(['python', 'long_script.py'],
                          stdout=subprocess.PIPE,
                          stderr=subprocess.STDOUT,
                          text=True,
                          bufsize=1)
read_output(process)
```

### Convenience Functions

```python
# Quick functions for common tasks
output = subprocess.check_output(['date'], text=True)
subprocess.check_call(['mkdir', 'new_directory'])

# Get command output as bytes
output_bytes = subprocess.check_output(['ls', '-la'])

# Run multiple commands
commands = [
    ['mkdir', 'temp'],
    ['touch', 'temp/file.txt'],
    ['ls', 'temp']
]

for cmd in commands:
    subprocess.run(cmd, check=True)
```

## threading Module - Thread-Based Parallelism

### Creating and Managing Threads

```python
import threading
import time

# Function-based threads
def worker_thread(name, delay):
    print(f"Thread {name} starting")
    time.sleep(delay)
    print(f"Thread {name} finished")

# Create and start thread
thread = threading.Thread(target=worker_thread, args=("Thread-1", 2))
thread.start()            # Start thread
thread.join()             # Wait for completion
thread.is_alive()         # Check if running

# Class-based threads
class WorkerThread(threading.Thread):
    def __init__(self, name, delay):
        super().__init__()
        self.name = name
        self.delay = delay
    
    def run(self):
        print(f"Thread {self.name} starting")
        time.sleep(self.delay)
        print(f"Thread {self.name} finished")

worker = WorkerThread("CustomThread", 1)
worker.start()
worker.join()

# Daemon threads
thread.daemon = True      # Dies when main thread exits
threading.current_thread()  # Get current thread
threading.active_count()    # Number of active threads
threading.enumerate()       # List all threads
```

### Thread Synchronization

```python
# Lock (Mutex)
lock = threading.Lock()
def critical_section():
    with lock:  # Automatic acquire/release
        # Thread-safe code here
        pass

# RLock (Reentrant Lock)
rlock = threading.RLock()
def recursive_function(n):
    with rlock:
        if n > 0:
            recursive_function(n-1)

# Semaphore
semaphore = threading.Semaphore(2)  # Allow 2 threads
def limited_resource():
    with semaphore:
        # Only 2 threads can execute this simultaneously
        time.sleep(1)

# Event
event = threading.Event()
def waiter():
    print("Waiting for event")
    event.wait()          # Block until event is set
    print("Event received")

def setter():
    time.sleep(2)
    event.set()           # Set the event

threading.Thread(target=waiter).start()
threading.Thread(target=setter).start()

# Condition
condition = threading.Condition()
items = []

def consumer():
    with condition:
        while len(items) == 0:
            condition.wait()  # Wait for items
        item = items.pop(0)
        print(f"Consumed {item}")

def producer():
    with condition:
        items.append("item")
        condition.notify()  # Wake up waiting thread

# Barrier
barrier = threading.Barrier(3)  # Wait for 3 threads
def sync_point():
    print(f"Thread {threading.current_thread().name} reached barrier")
    barrier.wait()      # Wait for all threads
    print(f"Thread {threading.current_thread().name} passed barrier")
```

### Thread-Safe Data Structures

```python
import queue
import threading

# Queue for thread communication
q = queue.Queue()         # FIFO queue
q = queue.LifoQueue()     # LIFO (stack)
q = queue.PriorityQueue() # Priority queue

# Producer-Consumer pattern
def producer(q):
    for i in range(5):
        q.put(f"item-{i}")
        print(f"Produced item-{i}")

def consumer(q):
    while True:
        try:
            item = q.get(timeout=1)
            print(f"Consumed {item}")
            q.task_done()
        except queue.Empty:
            break

# Thread pool using concurrent.futures
from concurrent.futures import ThreadPoolExecutor, as_completed

def task(n):
    time.sleep(1)
    return n * n

with ThreadPoolExecutor(max_workers=4) as executor:
    futures = [executor.submit(task, i) for i in range(10)]
    for future in as_completed(futures):
        result = future.result()
        print(f"Result: {result}")
```

## mmap Module - Memory-Mapped Files

### Basic Memory Mapping

```python
import mmap
import os

# Create a file for mapping
with open('test_file.txt', 'w') as f:
    f.write('Hello, World!' * 100)

# Memory map a file
with open('test_file.txt', 'r+b') as f:
    # Map the entire file
    with mmap.mmap(f.fileno(), 0, access=mmap.ACCESS_WRITE) as mm:
        print(f"File size: {len(mm)}")
        print(f"First 20 bytes: {mm[:20]}")
        
        # Modify the file through memory
        mm[0:5] = b'Goodbye'
        mm.flush()  # Ensure changes are written
        
        # Find and replace
        pos = mm.find(b'World')
        if pos != -1:
            mm[pos:pos+5] = b'Earth'
        
        # Move file pointer
        mm.seek(0)
        print(f"After modification: {mm.read(20)}")

# Read-only mapping
with open('test_file.txt', 'rb') as f:
    with mmap.mmap(f.fileno(), 0, access=mmap.ACCESS_READ) as mm:
        # Read-only operations
        data = mm[10:30]
        print(f"Read data: {data}")

# Copy-on-write mapping
with open('test_file.txt', 'rb') as f:
    with mmap.mmap(f.fileno(), 0, access=mmap.ACCESS_COPY) as mm:
        # Changes don't affect the original file
        mm[0:5] = b'COPY!'
        print(f"Modified copy: {mm[:20]}")
```

### Shared Memory Between Processes

```python
import mmap
import multiprocessing
import os

def create_shared_memory():
    # Create anonymous shared memory
    return mmap.mmap(-1, 1024)  # 1KB shared memory

def worker_process(shared_mem, worker_id):
    # Write to shared memory
    message = f"Hello from worker {worker_id}".encode()
    shared_mem.seek(worker_id * 50)
    shared_mem.write(message)
    shared_mem.flush()

# Create shared memory
shared_memory = create_shared_memory()

# Create processes that share memory
processes = []
for i in range(3):
    p = multiprocessing.Process(target=worker_process, 
                               args=(shared_memory, i))
    p.start()
    processes.append(p)

# Wait for all processes
for p in processes:
    p.join()

# Read results from shared memory
shared_memory.seek(0)
for i in range(3):
    shared_memory.seek(i * 50)
    data = shared_memory.read(50).rstrip(b'\x00')
    print(f"Worker {i}: {data.decode()}")

shared_memory.close()
```

## resource Module - System Resource Information

### Resource Usage Statistics

```python
import resource
import time

# Get resource usage
usage = resource.getrusage(resource.RUSAGE_SELF)
print(f"User CPU time: {usage.ru_utime:.2f} seconds")
print(f"System CPU time: {usage.ru_stime:.2f} seconds")
print(f"Max memory usage: {usage.ru_maxrss} KB")  # Linux: KB, macOS: bytes
print(f"Page faults: {usage.ru_majflt}")
print(f"Context switches: {usage.ru_nvcsw} voluntary, {usage.ru_nivcsw} involuntary")

# Monitor resource usage over time
def measure_function_resources(func):
    def wrapper(*args, **kwargs):
        start_usage = resource.getrusage(resource.RUSAGE_SELF)
        start_time = time.time()
        
        result = func(*args, **kwargs)
        
        end_usage = resource.getrusage(resource.RUSAGE_SELF)
        end_time = time.time()
        
        user_cpu = end_usage.ru_utime - start_usage.ru_utime
        sys_cpu = end_usage.ru_stime - start_usage.ru_stime
        wall_time = end_time - start_time
        
        print(f"Function: {func.__name__}")
        print(f"Wall time: {wall_time:.4f}s")
        print(f"User CPU: {user_cpu:.4f}s")
        print(f"System CPU: {sys_cpu:.4f}s")
        print(f"CPU efficiency: {((user_cpu + sys_cpu) / wall_time * 100):.1f}%")
        
        return result
    return wrapper
```

### Resource Limits

```python
# Get and set resource limits
def show_limits():
    limits = [
        ('CPU time', resource.RLIMIT_CPU),
        ('File size', resource.RLIMIT_FSIZE),
        ('Memory', resource.RLIMIT_AS),
        ('Open files', resource.RLIMIT_NOFILE),
        ('Processes', resource.RLIMIT_NPROC),
    ]
    
    for name, limit_type in limits:
        try:
            soft, hard = resource.getrlimit(limit_type)
            print(f"{name}: soft={soft}, hard={hard}")
        except AttributeError:
            print(f"{name}: Not available on this system")

show_limits()

# Set resource limits
try:
    # Limit CPU time to 10 seconds
    resource.setrlimit(resource.RLIMIT_CPU, (10, 10))
    
    # Limit memory to 100MB
    resource.setrlimit(resource.RLIMIT_AS, (100 * 1024 * 1024, 
                                           100 * 1024 * 1024))
except ValueError as e:
    print(f"Cannot set limit: {e}")

# Page size
page_size = resource.getpagesize()
print(f"System page size: {page_size} bytes")
```

## gc Module - Garbage Collection Control

### Garbage Collection Control

```python
import gc
import sys

# Manual garbage collection
collected = gc.collect()  # Force garbage collection
print(f"Collected {collected} objects")

# Garbage collection statistics
stats = gc.get_stats()
for i, stat in enumerate(stats):
    print(f"Generation {i}: {stat}")

# Enable/disable garbage collection
gc.disable()              # Disable automatic GC
gc.enable()               # Enable automatic GC
print(f"GC enabled: {gc.isenabled()}")

# Generation thresholds
thresholds = gc.get_threshold()
print(f"GC thresholds: {thresholds}")
gc.set_threshold(700, 10, 10)  # Set new thresholds

# Object counting
print(f"Total objects: {len(gc.get_objects())}")
print(f"Uncollectable objects: {len(gc.garbage)}")

# Reference counting (not true GC, but related)
import sys
obj = [1, 2, 3]
print(f"Reference count: {sys.getrefcount(obj)}")  # Includes temp reference
```

### Memory Debugging

```python
import gc
import weakref

# Track object creation and destruction
class TrackedObject:
    instances = []
    
    def __init__(self, name):
        self.name = name
        TrackedObject.instances.append(weakref.ref(self))
        print(f"Created {name}")
    
    def __del__(self):
        print(f"Destroyed {self.name}")

# Create objects
obj1 = TrackedObject("Object1")
obj2 = TrackedObject("Object2")

# Check alive instances
alive = [ref() for ref in TrackedObject.instances if ref() is not None]
print(f"Alive instances: {len(alive)}")

# Force cleanup
del obj1, obj2
gc.collect()

# Find circular references
def find_circular_references():
    gc.collect()  # Ensure GC has run
    
    for obj in gc.garbage:
        print(f"Uncollectable object: {type(obj)} - {obj}")
        
    # Find referrers of an object
    obj = [1, 2, 3]
    referrers = gc.get_referrers(obj)
    print(f"Objects referring to list: {len(referrers)}")

# Memory growth tracking
def track_memory_growth():
    import psutil
    import os
    
    process = psutil.Process(os.getpid())
    
    print(f"Memory before: {process.memory_info().rss / 1024 / 1024:.1f} MB")
    
    # Create many objects
    big_list = [list(range(1000)) for _ in range(1000)]
    
    print(f"Memory after creation: {process.memory_info().rss / 1024 / 1024:.1f} MB")
    
    del big_list
    gc.collect()
    
    print(f"Memory after deletion: {process.memory_info().rss / 1024 / 1024:.1f} MB")
```

## platform Module - Platform Information

### System Information

```python
import platform
import os

# Basic platform information
print(f"System: {platform.system()}")           # 'Linux', 'Windows', 'Darwin'
print(f"Release: {platform.release()}")         # OS release version
print(f"Version: {platform.version()}")         # Detailed version info
print(f"Machine: {platform.machine()}")         # Machine type (x86_64, ARM, etc.)
print(f"Processor: {platform.processor()}")     # Processor name
print(f"Architecture: {platform.architecture()}")  # ('64bit', 'ELF')
print(f"Platform: {platform.platform()}")       # Complete platform string

# Python information
print(f"Python version: {platform.python_version()}")
print(f"Python implementation: {platform.python_implementation()}")  # CPython, PyPy
print(f"Python compiler: {platform.python_compiler()}")
print(f"Python build: {platform.python_build()}")

# Network information
print(f"Node (hostname): {platform.node()}")
print(f"FQDN: {platform.fqdn()}")  # Fully qualified domain name

# Detailed system info
uname = platform.uname()
print(f"System info: {uname}")
print(f"OS: {uname.system} {uname.release}")
print(f"Machine: {uname.machine}")
print(f"Processor: {uname.processor}")
```

### Platform-Specific Code

```python
import platform
import sys

# Cross-platform file paths
if platform.system() == 'Windows':
    config_dir = os.path.join(os.environ['APPDATA'], 'MyApp')
    temp_dir = os.environ['TEMP']
elif platform.system() == 'Darwin':  # macOS
    config_dir = os.path.expanduser('~/Library/Application Support/MyApp')
    temp_dir = '/tmp'
else:  # Linux and other Unix-like
    config_dir = os.path.expanduser('~/.config/MyApp')
    temp_dir = '/tmp'

print(f"Config directory: {config_dir}")
print(f"Temp directory: {temp_dir}")

# Platform-specific features
def get_cpu_count():
    if hasattr(os, 'cpu_count'):
        return os.cpu_count()
    else:
        # Fallback for older Python versions
        import multiprocessing
        return multiprocessing.cpu_count()

# Check for specific OS features
def has_fork():
    return hasattr(os, 'fork')

def supports_symlinks():
    return hasattr(os, 'symlink')

print(f"CPU cores: {get_cpu_count()}")
print(f"Supports fork: {has_fork()}")
print(f"Supports symlinks: {supports_symlinks()}")
```

## sys Module - System-Specific Parameters

### Python Runtime Information

```python
import sys

# Python version info
print(f"Python version: {sys.version}")
print(f"Version info: {sys.version_info}")
print(f"API version: {sys.api_version}")
print(f"Platform: {sys.platform}")

# Executable and paths
print(f"Python executable: {sys.executable}")
print(f"Installation prefix: {sys.prefix}")
print(f"Exec prefix: {sys.exec_prefix}")

# Module search paths
print("Python path:")
for path in sys.path:
    print(f"  {path}")

# Built-in modules
print(f"Built-in modules: {len(sys.builtin_module_names)}")
print(f"Some built-ins: {list(sys.builtin_module_names)[:10]}")

# Loaded modules
print(f"Loaded modules: {len(sys.modules)}")
print(f"Some loaded: {list(sys.modules.keys())[:10]}")
```

### Memory and Performance

```python
import sys

# Memory information
def show_memory_info():
    print(f"Reference count limit: {sys.maxsize}")
    print(f"Max Unicode: {sys.maxunicode}")
    print(f"Float info: {sys.float_info}")
    print(f"Int info: {sys.int_info}")
    
    # Object sizes
    print(f"Size of empty list: {sys.getsizeof([])} bytes")
    print(f"Size of empty dict: {sys.getsizeof({})} bytes")
    print(f"Size of empty string: {sys.getsizeof('')} bytes")

show_memory_info()

# Recursion limit
print(f"Recursion limit: {sys.getrecursionlimit()}")
sys.setrecursionlimit(2000)  # Increase if needed

# Reference counting
obj = [1, 2, 3, 4, 5]
print(f"Reference count: {sys.getrefcount(obj)}")

# Function call tracing
def trace_calls(frame, event, arg):
    if event == 'call':
        filename = frame.f_code.co_filename
        func_name = frame.f_code.co_name
        print(f"Calling {func_name} in {filename}")
    return trace_calls

# Enable tracing (use with caution - very slow!)
# sys.settrace(trace_calls)
```

### Command Line and Environment

```python
import sys
import os

# Command line arguments
print(f"Script name: {sys.argv[0]}")
print(f"Arguments: {sys.argv[1:]}")
print(f"Argument count: {len(sys.argv)}")

# Standard streams
print("This goes to stdout", file=sys.stdout)
print("This goes to stderr", file=sys.stderr)

# Check if running interactively
def is_interactive():
    return hasattr(sys, 'ps1')

# Exit codes
def exit_with_code(code=0):
    sys.exit(code)  # 0 = success, non-zero = error

# Platform checks
def is_windows():
    return sys.platform.startswith('win')

def is_linux():
    return sys.platform.startswith('linux')

def is_macos():
    return sys.platform == 'darwin'

print(f"Running on Windows: {is_windows()}")
print(f"Running on Linux: {is_linux()}")
print(f"Running on macOS: {is_macos()}")
```

## psutil Module - System and Process Monitoring

### System Information

```python
import psutil
import datetime

# CPU information
print(f"CPU cores (logical): {psutil.cpu_count()}")
print(f"CPU cores (physical): {psutil.cpu_count(logical=False)}")
print(f"CPU frequency: {psutil.cpu_freq()}")
print(f"CPU usage: {psutil.cpu_percent(interval=1)}%")

# Per-CPU usage
cpu_percents = psutil.cpu_percent(interval=1, percpu=True)
for i, percent in enumerate(cpu_percents):
    print(f"CPU {i}: {percent}%")

# Memory information
memory = psutil.virtual_memory()
print(f"Total memory: {memory.total / (1024**3):.2f} GB")
print(f"Available memory: {memory.available / (1024**3):.2f} GB")
print(f"Memory usage: {memory.percent}%")

# Swap memory
swap = psutil.swap_memory()
print(f"Total swap: {swap.total / (1024**3):.2f} GB")
print(f"Swap usage: {swap.percent}%")

# Disk information
disk_usage = psutil.disk_usage('/')
print(f"Total disk: {disk_usage.total / (1024**3):.2f} GB")
print(f"Free disk: {disk_usage.free / (1024**3):.2f} GB")
print(f"Disk usage: {(disk_usage.used / disk_usage.total) * 100:.1f}%")

# Disk I/O statistics
disk_io = psutil.disk_io_counters()
if disk_io:
    print(f"Disk reads: {disk_io.read_count}")
    print(f"Disk writes: {disk_io.write_count}")
    print(f"Bytes read: {disk_io.read_bytes / (1024**2):.2f} MB")
    print(f"Bytes written: {disk_io.write_bytes / (1024**2):.2f} MB")

# Network information
network_io = psutil.net_io_counters()
print(f"Bytes sent: {network_io.bytes_sent / (1024**2):.2f} MB")
print(f"Bytes received: {network_io.bytes_recv / (1024**2):.2f} MB")
print(f"Packets sent: {network_io.packets_sent}")
print(f"Packets received: {network_io.packets_recv}")

# Boot time
boot_time = datetime.datetime.fromtimestamp(psutil.boot_time())
print(f"System boot time: {boot_time}")
```

### Process Management

```python
import psutil
import os

# Current process
current = psutil.Process()
print(f"Current PID: {current.pid}")
print(f"Process name: {current.name()}")
print(f"Command line: {current.cmdline()}")
print(f"Status: {current.status()}")
print(f"CPU usage: {current.cpu_percent()}%")
print(f"Memory usage: {current.memory_info().rss / (1024**2):.2f} MB")
print(f"Memory percent: {current.memory_percent():.2f}%")
print(f"Number of threads: {current.num_threads()}")

# Process list
def show_processes():
    print(f"{'PID':<8} {'Name':<20} {'CPU%':<8} {'Memory%':<10} {'Status'}")
    print("-" * 60)
    
    for proc in psutil.process_iter(['pid', 'name', 'cpu_percent', 
                                    'memory_percent', 'status']):
        try:
            info = proc.info
            print(f"{info['pid']:<8} {info['name'][:19]:<20} "
                  f"{info['cpu_percent']:<8.1f} {info['memory_percent']:<10.2f} "
                  f"{info['status']}")
        except (psutil.NoSuchProcess, psutil.AccessDenied):
            pass

# show_processes()  # Uncomment to run

# Find processes by name
def find_process_by_name(name):
    processes = []
    for proc in psutil.process_iter(['pid', 'name']):
        if name.lower() in proc.info['name'].lower():
            processes.append(proc)
    return processes

# Monitor specific process
def monitor_process(pid, duration=10):
    try:
        proc = psutil.Process(pid)
        print(f"Monitoring process {pid} ({proc.name()}) for {duration} seconds")
        
        for i in range(duration):
            cpu = proc.cpu_percent()
            memory = proc.memory_info().rss / (1024**2)
            print(f"Time {i+1}: CPU={cpu:.1f}%, Memory={memory:.2f}MB")
            time.sleep(1)
            
    except psutil.NoSuchProcess:
        print(f"Process {pid} not found")
    except psutil.AccessDenied:
        print(f"Access denied to process {pid}")

# Kill process by name
def kill_process_by_name(name):
    killed = 0
    for proc in psutil.process_iter(['pid', 'name']):
        if name.lower() in proc.info['name'].lower():
            try:
                proc.terminate()  # Send SIGTERM
                proc.wait(timeout=3)  # Wait for graceful termination
                killed += 1
            except (psutil.NoSuchProcess, psutil.AccessDenied, psutil.TimeoutExpired):
                try:
                    proc.kill()  # Force kill
                    killed += 1
                except:
                    pass
    return killed

# Process tree
def show_process_tree(pid=None):
    if pid is None:
        pid = os.getpid()
    
    try:
        parent = psutil.Process(pid)
        children = parent.children(recursive=True)
        
        print(f"Process tree for {parent.name()} (PID: {pid}):")
        print(f"  {parent.name()} (PID: {parent.pid})")
        
        for child in children:
            try:
                print(f"    └─ {child.name()} (PID: {child.pid})")
            except psutil.NoSuchProcess:
                pass
                
    except psutil.NoSuchProcess:
        print(f"Process {pid} not found")
```

### Network Connections

```python
# Network connections
def show_network_connections():
    connections = psutil.net_connections()
    print(f"{'Proto':<6} {'Local Address':<20} {'Remote Address':<20} {'Status':<12} {'PID'}")
    print("-" * 80)
    
    for conn in connections:
        proto = "TCP" if conn.type == 1 else "UDP"
        local = f"{conn.laddr.ip}:{conn.laddr.port}" if conn.laddr else ""
        remote = f"{conn.raddr.ip}:{conn.raddr.port}" if conn.raddr else ""
        status = conn.status or ""
        pid = conn.pid or ""
        
        print(f"{proto:<6} {local:<20} {remote:<20} {status:<12} {pid}")

# Network interfaces
def show_network_interfaces():
    interfaces = psutil.net_if_addrs()
    stats = psutil.net_if_stats()
    
    for interface, addresses in interfaces.items():
        print(f"\nInterface: {interface}")
        
        if interface in stats:
            stat = stats[interface]
            print(f"  Status: {'Up' if stat.isup else 'Down'}")
            print(f"  Speed: {stat.speed} Mbps")
            print(f"  MTU: {stat.mtu}")
        
        for addr in addresses:
            if addr.family == 2:  # IPv4
                print(f"  IPv4: {addr.address}")
                if addr.netmask:
                    print(f"  Netmask: {addr.netmask}")
            elif addr.family == 10:  # IPv6
                print(f"  IPv6: {addr.address}")

# Real-time network monitoring
def monitor_network(duration=10):
    print("Monitoring network I/O (press Ctrl+C to stop)")
    
    old_stats = psutil.net_io_counters()
    
    for i in range(duration):
        time.sleep(1)
        new_stats = psutil.net_io_counters()
        
        bytes_sent = new_stats.bytes_sent - old_stats.bytes_sent
        bytes_recv = new_stats.bytes_recv - old_stats.bytes_recv
        
        print(f"Sent: {bytes_sent/1024:.2f} KB/s, "
              f"Received: {bytes_recv/1024:.2f} KB/s")
        
        old_stats = new_stats
```

### System Monitoring and Alerts

```python
import time
import smtplib
from email.mime.text import MIMEText

class SystemMonitor:
    def __init__(self):
        self.cpu_threshold = 80
        self.memory_threshold = 85
        self.disk_threshold = 90
        self.alerts_sent = {}
    
    def check_cpu(self):
        cpu_percent = psutil.cpu_percent(interval=1)
        if cpu_percent > self.cpu_threshold:
            return f"HIGH CPU USAGE: {cpu_percent:.1f}%"
        return None
    
    def check_memory(self):
        memory = psutil.virtual_memory()
        if memory.percent > self.memory_threshold:
            return f"HIGH MEMORY USAGE: {memory.percent:.1f}%"
        return None
    
    def check_disk(self):
        disk = psutil.disk_usage('/')
        disk_percent = (disk.used / disk.total) * 100
        if disk_percent > self.disk_threshold:
            return f"LOW DISK SPACE: {disk_percent:.1f}% used"
        return None
    
    def check_processes(self):
        high_cpu_procs = []
        for proc in psutil.process_iter(['pid', 'name', 'cpu_percent']):
            try:
                if proc.info['cpu_percent'] > 50:
                    high_cpu_procs.append(
                        f"{proc.info['name']} (PID: {proc.info['pid']}) - "
                        f"{proc.info['cpu_percent']:.1f}% CPU"
                    )
            except (psutil.NoSuchProcess, psutil.AccessDenied):
                pass
        
        if high_cpu_procs:
            return "HIGH CPU PROCESSES:\n" + "\n".join(high_cpu_procs[:5])
        return None
    
    def send_alert(self, message):
        # Simple alert mechanism (print to console)
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        print(f"[ALERT {timestamp}] {message}")
        
        # Could extend to send email, write to log file, etc.
    
    def monitor(self, interval=60):
        print(f"Starting system monitoring (checking every {interval}s)")
        
        while True:
            alerts = []
            
            # Check various system metrics
            cpu_alert = self.check_cpu()
            if cpu_alert:
                alerts.append(cpu_alert)
            
            memory_alert = self.check_memory()
            if memory_alert:
                alerts.append(memory_alert)
            
            disk_alert = self.check_disk()
            if disk_alert:
                alerts.append(disk_alert)
            
            proc_alert = self.check_processes()
            if proc_alert:
                alerts.append(proc_alert)
            
            # Send alerts
            for alert in alerts:
                self.send_alert(alert)
            
            time.sleep(interval)

# Usage example
# monitor = SystemMonitor()
# monitor.monitor(interval=30)  # Check every 30 seconds
```

### Performance Profiling

```python
import psutil
import time
import json

class PerformanceProfiler:
    def __init__(self, pid=None):
        self.pid = pid or os.getpid()
        self.process = psutil.Process(self.pid)
        self.data = []
    
    def collect_sample(self):
        try:
            sample = {
                'timestamp': time.time(),
                'cpu_percent': self.process.cpu_percent(),
                'memory_rss': self.process.memory_info().rss,
                'memory_vms': self.process.memory_info().vms,
                'memory_percent': self.process.memory_percent(),
                'num_threads': self.process.num_threads(),
                'num_fds': self.process.num_fds() if hasattr(self.process, 'num_fds') else 0,
                'io_read_count': self.process.io_counters().read_count if hasattr(self.process, 'io_counters') else 0,
                'io_write_count': self.process.io_counters().write_count if hasattr(self.process, 'io_counters') else 0,
            }
            self.data.append(sample)
            return sample
        except (psutil.NoSuchProcess, psutil.AccessDenied):
            return None
    
    def profile_function(self, func, *args, **kwargs):
        # Profile a function execution
        start_time = time.time()
        start_sample = self.collect_sample()
        
        result = func(*args, **kwargs)
        
        end_sample = self.collect_sample()
        end_time = time.time()
        
        if start_sample and end_sample:
            duration = end_time - start_time
            cpu_usage = end_sample['cpu_percent'] - start_sample['cpu_percent']
            memory_diff = end_sample['memory_rss'] - start_sample['memory_rss']
            
            profile_result = {
                'function': func.__name__,
                'duration': duration,
                'cpu_usage': cpu_usage,
                'memory_change': memory_diff,
                'start_memory': start_sample['memory_rss'],
                'end_memory': end_sample['memory_rss']
            }
            
            print(f"Function: {func.__name__}")
            print(f"Duration: {duration:.4f}s")
            print(f"CPU Usage: {cpu_usage:.2f}%")
            print(f"Memory Change: {memory_diff / 1024 / 1024:.2f} MB")
            
            return result, profile_result
        
        return result, None
    
    def continuous_profile(self, duration, interval=1):
        print(f"Profiling process {self.pid} for {duration} seconds")
        
        start_time = time.time()
        while time.time() - start_time < duration:
            sample = self.collect_sample()
            if sample:
                print(f"CPU: {sample['cpu_percent']:6.1f}% | "
                      f"Memory: {sample['memory_rss']/1024/1024:7.2f} MB | "
                      f"Threads: {sample['num_threads']:3d}")
            time.sleep(interval)
    
    def save_profile(self, filename):
        with open(filename, 'w') as f:
            json.dump(self.data, f, indent=2)
        print(f"Profile data saved to {filename}")
    
    def get_summary(self):
        if not self.data:
            return "No profile data collected"
        
        cpu_values = [d['cpu_percent'] for d in self.data]
        memory_values = [d['memory_rss'] for d in self.data]
        
        return {
            'samples': len(self.data),
            'avg_cpu': sum(cpu_values) / len(cpu_values),
            'max_cpu': max(cpu_values),
            'avg_memory_mb': sum(memory_values) / len(memory_values) / 1024 / 1024,
            'max_memory_mb': max(memory_values) / 1024 / 1024,
        }

# Usage examples
def example_function():
    # Simulate some work
    data = [i**2 for i in range(100000)]
    time.sleep(0.1)
    return len(data)

# Profile a function
profiler = PerformanceProfiler()
result, profile = profiler.profile_function(example_function)

# Continuous profiling
# profiler.continuous_profile(duration=10, interval=0.5)

# Save and analyze results
# profiler.save_profile('profile_results.json')
# summary = profiler.get_summary()
# print(f"Profile Summary: {summary}")
```

## Quick Reference Summary

### Most Common Operations

```python
# Process Management
import multiprocessing as mp
with mp.Pool() as pool:
    results = pool.map(func, data)

# Threading
import threading
lock = threading.Lock()
thread = threading.Thread(target=func, args=(arg,))

# Subprocess
import subprocess
result = subprocess.run(['ls', '-la'], capture_output=True, text=True)

# System Info
import psutil
print(f"CPU: {psutil.cpu_percent()}%, Memory: {psutil.virtual_memory().percent}%")

# Memory Mapping
import mmap
with open('file', 'r+b') as f:
    with mmap.mmap(f.fileno(), 0) as mm:
        data = mm.read()

# Resource Monitoring
import resource
usage = resource.getrusage(resource.RUSAGE_SELF)
print(f"CPU time: {usage.ru_utime:.2f}s")

# Platform Detection
import platform
if platform.system() == 'Windows':
    # Windows-specific code
    pass

# System Configuration
import sys
print(f"Python path: {sys.path}")
print(f"Platform: {sys.platform}")
```

### Performance Tips

1. **Use multiprocessing for CPU-bound tasks**
2. **Use threading for I/O-bound tasks**
3. **Memory-map large files instead of loading into RAM**
4. **Monitor resource usage with psutil**
5. **Set appropriate resource limits**
6. **Use subprocess for external commands**
7. **Check platform compatibility before using OS-specific features**
8. **Profile your applications to identify bottlenecks**
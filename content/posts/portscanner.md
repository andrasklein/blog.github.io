---
title: "Building a port scanner in Python"
date: 2024-09-13T17:25:24+02:00
draft: false
---

I have found myself in this situation many times along my journey on HTB and THM machines, I needed a port scanner that was easy to transfer to a target machine and use from that point utilizing available technologies.

Original author credits go to Heath Adams. My tool is based heavily on what I learned from TCM Academy in the PEH course.

This is what was used in the PEH course.


```python

import sys
import socket
from datetime import datetime

if len(sys.argv) == 2:
	target = socket.gethostbyname(sys.argv[1]) 
else:
	print("Invalid amount of arguments.")
	print("Syntax: python3 scanner.py")

print("-" * 50)
print("Scanning target "+target)
print("Time started: "+str(datetime.now()))
print("-" * 50)

try:
	for port in range(50,85):
		s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		socket.setdefaulttimeout(1)
		result = s.connect_ex((target,port)) 
		if result == 0:
			print("Port {} is open".format(port))
		s.close()

except KeyboardInterrupt:
	print("\nExiting program.")
	sys.exit()
	
except socket.gaierror:
	print("Hostname could not be resolved.")
	sys.exit()

except socket.error:
	print("Could not connect to server.")
	sys.exit()
```
---

Here are the changes I made to it:

I added the "argparse" library to have the freedom to have a greater syntax when executing the code.

```python
import argparse
```

It allows the user to specify with the -p and -i what IP address and port the user wants to scan for.

I added the "threading" Python module which allows to have separate operations by creating so-called threads. Why is it important? Because with threading a user can scan multiple ports at the same time so it speeds up the scan.

```python
import threading
```
---
I also created functions for more robust solutions and defined the following things in the scan_port function:

- setting up a timeout
```python
 s.settimeout(1)
```

This makes it possible to prevent the program from hanging if a port is closed or there are any other issues. This makes it possible to move on to the next port if needed.

- checking for open port

```python
result = s.connect_ex((target, port))
```

This is the core function. What does it do? It attempts to connect to the specified IP on the specified port(s).

- grabbing the banner

```python
if result == 0:
    try:
        banner = s.recv(1024).decode().strip()
        print(f"Port {port} is open | Service: {banner}")
    except:
        print(f"Port {port} is open | Service: Unknown")
```

This part helps to understand what service is behind the currently scanned port.

- handling errors

```python
except socket.error:
        print(f"Could not connect to port {port}")
    except Exception as e:
        print(f"Error scanning port {port}: {e}")
```
This way the scanner can easily handle issues.

---
I added the run_scan function and defined the following things:

- banner for the output
```python
print("-" * 50)
print(f"Scanning target {target}")
print("Time started: " + str(datetime.now()))
print("-" * 50)
```
This prints out a banner to the console and also provides the user with feedback.

- utilized the previously imported threading module
```python
threads = []
for port in port_range:
    thread = threading.Thread(target=scan_port, args=(target, port))
    threads.append(thread)
    thread.start()
```
For each port, it creates a thread that will execute the function that it belongs to.

- output the finished scan

```python
print("-" * 50)
print("Scanning complete.")
print(f"Time finished: {str(datetime.now())}")
```
This provides feedback about the scan's completion and duration.

---

The last part of the code defines the following things:

- which argument should do wath
```python
parser = argparse.ArgumentParser(description="A simple multi-threaded port scanner.")
parser.add_argument('-i', '--ip', type=str, required=True, help='Target IP address')
parser.add_argument('-p', '--ports', type=str, required=True, help='Port range to scan (e.g., 20-80)')
```
This allows the user of the script to specify which IP address and port range they want to scan.

- store user input in a variable
```python
args = parser.parse_args()
```
This variable named args stores what the user inputs into the command line. This variable is used later in other processes.

- processing the port range 

```python
port_range = args.ports.split('-')
start_port = int(port_range[0])
end_port = int(port_range[1])
port_range = range(start_port, end_port + 1)
```
This way the script can split the range into a start and an end value. Also allows the script can easily manage and iterate over the specified port range.

- handling different types of errors

```python
try:
    run_scan(target, port_range)

except KeyboardInterrupt:
    print("\nExiting program.")
    sys.exit()

except socket.gaierror:
    print("Hostname could not be resolved.")
    sys.exit()

except socket.error:
    print("Could not connect to server.")
    sys.exit()
```
This section ensures that the script can respond correctly to issues that could happen. For example issues like interruption by a user or network problems.

---

The final script:

```python
#!/bin/python3

import sys
import socket
from datetime import datetime
import threading
import argparse


def scan_port(target, port):
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(1)  
        result = s.connect_ex((target, port))  
        
        if result == 0:
            try:
                
                banner = s.recv(1024).decode().strip()
                print(f"Port {port} is open | Service: {banner}")
            except:
                print(f"Port {port} is open | Service: Unknown")
        s.close()
    except socket.error:
        print(f"Could not connect to port {port}")
    except Exception as e:
        print(f"Error scanning port {port}: {e}")


def run_scan(target, port_range):
    
    print("-" * 50)
    print(f"Scanning target {target}")
    print("Time started: " + str(datetime.now()))
    print("-" * 50)

    threads = []
    for port in port_range:
        thread = threading.Thread(target=scan_port, args=(target, port))
        threads.append(thread)
        thread.start()

    for thread in threads:
        thread.join()

    print("-" * 50)
    print("Scanning complete.")
    print(f"Time finished: {str(datetime.now())}")


parser = argparse.ArgumentParser(description="A simple multi-threaded port scanner.")
parser.add_argument('-i', '--ip', type=str, required=True, help='Target IP address')
parser.add_argument('-p', '--ports', type=str, required=True, help='Port range to scan (e.g., 20-80)')

args = parser.parse_args()


target = socket.gethostbyname(args.ip)  


port_range = args.ports.split('-')
start_port = int(port_range[0])
end_port = int(port_range[1])
port_range = range(start_port, end_port + 1)


try:
    run_scan(target, port_range)

except KeyboardInterrupt:
    print("\nExiting program.")
    sys.exit()

except socket.gaierror:
    print("Hostname could not be resolved.")
    sys.exit()

except socket.error:
    print("Could not connect to server.")
    sys.exit()

```

**If you notice mistakes or typos or you just have suggestions I would love to hear your feedback and improve at the same time.**
# PRD-in-Docker-via-VSCode
Tutorial about remote debugging of python application in a docker container via VScode

## **Preparations:**
### **Host**
1. Install **ptvsd** python package
```
pip3 install ptvsd
```
2. Configure **launch.json** settings file to support docker container remote debug:
```
{
    "name": "Docker remote debug",
    "type": "python",
    "request": "attach",
    "localRoot": "${workspaceRoot}",
    "remoteRoot": "/folder",
    "port": 3000,
    "secret": "my_secret",
    "host": "localhost"
}
```
**keywords:**
* *host* - The docker container's IP (in case you run the docker container with a '--host' flag, its IP will be like the host, so you can use 'localhost' here).
* *port* - The port of the ptvsd listener in the docker container.
* *secret* - The secret keyword to recognize you in the docker container.
* *remoteRoot* - The folder of the project.

### **Docker container**
When you start the docker container, you need to expose the port outside the container, so you need to supply the *port* flag (in this example: 3000).

## **Usage:**
### PTVSD Commands:
1. *enable_attach* - Enable the remote debugging server in the container (constructed from *secret* and address to bind - IP/Port).
2. *wait_for_attach* - Make the container to wait for the remote debugger (the host) to connect.
3. *break_into_debugger* - Cause the program to hold the execution and pause on the current line (like a breakpoint in the debugger).

---

## **Notes:**
1. *Watch* variables - Unnecessery variables cause to unexpected issues, keep **only** *watch* variables which are relevant to the current execution.
2. *RuntimeError* - Sometimes when you run the 'ptvsd' code (import, enable_attach, etc..) in the middle of the execution, it cause to *RuntimeError*, to fix this you need to the remote debug preparations at the begining of the code.
3. *break_into_debugger* - Sometimes the program will miss the first breakpoint, you cab bypass this isuee by put this command right after the attaching phase in order to break in your first breakpoint correctly.

## **Example**:
### **#1**
```python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--debug_mode', help='Use vscode to debug', action='store_true')
args = parser.parse_args()

def debug_mode(debug):
    if debug:
        import ptvsd
        from colorama import Fore, Style
        print('{c}Enter debug mode'.format(c=Style.BRIGHT + Fore.YELLOW))
        ptvsd.enable_attach('my_secret', address=('127.0.0.1', 3000))
        print('{c}Waiting for attaching...'.format(c=Style.BRIGHT + Fore.YELLOW))
        ptvsd.wait_for_attach()
        print('{c}Attached!'.format(c=Style.BRIGHT + Fore.GREEN))
        print(Style.RESET_ALL)

debug_mode(args.debug_mode)
```
This will break into debug mode when you pass to your program the *--debug_mode* argument.

### **#2:**
```python
def get_hostname():
    import os
    return os.uname()[1]

def debug_mode():
    hostname = get_hostname()
    if hostname == 'your_hostname':
        import ptvsd
        from colorama import Fore, Style
        print('{c}Enter debug mode'.format(c=Style.BRIGHT + Fore.YELLOW))
        ptvsd.enable_attach('my_secret', address=('127.0.0.1', 3000))
        print('{c}Waiting for attaching...'.format(c=Style.BRIGHT + Fore.YELLOW))
        ptvsd.wait_for_attach()
        print('{c}Attached!'.format(c=Style.BRIGHT + Fore.GREEN))
        print(Style.RESET_ALL)
        ptvsd.break_into_debugger()

debug_mode()
```
1. Will break into debug mode only when you run the app on the machine with the specified hostname (good for test when you cant supply argument from cli).
2. *break_into_debugger* - will break the program execution at this point.

Good luck & have fun :)

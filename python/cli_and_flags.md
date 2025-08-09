# Python CLI Commands & Flags Cheatsheet

## Basic Python Command Structure
```bash
python [options] [-c cmd | -m mod | file | -] [args]
```

## Essential Commands

### Running Python Code
```bash
python script.py              # Run a Python script
python -c "print('Hello')"    # Execute Python code directly
python -m module_name         # Run a module as a script
python -                      # Read from stdin
python -i script.py           # Run script then enter interactive mode
```

### Interactive Mode
```bash
python                        # Start Python REPL
python -i                     # Start interactive mode after script
python -q                     # Quiet mode (no version/copyright info)
```

## Important Flags & Options

### Code Execution
| Flag | Description | Example |
|------|-------------|---------|
| `-c` | Execute Python code from command line | `python -c "import sys; print(sys.version)"` |
| `-m` | Run module as script | `python -m http.server 8000` |
| `-i` | Interactive mode after script execution | `python -i myscript.py` |

### Output & Verbosity
| Flag | Description | Example |
|------|-------------|---------|
| `-v` | Verbose (trace import statements) | `python -v script.py` |
| `-q` | Quiet (suppress version/copyright) | `python -q` |
| `-u` | Unbuffered stdout/stderr | `python -u script.py` |
| `-s` | Don't add user site directory to sys.path | `python -s script.py` |

### Warnings & Debugging
| Flag | Description | Example |
|------|-------------|---------|
| `-W` | Warning control | `python -W ignore script.py` |
| `-d` | Debug output from parser | `python -d script.py` |
| `-O` | Basic optimizations | `python -O script.py` |
| `-OO` | Remove docstrings too | `python -OO script.py` |
| `-B` | Don't create .pyc files | `python -B script.py` |

### Path & Environment
| Flag | Description | Example |
|------|-------------|---------|
| `-S` | Don't run site.py on startup | `python -S script.py` |
| `-E` | Ignore environment variables | `python -E script.py` |
| `-I` | Isolated mode (implies -E and -s) | `python -I script.py` |
| `-P` | Safe path (don't prepend cwd to sys.path) | `python -P script.py` |

### Encoding
| Flag | Description | Example |
|------|-------------|---------|
| `-X` | Set implementation-specific options | `python -X dev script.py` |

## Common Module Commands

### Package Management
```bash
python -m pip install package     # Install package
python -m pip list               # List installed packages  
python -m pip show package       # Show package info
python -m pip freeze             # List all packages with versions
python -m pip uninstall package  # Uninstall package
```

### Built-in Modules
```bash
python -m http.server 8000       # Start HTTP server on port 8000
python -m json.tool file.json    # Pretty-print JSON
python -m pdb script.py          # Run script with debugger
python -m cProfile script.py    # Profile script performance
python -m timeit "code"          # Time small code snippets
python -m compileall directory   # Compile .py files to .pyc
python -m doctest script.py     # Run doctests
python -m unittest discover     # Run unit tests
python -m venv myenv            # Create virtual environment
python -m zipapp myapp          # Create runnable ZIP archives
python -m pydoc module_name     # Show documentation in console
python -m pydoc -b              # Start documentation HTTP server
python -m ensurepip             # Install pip if missing
python -m site                  # Show Python's module search paths
python -m dis script.py         # Disassemble Python bytecode
python -m pstats profile_output # Analyze cProfile output
python -m asyncio               # Debug asyncio loop (interactive)
python -m xml.etree.ElementTree file.xml  # Parse & pretty-print XML
python -m turtle                # Launch turtle graphics window
python -m this                  # Display Zen of Python
python -m antigravity           # XKCD easter egg (opens browser)
```

### Legacy/Deprecated Modules
```bash
python -m smtpd -c DebuggingServer localhost:1025  # Debug SMTP server (≤3.8)
```

### Development Tools
```bash
python -m black script.py       # Format code (if black installed)
python -m flake8 script.py      # Lint code (if flake8 installed)
python -m pytest               # Run tests (if pytest installed)
python -m mypy script.py        # Type checking (if mypy installed)
```

## Environment & System Info

### Version Information
```bash
python --version                 # Show Python version
python -V                       # Show Python version (short)
python -c "import sys; print(sys.version_info)"
```

### System Information
```bash
python -c "import sys; print(sys.executable)"      # Python executable path
python -c "import sys; print(sys.path)"           # Python path
python -c "import platform; print(platform.platform())"  # Platform info
```

## Warning Control Examples

### Warning Filters
```bash
python -W error script.py        # Turn warnings into errors
python -W ignore script.py       # Ignore all warnings
python -W default script.py      # Default warning behavior
python -W once script.py         # Show warning once per location
python -W module script.py       # Show warning once per module
python -W "ignore::DeprecationWarning" script.py  # Ignore specific warning type
```

## Advanced Usage

### Multiple Flags
```bash
python -Bqu script.py            # No .pyc files, quiet, unbuffered
python -OO -W ignore script.py   # Optimize, remove docstrings, ignore warnings
```

### Environment Variables (Alternative to flags)
```bash
PYTHONPATH=/path/to/modules python script.py      # Set module search path
PYTHONDONTWRITEBYTECODE=1 python script.py        # Don't create .pyc files
PYTHONUNBUFFERED=1 python script.py               # Unbuffered output
PYTHONOPTIMIZE=1 python script.py                 # Optimize (like -O)
```

### Development Mode
```bash
python -X dev script.py          # Development mode (extra checks)
python -X faulthandler script.py # Enable fault handler
python -X importtime script.py   # Show import timing
```

## Quick Reference Card

| Task | Command |
|------|---------|
| Run script | `python script.py` |
| Interactive after script | `python -i script.py` |
| Execute string | `python -c "code"` |
| Run module | `python -m module` |
| No bytecode files | `python -B script.py` |
| Unbuffered output | `python -u script.py` |
| Ignore warnings | `python -W ignore script.py` |
| Start HTTP server | `python -m http.server` |
| Install package | `python -m pip install pkg` |
| Run debugger | `python -m pdb script.py` |
| Format JSON | `python -m json.tool file.json` |
| Create venv | `python -m venv env_name` |
| Profile code | `python -m cProfile script.py` |
| Compile to bytecode | `python -m compileall dir/` |
| Create ZIP app | `python -m zipapp myapp` |
| Show docs | `python -m pydoc module` |
| Check module paths | `python -m site` |
| Disassemble bytecode | `python -m dis script.py` |
| Turtle graphics | `python -m turtle` |
| Zen of Python | `python -m this` |

## Tips

- Use `python3` instead of `python` on systems with both Python 2 and 3
- Combine flags: `python -Bqu` for no bytecode, quiet, unbuffered
- Use `python -X dev` during development for extra debugging features
- Set `PYTHONDONTWRITEBYTECODE=1` to globally prevent .pyc file creation
- Use `python -m pip` instead of just `pip` to ensure you're using the right Python installation
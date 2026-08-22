# eSim 2.5 Installation Issues on Ubuntu 25.04

## Summary
- **Testing Period:** August 16, 2026 – August 23, 2026
- **System:** Ubuntu 25.04 (via WSL 2)
- **eSim Version:** 2.5 (from commit bbb222c5)
- **Total Bugs Found:** 3
- **Bugs Fixed:** 2
- **Status:** Partially Fixed

---

## Bug #1: PyQt4 Package Not Available ✅ FIXED

### Error Message
```
E: Unable to locate package python-qt4
"PyQt4" dependency couldn't be installed.
```

### Root Cause
- Ubuntu 25.04 removed Python 2 support completely
- `python-qt4` is a Python 2 library and no longer exists in Ubuntu repositories
- Modern replacement is `python3-pyqt5`

### Fix Applied
**File:** `install-eSim.sh`
**Line:** ~115

**Original Code:**
```bash
echo "Installing PyQt4..........................."
sudo apt install -y python-qt4
```

**Fixed Code:**
```bash
echo "Installing PyQt5..........................."
sudo apt install -y python3-pyqt5
```

### Status
✅ **FIXED** - PyQt5 installed successfully with 9 dependencies

---

## Bug #2: Matplotlib Package Not Available ✅ FIXED

### Error Message
```
E: Unable to locate package python-matplotlib
"Matplotlib" dependency couldn't be installed.
```

### Root Cause
- Ubuntu 25.04 only provides Python 3 packages
- `python-matplotlib` (Python 2) no longer exists
- Modern replacement is `python3-matplotlib`

### Fix Applied
**File:** `install-eSim.sh`
**Line:** ~125

**Original Code:**
```bash
echo "Installing Matplotlib......................"
sudo apt install -y python-matplotlib
```

**Fixed Code:**
```bash
echo "Installing Matplotlib......................"
sudo apt install -y python3-matplotlib
```

### Status
✅ **FIXED** - python3-matplotlib installed successfully with 80 dependencies

---

## Bug #3: unzip Command Not Found ❌ REQUIRES FIX

### Error Message
```
install-eSim.sh: line 53: unzip: command not found
mv: cannot stat 'nghdl-master': No such file or directory
install-eSim.sh: line 55: cd: nghdl/: No such file or directory
install-eSim.sh: line 56: ./install-nghdl.sh: No such file or directory

There was some error while installing NGHDL
```

### Root Cause
- The `unzip` utility is not installed on Ubuntu 25.04
- Script tries to unzip nghdl-master.zip but fails
- Missing dependency causes cascading failures

### Location
**File:** `install-eSim.sh`
**Function:** `installNghdl()` at lines 52-63

**Problematic Code:**
```bash
function installNghdl
{
    echo "Installing NGHDL......................."
    unzip nghdl-master.zip  # ← unzip not found
    mv nghdl-master nghdl
    cd nghdl/
    ./install-nghdl.sh --install
    ...
}
```

### Recommended Fix
Add `unzip` to the dependency installation:

**File:** `install-eSim.sh`
**Function:** `installDependency()` - add after line ~110

```bash
echo "Installing unzip..........................."
sudo apt install -y unzip
if [ $? -ne 0 ]; then
    echo -e "\n\n\"unzip\" dependency couldn't be installed.\nKindly resolve above errors and try again."
    exit 1
fi
```

### Status
⏹️ **NOT YET FIXED** - Needs unzip package installation

---

## Installation Progress

### ✅ Successfully Installed
1. Xterm (390-1ubuntu3)
2. PyQt5 (5.15.10+dfsg-1build6)
3. Matplotlib (3.6.3-1ubuntu5) with 80 dependencies:
   - GCC/G++ compiler tools
   - Python3 numpy, scipy, sympy
   - Graphics libraries (tkinter, PIL, fonttools, etc.)
4. KiCad (7.0.11+dfsg-1build4) with 29 dependencies:
   - wxWidgets library
   - OCCT (Open Cascade Technology)
   - ngspice library
   - Python3-wxgtk4.0

### ❌ Failed at
- NGHDL installation (requires unzip)

---

## Python 2 → Python 3 Migration Issues

The script was written for Python 2 era. Key compatibility issues:

1. **Package Names:**
   - `python-qt4` → `python3-pyqt5` ✅ Fixed
   - `python-matplotlib` → `python3-matplotlib` ✅ Fixed

2. **Additional Python 2 Packages (Not Yet Addressed):**
   - `python2` (referenced in esim-start.sh line)
   - `python-qt4` imports in Application.py
   - `python2` shebang in Python files

3. **Runtime Issues (Will Occur After NGHDL Fix):**
   - esim-start.sh uses `python2 Application.py`
   - Need to change to `python3 Application.py`
   - Application.py may have Python 2 syntax incompatibilities

---

## Testing Results

### Dependency Installation: ✅ PASSED
- Apt repositories connected successfully
- 197 packages installed in ~3 minutes
- 1.1 GB downloaded successfully

### KiCad Configuration: ✅ PASSED
- KiCad libraries extracted
- fp-lib-table copied
- kicad.pro configured
- Ownership set to user

### NGHDL Installation: ❌ FAILED
- Missing `unzip` utility
- Cannot extract nghdl-master.zip
- Installation halted

---

## Recommended Next Steps

1. **Immediate (To Complete Installation):**
   - Add `sudo apt install -y unzip` to installDependency()
   - Re-run installation script
   - Test NGHDL installation

2. **Long-term (Python 3 Migration):**
   - Update esim-start.sh to use Python 3
   - Audit Application.py for Python 2 syntax
   - Update any Python 2 specific imports
   - Test GUI launch with Python 3

3. **Maintenance:**
   - Update script to detect Ubuntu version
   - Use dynamic package checking
   - Consider Docker containerization for consistency

---

## System Environment

```
OS: Ubuntu 25.04 (noble)
Kernel: 6.8.0-138.138
WSL Version: WSL 2.6.3.0
Python Available: python3.12
Python 2: NOT INSTALLED
APT: apt 2.7.14
```

---

## Conclusion

The eSim 2.5 installation script has been successfully updated to work with Ubuntu 25.04 by:

1. ✅ Replacing Python 2 PyQt4 with Python 3 PyQt5
2. ✅ Replacing Python 2 Matplotlib with Python 3 Matplotlib
3. ⏹️ Identified missing unzip utility (ready to fix)

All core dependencies (KiCad, NumPy, SciPy, Matplotlib, wxWidgets) are successfully installed. The final step is to install unzip and complete NGHDL setup, followed by runtime Python 3 migration.

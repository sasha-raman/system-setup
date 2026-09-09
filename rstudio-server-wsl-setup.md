# RStudio Server on WSL 2

Run RStudio as a browser-based server inside WSL 2. This gives you a native Linux R environment while you work from Windows through `http://localhost:8787`.

## Why RStudio Server in WSL

- Projects live on the Linux filesystem, so file paths, permissions, and package compilation work correctly.
- No `\\wsl$\` network-path overhead or file-watching issues.
- The browser UI is functionally identical to the desktop IDE for editing, plots, packages, and the console.

> **Note:** Posit does not officially support RStudio Server on WSL. These instructions are for a single-user, open-source setup. Multi-user or production deployments should use Posit Workbench.

## Prerequisites

- Windows 10 version 2004 (build 19041) or later, or Windows 11.
- WSL 2 with Ubuntu installed and systemd enabled (see `wsl-ubuntu-setup.md`).
- `networkingMode=mirrored` in `%USERPROFILE%\.wslconfig` is recommended so `localhost` is shared cleanly between Windows and WSL.

## 1. Install R

Open the Ubuntu terminal and add the CRAN repository for the latest R version (Ubuntu 24.04 / noble):

```bash
# Add CRAN GPG key and repository
curl -fsSL https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc \
  | sudo gpg --dearmor -o /usr/share/keyrings/cran-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/cran-archive-keyring.gpg] https://cloud.r-project.org/bin/linux/ubuntu noble-cran40/" \
  | sudo tee /etc/apt/sources.list.d/cran.list

sudo apt update
```

Install R and common build dependencies used by RStudio Server and `devtools`:

```bash
sudo apt install -y \
  r-base r-base-core r-recommended r-base-dev \
  gdebi-core build-essential \
  libcurl4-openssl-dev libxml2-dev libssl-dev
```

Verify R:

```bash
R --version
```

## 2. Install RStudio Server

Download and install the latest stable RStudio Server for Ubuntu:

```bash
cd /tmp
wget https://rstudio.org/download/latest/stable/server/bionic/rstudio-server-latest-amd64.deb \
  -O rstudio-server-latest-amd64.deb

sudo gdebi rstudio-server-latest-amd64.deb
```

> The URL contains `bionic` because Posit uses a single redirect for the latest stable server build. It installs cleanly on Ubuntu 22.04, 24.04, and later. If you prefer a distro-specific build, visit https://posit.co/download/rstudio-server/ and replace the `.deb` URL.

## 3. Start and Access RStudio Server

Start the server manually:

```bash
sudo rstudio-server start
```

Or enable it to start automatically via systemd (recommended if systemd is enabled in WSL):

```bash
sudo systemctl enable --now rstudio-server
```

Open a browser on Windows and go to:

```
http://localhost:8787
```

Log in with your WSL Ubuntu username and password.

If you restart WSL and did not enable systemd, start the server again with:

```bash
sudo rstudio-server start
```

## 4. Create an R Project

Inside WSL, create a project directory and an `.Rproj` file:

```bash
mkdir -p ~/projects/my-analysis
cd ~/projects/my-analysis
cat > my-analysis.Rproj <<'EOF'
Version: 1.0

RestoreWorkspace: Default
SaveWorkspace: Default
AlwaysSaveHistory: Default

EnableCodeIndexing: Yes
UseSpacesForTab: Yes
NumSpacesForTab: 2
Encoding: UTF-8

RnwWeave: Sweave
LaTeX: pdfLaTeX
EOF
```

In RStudio Server, click **File → Open Project**, navigate to `/home/<username>/projects/my-analysis`, and open `my-analysis.Rproj`.

Keep active projects inside the WSL filesystem. Avoid `/mnt/c` for ongoing work because cross-OS file access is slow and permissions are synthetic.

## 5. Debugging in RStudio Server

RStudio Server includes the same interactive debugger as the desktop IDE. You access it through the browser at `http://localhost:8787`.

### Interactive debugger in the browser

1. Open an R script.
2. Click in the left gutter next to a line number to set a breakpoint.
3. Source the file or run the function.
4. Execution pauses at the breakpoint. Use the toolbar buttons or keyboard shortcuts to step over, step into, step out, or continue.

### In-code breakpoints with `browser()`

Insert `browser()` anywhere in a function to pause execution when that line is reached:

```r
my_function <- function(x) {
  y <- x * 2
  browser()
  z <- y + 10
  return(z)
}

my_function(5)
```

When execution pauses, the R console becomes an interactive browser where you can inspect variables and run expressions.

### Function-level debugging with `debug()` and `debugonce()`

Debug a function every time it runs:

```r
debug(my_function)
my_function(5)
```

Debug a function only on the next call:

```r
debugonce(my_function)
my_function(5)
```

Remove the debug flag:

```r
undebug(my_function)
```

### Post-error debugging

To enter the debugger automatically after an error:

```r
options(error = recover)
```

When an error occurs, R shows a menu of frames you can inspect. Reset with:

```r
options(error = NULL)
```

For a stack trace after an error without entering the debugger:

```r
traceback()
```

### Modify functions with `trace()`

Add tracing or temporary breakpoints to existing functions:

```r
trace(my_function, browser, at = 3)
untrace(my_function)
```

### Debugging compiled code

For C/C++ code called from R, use `gdb` or `valgrind`:

```bash
# Start R under gdb
R -d gdb

# Inside gdb, run R
run
```

Or debug memory issues:

```bash
R -d valgrind --args -f my_script.R
```

This requires the compiled code to be built with debug symbols.

## 6. Remote Access from Another Machine

By default RStudio Server listens only on `localhost`. If you want to reach it from another machine on your network, create `/etc/rstudio/rserver.conf`:

```ini
# Not recommended without authentication hardening
www-address=0.0.0.0
```

Then restart:

```bash
sudo rstudio-server restart
```

A safer pattern is to leave the server bound to `localhost` and tunnel port 8787 over SSH from the remote machine:

```bash
ssh -L 8787:localhost:8787 user@windows-host
```

Then browse to `http://localhost:8787` on the remote machine.

## 7. Troubleshooting

### RStudio Server does not start after WSL restart

If you did not enable the systemd service, start it manually:

```bash
sudo rstudio-server start
```

### Port 8787 is already in use

Find and stop the existing process, or configure RStudio Server to use a different port in `/etc/rstudio/rserver.conf`:

```ini
www-port=8788
```

Then restart:

```bash
sudo rstudio-server restart
```

### Cannot connect from Windows

- Verify WSL is running and `systemd` is enabled (`systemctl is-system-running`).
- Confirm `rstudio-server` is running (`sudo rstudio-server status`).
- If using `networkingMode=mirrored`, `localhost` is shared. Otherwise use `http://127.0.0.1:8787` or the WSL IP from `ip addr`.

### Missing system dependencies for R packages

Many R packages need headers and libraries. Install the common ones:

```bash
sudo apt install -y libxml2-dev libcurl4-openssl-dev libssl-dev libfontconfig1-dev \
  libharfbuzz-dev libfribidi-dev libfreetype6-dev libpng-dev libtiff5-dev libjpeg-dev
```

## Notes

- R does not have a standalone "remote debugger protocol" like Python's `debugpy`. The browser-based RStudio Server session *is* the remote interface: the R process runs in WSL and you interact with it through the web UI, including breakpoints, stepping, and the interactive console.
- For Posit-supported production or multi-user deployments, evaluate Posit Workbench instead of the open-source RStudio Server on WSL.

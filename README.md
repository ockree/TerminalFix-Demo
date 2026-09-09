# TerminalFix Awareness Demo

A small security-awareness and detection-validation demonstration of the **TerminalFix** social-engineering technique.

This repository contains a browser page that presents a simulated verification challenge and instructs the user to open Windows Terminal using:

1. <kbd>Win</kbd> + <kbd>X</kbd>
2. Press <kbd>I</kbd> to open Windows Terminal
3. Press <kbd>Ctrl</kbd> + <kbd>V</kbd> to paste
4. Press <kbd>Enter</kbd> to execute

The copied command is deliberately benign. It creates a batch file, invokes that file through `cmd.exe`, writes a local validation marker, and opens Windows Calculator as the proof of execution.

> **Important:** This project is an authorized security-awareness and defensive-testing demonstration. Do not deploy it as a real verification page or present it to users outside an approved exercise.

## What is TerminalFix?

TerminalFix is a social-engineering technique related to the broader family of **ClickFix** attacks.

Instead of exploiting a software vulnerability, the technique convinces a user to manually execute an attacker-supplied command. A lure may claim that a CAPTCHA, browser check, document, media file, or security verification has failed and that a terminal command is required to resolve the problem.

A typical flow is:

1. A user visits a malicious or compromised webpage.
2. The page displays a fake verification or troubleshooting prompt.
3. JavaScript places a command in the user's clipboard.
4. The user is instructed to open PowerShell or Windows Terminal.
5. The user pastes and executes the command.
6. In a real attack, the command may download payloads, establish persistence, run malware, or create a remote-access channel.

The technique crosses an important security boundary: the browser itself does not directly execute the operating-system command. The user is manipulated into transferring the command from the browser into a trusted local shell.

## What this demo does

The demo reproduces the visible interaction pattern without downloading malware, establishing persistence, changing security settings, or creating a network connection.

When the user selects **Copy verification command**, the page copies a PowerShell command to the clipboard. If the user pastes and executes it, the command:

1. Creates this directory:

   ```text
   C:\ProgramData\TerminalFixTraining
   ```

2. Creates this batch file:

   ```text
   C:\ProgramData\TerminalFixTraining\1.bat
   ```

3. Invokes the batch file through:

   ```text
   C:\Windows\System32\cmd.exe /d /c
   ```

4. Passes Windows Calculator as the batch file's first argument:

   ```text
   C:\Windows\System32\calc.exe
   ```

5. Writes this validation marker:

   ```text
   C:\ProgramData\TerminalFixTraining\executed.txt
   ```

6. Opens Windows Calculator as a visible proof of execution.

The demo does **not**:

- Download or retrieve remote content
- Execute an unknown binary
- Establish persistence
- Modify security controls
- Collect credentials or user data
- Open a reverse shell or tunnel
- Contact a command-and-control service
- Transmit telemetry back to the page

## Execution chain

The intentionally visible execution chain is:

```text
Browser
  └─ User copies and pastes PowerShell
      └─ powershell.exe
          ├─ Creates C:\ProgramData\TerminalFixTraining\1.bat
          └─ Starts cmd.exe /d /c
              └─ Executes 1.bat
                  ├─ Writes executed.txt
                  └─ Starts calc.exe
```

Although the payload is harmless, this process chain gives security teams representative telemetry for testing detections around browser-assisted command execution and script-interpreter chaining.

## Repository contents

A minimal repository can use the following layout:

```text
.
├── index.html
├── README.md
└── LICENSE
```

- `index.html` contains the simulation page and clipboard command.
- `README.md` explains the technique and expected behavior.
- `LICENSE` defines how the demonstration may be reused.

## Run locally

Some browsers restrict clipboard access for pages opened directly with a `file://` URL. Serving the page from `localhost` provides more consistent Clipboard API behavior.

From the repository directory:

```powershell
python -m http.server 8080
Start-Process "http://localhost:8080/"
```

Alternatively, with Python on Linux or macOS:

```bash
python3 -m http.server 8080
```

Then browse to:

```text
http://localhost:8080/
```

## Publish with GitHub Pages

1. Push `index.html` and `README.md` to a GitHub repository.
2. Open the repository's **Settings**.
3. Select **Pages**.
4. Under **Build and deployment**, select **Deploy from a branch**.
5. Select the branch containing the demo, typically `main`.
6. Select `/ (root)` as the directory.
7. Save the configuration.

GitHub will display the resulting Pages URL after deployment.

### Recommended repository description

```text
Benign TerminalFix security-awareness and detection-validation demonstration.
```

### Recommended repository topics

```text
terminalfix
clickfix
security-awareness
social-engineering
powershell
detection-engineering
purple-team
```

## Validate execution

After completing the demo, inspect the generated batch file and marker:

```powershell
Get-Content "$env:ProgramData\TerminalFixTraining\1.bat"
Get-Content "$env:ProgramData\TerminalFixTraining\executed.txt"
```

Check whether Calculator is running:

```powershell
Get-Process calc, CalculatorApp -ErrorAction SilentlyContinue
```

Depending on the Windows version, Calculator may transition from `calc.exe` to the packaged Calculator application after launch.

## Cleanup

Close Calculator and remove the training artifacts:

```powershell
Get-Process calc, CalculatorApp -ErrorAction SilentlyContinue |
    Stop-Process -Force -ErrorAction SilentlyContinue

Remove-Item "$env:ProgramData\TerminalFixTraining" `
    -Recurse -Force -ErrorAction SilentlyContinue
```

The demo does not install anything and does not configure persistence, so no additional cleanup should be required.

## Defensive testing opportunities

This demo can help validate visibility into behaviors such as:

- A browser placing shell content onto the clipboard
- Interactive PowerShell execution
- PowerShell creating a `.bat` file under `C:\ProgramData`
- `powershell.exe` spawning `cmd.exe`
- `cmd.exe` executing a newly created batch file
- A batch file launching another process
- Script-created files in an unusual `ProgramData` subdirectory
- User execution following a browser-based verification prompt

Useful Windows telemetry may include:

- EDR process and file events
- PowerShell Script Block Logging, Event ID `4104`
- PowerShell Module Logging, Event ID `4103`
- Security process creation, Event ID `4688`
- Sysmon process creation, Event ID `1`
- Sysmon file creation, Event ID `11`
- Browser history or proxy records associated with the demonstration page

Exact event availability depends on local audit policy, PowerShell logging configuration, EDR coverage, and browser telemetry.

## Safety and deployment guidance

For an approved exercise:

- Use a clearly defined scope and testing window.
- Notify the appropriate exercise owners and support teams.
- Keep the payload benign and independently review it before deployment.
- Do not collect credentials, clipboard contents, or unrelated user data.
- Do not add downloads, persistence, evasion, remote access, or outbound callbacks.
- Provide participants with a reporting and debriefing path.
- Remove the public page after the exercise if continued hosting is unnecessary.
- Avoid using a production organization's domain or branding without approval.

The page includes a visible security-awareness simulation banner and exposes the copied command for inspection. Preserve those safeguards if the repository is published publicly.

## Branding notice

The visual design is intended to demonstrate how attackers imitate familiar verification pages. It is not a real CAPTCHA or security check and is not affiliated with, endorsed by, or operated by Cloudflare or Microsoft.

For public hosting, consider replacing third-party names and visual elements with neutral fictional branding while retaining the interaction pattern.

## ATT&CK context

Depending on how a real intrusion implements the technique, relevant MITRE ATT&CK concepts can include:

- **T1204.002 — User Execution: Malicious File**
- **T1059.001 — Command and Scripting Interpreter: PowerShell**
- **T1059.003 — Command and Scripting Interpreter: Windows Command Shell**

The precise mapping depends on the delivered command and subsequent payload behavior. This demonstration exercises PowerShell and Windows Command Shell execution but does not deliver a malicious payload.

## Disclaimer

This repository is provided for controlled security-awareness exercises, defensive research, purple-team validation, and education. Operators are responsible for obtaining authorization and following their applicable rules of engagement, code of conduct, privacy requirements, and organizational policies.

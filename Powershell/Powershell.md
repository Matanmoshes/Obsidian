


---


---



---

# PowerShell Variables
## code

```powershell

#https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_variables?view=powershell-6
#https://docs.microsoft.com/en-us/powershell/scripting/learn/using-variables-to-store-objects?view=powershell-6
#____________________________________________________________
Get-Process
$processes = Get-Process
$processes
#----------------------------------------
# not using variable
Get-Process | Where-Object { $_.CPU -gt 5000 } #find processes keeping the CPU busy
Get-Process | Sort-Object WorkingSet64 -Descending #sort processes by memory usage
#----------------------------------------
# using variable
$processes = Get-Process
$processes | Where-Object { $_.CPU -gt 5000 } #find processes keeping the CPU busy
$processes | Sort-Object WorkingSet64 -Descending #sort processes by memory usage
#----------------------------------------
# not strong typed
$myNewVariable

# data types
#----------------------------------------
$total = 2 + 2
$total
$total | Get-Member
#----------------------------------------
$total = '2 + 2'
$total
$total | Get-Member
#____________________________________________________________
# understanding data types

$num1 = 2
$num2 = 2
$total = $num1 + $num2
$total
#----------------------------------------
$num1 = '2'
$num2 = '2'
$total = $num1 + $num2
$total
#----------------------------------------

#strong type
#----------------------------------------
[int]$num1 = '2'
[int]$num2 = '2'
$total = $num1 + $num2
#----------------------------------------

#convert
#----------------------------------------
$stringReturn = $total.ToString()
$total | Get-Member
#----------------------------------------

#quotes
$literal = 'Two plus one equals: $(1 + 2)'
$literal
$escaped = "Two plus one equals: $(1 + 2)"
$escaped
Write-Host '$escaped'
Write-Host "$escaped"

#constant variables are reserved
Get-Variable
# you can't use these
$HOME = 'c:\test'

#environment variables
Get-ChildItem env:
$env:COMPUTERNAME
$env:USERNAME

#putting it all together with an example
$path = Read-Host -Prompt 'Please enter the file path you wish to scan for large files...'
$rawFileData = Get-ChildItem -Path $path -Recurse
$largeFiles = $rawFileData | Where-Object { $_.Length -gt 100MB }
$largeFilesCount = $largeFiles | Measure-Object | Select-Object -ExpandProperty Count
Write-Host "You have $largeFilesCount large file(s) in $path"
#____________________________________________________________
```
---
## Explanation

Your script includes a wide range of PowerShell concepts, from handling variables and data types to using cmdlets for system information and file management. Let's break down these concepts and examine the different parts of your script:

### Basic Cmdlets and Variables
- `Get-Process` retrieves all currently running processes.
- `$processes = Get-Process` stores the output of `Get-Process` in the variable `$processes` for further use, reducing the need to repeatedly call `Get-Process`.

### Using Variables with Cmdlets
- Variables like `$processes` are used with pipeline operations to filter or sort data. For example:
  - `$processes | Where-Object { $_.CPU -gt 5000 }` finds processes using more than 5000 CPU units.
  - `$processes | Sort-Object WorkingSet64 -Descending` sorts processes by memory usage in descending order.

### Understanding Data Types
- PowerShell is not strongly typed by default, meaning you can assign different types of data to the same variable without explicit declarations.
- **Example with integers**:
  - `$num1 = 2; $num2 = 2; $total = $num1 + $num2` results in `$total` being `4`.
- **Example with strings**:
  - `$num1 = '2'; $num2 = '2'; $total = $num1 + $num2` results in `$total` being `'22'`, illustrating string concatenation instead of arithmetic addition.

### Strong Typing
- You can enforce data types by declaring a type:
  - `[int]$num1 = '2'` ensures `$num1` is treated as an integer.

### Conversions
- Converting data types is straightforward:
  - `$stringReturn = $total.ToString()` converts `$total` to a string.
- The `Get-Member` cmdlet reveals the methods and properties associated with a variable or object.

### String Interpolation and Quotes
- Single quotes (`'`) are used for literals.
- Double quotes (`"`) are used for interpolation (expanding variables and expressions within strings).
- **Example**:
  - `$literal = 'Two plus one equals: $(1 + 2)'` does not expand the expression.
  - `$escaped = "Two plus one equals: $(1 + 2)"` evaluates the expression, displaying "Two plus one equals: 3".

### Environment Variables and Constants
- Environment variables like `$env:COMPUTERNAME` can be accessed directly.
- Constants like `$HOME` should not be reassigned as they are reserved.

### Practical Application Example
- The script prompts for a file path and scans for large files (greater than 100MB), counting them and displaying the count.
  - `Read-Host` captures user input.
  - `Get-ChildItem -Path $path -Recurse` retrieves files and directories.
  - The files are filtered and counted, and the results are displayed using `Write-Host`.

This example demonstrates powerful ways to use PowerShell for automation and system management tasks, highlighting its flexibility and capability in handling various data types and operations seamlessly. 

---
# PowerShell Logic
## code
 
``` powershell

##################################################
#CONDITIONAL
# do something based on a condition
if ('a condition is met') {
    # do something / take an action
}
elseif ('a different condition is met') {
    # do something different
}
else {
    # do something else
}

#-------------------------------------------------

$path = 'C:\Test' #windows
# $path = /home #linux
$evalPath = Test-Path $path
if ($evalPath -eq $true) {
    Write-Host "$path VERIFIED"
}
elseif ($evalPath -eq $false) {
    Write-Host "$path NOT VERIFIED"
}

#COMPARISON OPERATORS
##################################################
#SWITCH
[int]$aValue = Read-Host 'Enter a number'
switch ($aValue) {
    1 {
        Write-Host 'You entered the number ONE'
    }
    2 {
        Write-Host 'You entered the number TWO'
    }
    3 {
        Write-Host 'You entered the number THREE'
    }
    4 {
        Write-Host 'You entered the number FOUR'
    }
    5 {
        Write-Host 'You entered the number FIVE'
    }
    Default {
        Write-Host "Sorry, I don't know what to do with $aValue"
    }
}

##################################################
# LOOPS
for ($i = 0; $i -le 15; $i++) {
    Write-Host $i -ForegroundColor $i
}

#-------------------------------------------------

$aString = 'Jean-Luc Picard'
$reverseString = ''
for ($i = $aString.Length; $i -ge 0; $i--) {
    $reverseString += $aString[$i]
}
$reverseString

##################################################

$path = 'C:\Test'
[int]$totalSize = 0
$fileInfo = Get-ChildItem $path -Recurse
foreach ($file in $fileInfo) {
    $totalSize += $file.Length
}
Write-Host "The total size of file in $path is $($totalSize /1MB) MB."

##################################################

#-------------------------------------------------
# do while loop
$pathVerified = $false
do {
    # in a do while, the user will always be prompted at least once
    $path = Read-Host 'Please enter a file path to evaluate'
    if (Test-Path $path) {
        $pathVerified = $true
    }
} while ($pathVerified -eq $false)
#-------------------------------------------------
# while loop
$pathVerified = $true
while ($pathVerified -eq $false) {
    # in a while loop, you might never enter the loop
    $path = Read-Host 'Please enter a file path to evaluate'
    if (Test-Path $path) {
        $pathVerified = $true
    }
}
#-------------------------------------------------

$largeProcesses = Get-Process | Where-Object { $_.WorkingSet64 -gt 50MB }

$largeProcesses = @()
$processes = Get-Process
foreach ($process in $processes) {
    if ($process.WorkingSet64 -gt 50MB) {
        $largeProcesses += $process
    }
}

#-------------------------------------------------

$path = 'C:\Test'
$folderCount = 0
Get-ChildItem $path | ForEach-Object -Process { if ($_.PSIsContainer) { $folderCount++ } }
$folderCount

#-------------------------------------------------

# declare a few variables for counting
[int]$fileCount = 0
[int]$folderCount = 0
[int]$totalSize = 0

# declare our path we want to evaluate
$path = 'C:\Test'

# get the file information
$rawFileInfo = Get-ChildItem $path -Recurse

# loop through that file information
foreach ($item in $rawFileInfo) {
    if ($item.PSIsContainer) {
        # this is a folder/directory
        $folderCount++
    }
    else {
        # this is a file, because it is not a PSIsContainer
        $fileCount++
        $totalSize += $item.Length
    }
}

# generate output
Write-Host "Breakdown of $path"
Write-Host "Total Directories: $folderCount"
Write-Host "Total Files: $fileCount"
Write-Host "Total Size of files: $($totalSize / 1MB) MB"

#-------------------------------------------------

```
## Explanation

### Conditional Statements (`if`, `elseif`, `else`)
Your script uses basic conditional structures to check if a specific file path exists:

```powershell
$path = 'C:\Test' #windows
$evalPath = Test-Path $path
if ($evalPath -eq $true) {
    Write-Host "$path VERIFIED"
}
elseif ($evalPath -eq $false) {
    Write-Host "$path NOT VERIFIED"
}
```
This effectively checks whether `$path` exists and prints a message accordingly.

### Switch Statement
You've implemented a switch statement that reacts based on the user's input:

```powershell
[int]$aValue = Read-Host 'Enter a number'
switch ($aValue) {
    1 { Write-Host 'You entered the number ONE' }
    2 { Write-Host 'You entered the number TWO' }
    ... and so on ...
    Default { Write-Host "Sorry, I don't know what to do with $aValue" }
}
```
This is a neat way of handling multiple conditional branches, providing a specific output for each recognized input and a default response otherwise.

### Loops
You've demonstrated the use of `for`, `foreach`, and `do-while` loops, showing different applications like printing sequences, reversing strings, calculating total file sizes, and verifying paths:

- **For Loop** (Color output and string reversal):
  ```powershell
  for ($i = 0; $i -le 15; $i++) {
      Write-Host $i -ForegroundColor $i
  }

  $aString = 'Jean-Luc Picard'
  $reverseString = ''
  for ($i = $aString.Length; $i -ge 0; $i--) {
      $reverseString += $aString[$i]
  }
  ```

- **Foreach Loop** (Summing file sizes):
  ```powershell
  $path = 'C:\Test'
  [int]$totalSize = 0
  $fileInfo = Get-ChildItem $path -Recurse
  foreach ($file in $fileInfo) {
      $totalSize += $file.Length
  }
  Write-Host "The total size of file in $path is $($totalSize /1MB) MB."
  ```

- **Do-While and While Loops** (Path verification):
  ```powershell
  do {
      $path = Read-Host 'Please enter a file path to evaluate'
      if (Test-Path $path) {
          $pathVerified = $true
      }
  } while ($pathVerified -eq $false)
  ```

### File and Process Handling
You've shown how to handle large processes and count files and folders within a specified path, both excellent for administrative scripts or system management tasks:

```powershell
$largeProcesses = Get-Process | Where-Object { $_.WorkingSet64 -gt 50MB }

$path = 'C:\Test'
[int]$fileCount = 0
[int]$folderCount = 0
[int]$totalSize = 0
$rawFileInfo = Get-ChildItem $path -Recurse
foreach ($item in $rawFileInfo) {
    if ($item.PSIsContainer) {
        $folderCount++
    } else {
        $fileCount++
        $totalSize += $item.Length
    }
}
Write-Host "Breakdown of $path"
Write-Host "Total Directories: $folderCount"
Write-Host "Total Files: $fileCount"
Write-Host "Total Size of files: $($totalSize / 1MB) MB"
```

These examples effectively showcase PowerShell's power in scripting and automation, emphasizing its object-oriented approach, versatility in handling files and processes, and its capability to interact with the user dynamically. 

---

# PowerShell Input Output
## code

```powershell

#INPUT

#-------------------------------------------------
#cmdlets - obviously

#retrieve dynamic content from a website
$webResults = Invoke-WebRequest -Uri 'https://reddit.com/r/powershell.json'
$rawJSON = $webResults.Content
$objData = $rawJSON | ConvertFrom-Json
$posts = $objData.data.children.data
$posts | Select-Object Title,Score | Sort-Object Score -Descending

#-------------------------------------------------
#Read Host
#https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/read-host?view=powershell-6

[int]$numPosts = Read-Host -Prompt "Enter the number of posts to read"
$posts | Select-Object Title,url | Sort-Object Score -Descending | Select-Object -First $numPosts

[int]$numFacts = Read-Host -Prompt "Enter the number of cat facts you would like"
$webResults = Invoke-RestMethod -Uri "https://catfact.ninja/facts?limit=$numFacts&max_length=140"
$webResults.data

#-------------------------------------------------
#Get Content
#https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-content?view=powershell-6

$logContent = Get-Content C:\Test\SampleLog.log
$logContent = Get-Content C:\Test\SampleLog.log -Tail 5
$b = Get-Content C:\Test\SampleLog.log -Raw

$logContent | Select-String -Pattern "INFO"
$regex = "\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b"
$logContent | Select-String -Pattern $regex -AllMatches
$logContent | Where-Object {$_ -like "*.*.*.*"}

$b | Select-String -Pattern $regex
$b | Where-Object {$_ -like "*.*.*.*"}

#__________________

$rawCSVInput = Get-Content C:\Test\tt.csv
$objData = $rawCSVInput | ConvertFrom-Csv

###############################################################################
###############################################################################

#OUTPUT
#https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/write-host?view=powershell-6
#https://www.jsnover.com/blog/2013/12/07/write-host-considered-harmful/

Write-Host 'Text output to console'
$hostInfo = Get-Host
Write-Host $hostInfo
Write-Host $hostInfo.Version
Write-Host "Warning" -ForegroundColor Yellow
Write-Host "ERROR" -ForegroundColor Red
Write-Host "Works Great" -ForegroundColor Green
Write-Host "CRITICAL ERROR" -BackgroundColor Red -ForegroundColor White

Write-Host "The display name is: $($bits.DisplayName)"

#-------------------------------------------------

Write-Output
#https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/write-output?view=powershell-6

<#
The Write-Output cmdlet sends the specified object down the pipeline to the next command. If the command is the last command in the pipeline, the object is displayed in the console.
Write-Output sends objects down the primary pipeline, also known as the "output stream" or the "success pipeline." To send error objects down the error pipeline, use Write-Error.
This cmdlet is typically used in scripts to display strings and other objects on the console. However, because the default behavior is to display the objects at the end of a pipeline, it is generally not necessary to use the cmdlet. For instance, Get-Process | Write-Output is equivalent to Get-Process.
#>
$processes = Get-Process
Write-Output $processes
$P

#-------------------------------------------------
#Out-File
#https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/out-file?view=powershell-6

$processes = Get-Process
$processes | Out-File C:\Test\processInfo.txt
$processes | ConvertTo-Csv -NoTypeInformation | Out-File c:\test\processes.csv
$processes | Export-Csv -Path c:\test\processes2.csv

$processes = Get-Process
$xml = ConvertTo-Xml -As "Document" -Depth 3 -InputObject $processes

#-------------------------------------------------
```

## Explanation

Your script demonstrates a broad range of PowerShell capabilities for handling input, processing it, and outputting results in various formats. Let’s discuss each part of your script:

### Input
1. **Invoke-WebRequest**:
   - You retrieve JSON content from a website, convert it to a PowerShell object, and then manipulate it. This is a great way to work with APIs and web data in PowerShell.
   ```powershell
   $webResults = Invoke-WebRequest -Uri 'https://reddit.com/r/powershell.json'
   $rawJSON = $webResults.Content
   $objData = $rawJSON | ConvertFrom-Json
   $posts = $objData.data.children.data
   $posts | Select-Object Title,Score | Sort-Object Score -Descending
   ```

2. **Read-Host**:
   - You interact with the user to get input, which you then use to filter and display data.
   ```powershell
   [int]$numPosts = Read-Host -Prompt "Enter the number of posts to read"
   $posts | Select-Object Title,url | Sort-Object Score -Descending | Select-Object -First $numPosts
   ```

3. **Get-Content**:
   - You demonstrate how to read file content, search for patterns with regex, and filter logs, which is essential for file handling and data parsing.
   ```powershell
   $logContent = Get-Content C:\Test\SampleLog.log
   $logContent | Select-String -Pattern "INFO"
   ```

### Output
1. **Write-Host**:
   - This cmdlet is used for displaying data in the console with formatting options. While useful for debugging and scripts that exclusively interact with the console, it is generally avoided in production scripts as it does not output to the PowerShell pipeline.
   ```powershell
   Write-Host "Works Great" -ForegroundColor Green
   ```

2. **Write-Output**:
   - This is the default way PowerShell outputs data. It’s often used explicitly in scripts to make them more readable or when the output needs to be passed explicitly to the next command in the pipeline.
   ```powershell
   $processes = Get-Process
   Write-Output $processes
   ```

3. **Out-File and Export-Csv**:
   - These cmdlets are used to write output to files. `Out-File` writes data to a text file, while `Export-Csv` outputs data in CSV format, which is useful for exporting data that needs to be consumed by other programs or services.
   ```powershell
   $processes | Out-File C:\Test\processInfo.txt
   $processes | Export-Csv -Path c:\test\processes2.csv
   ```

### Summary
Your script effectively showcases how to integrate web APIs, handle user input, read and write files, and display outputs in PowerShell. Each of these operations is fundamental in PowerShell scripting, especially for tasks involving automation and data manipulation.

This script could be used as a template or a learning tool for those who want to understand how to perform common tasks in PowerShell effectively. 

---


<#
we want to be notified when a drive in our home network is running low on space.
The script will check the desired drive for free space, 
and send a Telegram notification if it is below 20%.
All actions should be logged.
The script should support scannig a drive the user specifies.
The script should support both Linux and Windows.
#>
<#
Supports loggigng 
Checks desired drive for free space
Lets user specify drive 
Supports both windows and linux
send Telegram notification
#>


The script you've provided is quite comprehensive and well-structured, serving the purpose of monitoring drive space and sending notifications via Telegram if the space falls below a certain threshold. Here's a breakdown and analysis to help you understand its components and functionalities:


### Script Breakdown

First, ensure you have the `PoshGram` module installed. You can install it from the PowerShell Gallery with the following command:

```powershell
Install-Module -Name PoshGram -Scope CurrentUser
```
#### **1. Execution Policies and Script Handling**
This part of your script handles the execution policies in PowerShell, which determine how PowerShell loads configuration files and runs scripts. This ensures the script can execute without security restrictions that might block it.

```powershell
# List current execution policies
Get-ExecutionPolicy -List

# Set execution policy to allow local scripts and downloaded scripts that are signed
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Unblock a script downloaded from the internet
Unblock-File -Path .\drive_warn.ps1

# Run a local script
.\drive_warn.ps1

# Run a local script in the current scope
. .\drive_warn.ps1
```

#### **2. Script Example with Parameters**
The main part of your script includes parameters and logic for determining drive space, logging actions, and sending notifications if necessary.

- **Parameters**: The script starts by defining a mandatory parameter `$Drive`, which specifies which drive to monitor.
  
- **Platform-Specific Paths**: It sets log paths depending on the operating system, accommodating both Windows and Linux platforms.

- **Logging Setup**: It checks if the log directory and file exist; if not, it creates them. It logs every significant step to help with debugging and tracking the script's operations.

- **Drive Space Checking**: The script checks the specified drive's free space percentage. It handles both Unix-like and Windows systems differently due to differences in cmdlets and object properties available on these platforms.

- **Telegram Notification**: If the free space percentage is below the set threshold, it imports the `PoshGram` module, constructs a message, and sends it using the Telegram bot.

```powershell
param (
    [Parameter(Mandatory = $true)]
    [string] $Drive
)

# Different log path based on OS
if ($PSVersionTable.Platform -eq 'Unix') {
    $logPath = '/tmp'
} else {
    $logPath = 'C:\Logs'
}

$logFile = "$logPath\driveCheck.log"

# Ensure logging directory and file exist
try {
    if (-not (Test-Path -Path $logPath -ErrorAction Stop)) {
        New-Item -ItemType Directory -Path $logPath -ErrorAction Stop | Out-Null
        New-Item -ItemType File -Path $logFile -ErrorAction Stop | Out-Null
    }
} catch {
    throw
}

Add-Content -Path $logFile -Value "[INFO] Running $PSCommandPath"

# Checking drive space and logging
try {
    if ($PSVersionTable.Platform -eq 'Unix') {
        # Unix-like systems
        $volume = Get-PSDrive -Name $Drive -ErrorAction Stop
    } else {
        # Windows systems
        $volume = Get-Volume -ErrorAction Stop | Where-Object { $_.DriveLetter -eq $Drive }
    }

    if ($volume) {
        $total = $volume.Free + $volume.Used
        $percentFree = [int](($volume.Free / $total) * 100)
        Add-Content -Path $logFile -Value "[INFO] Percent Free: $percentFree%"
    } else {
        throw "Drive $Drive was not found."
    }
} catch {
    Add-Content -Path $logFile -Value "[ERROR] Unable to retrieve volume information: $_"
    throw
}

# Notification if below threshold
if ($percentFree -le 20) {
    try {
        Import-Module PoshGram -ErrorAction Stop
        Add-Content -Path $logFile -Value '[INFO] PoshGram imported successfully.'
        $messageSplat = @{
            BotToken    = "YOUR_BOT_TOKEN"
            ChatID      = "YOUR_CHAT_ID"
            Message     = "[LOW SPACE] Drive at: $percentFree%"
            ErrorAction = 'Stop'
        }
        Send-TelegramTextMessage @messageSplat
        Add-Content -Path $logFile -Value '[INFO] Message sent successfully'
    } catch {
        Add-Content -Path $logFile -Value "[ERROR] Error encountered sending message: $_"
        throw
    }
}
```

### Enhancements and Security
- **Secure Handling of Tokens**: Ensure that the Telegram bot token is securely handled, possibly using secure strings or encrypted storage.
- **Parameter Validation**: Adding validation logic for the `$Drive` parameter can help avoid errors due to invalid inputs.
- **Modular Design**: Consider wrapping different functionalities into functions for better readability and reusability.
- **Logging Improvements**: Implement more granular logging levels (INFO, WARN, ERROR) to categorize log entries effectively.

This
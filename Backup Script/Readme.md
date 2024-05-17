#Create a backup script
#A common task is to create a backup. A backup is usually a compressed file that stores all the files belonging to, for example, an app. When you installed PowerShell, you got the cmdlet Compress-Archive, which can help you complete this task.
#Step 1 create directory and files using linux
<#
mkdir app
cd app
touch index.html app.js
cd ..
#>

#Step 2 - start powershell type pwsh in terminal and hit enter
<#pwsh#>

#Step 3 - Create a script file named Backup.ps1 in the current directory and open it in your code editor. 
<#touch Backup.ps1
code Backup.ps1#>

#Step 4 - Add this content to the file and save the file. You can use CTRL-S on Windows and Linux 
$date = Get-Date -format "yyyy-MM-dd"
Compress-Archive -Path './app' -CompressionLevel 'Fastest' -DestinationPath "./backup-$date"
Write-Host "Created backup at $('./backup-' + $date + '.zip')"

<#The script invokes Compress-Archive and uses three parameters:

-Path is the directory of the files that you want to compress.
-CompressionLevel specifies how much to compress the files.
-DestinationPath is the path to the resulting compressed file.#>

#Step 5 - Run the script: 
.Backup.ps1

# You should see this output: Created backup at ./backup-<current date as YYYY-MM-DD>.zip

<#Add parameters to your script
If you add parameters to your script, users can provide values when it runs. You'll add parameters to your backup script to enable configuration of the locations of the source files and the resulting zip file.
#>

#Step 6 - Add the following code to the top of the Backup.ps1 file.
Param(
  [string]$Path = './app',
  [string]$DestinationPath = './'
)
$date = Get-Date -format "yyyy-MM-dd"
Compress-Archive -Path './app' -CompressionLevel 'Fastest' -DestinationPath "./backup-$date"
Write-Host "Created backup at $('./backup-' + $date + '.zip')"

<#You've added two parameters to your script: $Path and $DestinationPath. You've also provided default values so users don't need to provide the values. Users can override the default values if they need to. You need to adjust the script to use these parameters. You'll do so next.
#>

#Step 7 - Change the code in the file to use the parameters, then save the file. Backup.ps1 should now look like this:
Param(
  [string]$Path = './app',
  [string]$DestinationPath = './'
)
$date = Get-Date -format "yyyy-MM-dd"
Compress-Archive -Path $Path -CompressionLevel 'Fastest' -DestinationPath "$($DestinationPath + 'backup-' + $date)"
Write-Host "Created backup at $($DestinationPath + 'backup-' + $date + '.zip')"

#Step 8 - Rename your app directory to webapp by running this command: mv app webapp

<#Renaming the app directory simulates the fact that not all directories you'll need to back up will be called app.

You can no longer rely on the default value for $Path. You'll need to provide a value via the console when you run the script.
#>

#Step 9 - Remove your backup file, replacing <current date as YYYY-MM-DD> with the current date run cmd: rm backup-<current date as YYYY-MM-DD>.zip
<#You're removing this file to make sure you get a message stating that your $Path value doesn't exist. Otherwise, you'd get a message about the zip file already existing, and the problem we're trying to fix would be hidden.
#>

# if you run the script you will see the following error:
<#Line |
   8 |  Compress-Archive -Path $Path -CompressionLevel 'Fastest' -Destination …
     |  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     | The path './app' either does not exist or is not a valid file system path.
Created backup at ./backup-<current date as YYYY-MM-DD>.zip
#>

#The script notifies you that it can't find the directory ./app. Now it's time to provide a value to the $Path parameter and see the benefit of adding parameters to your script.

#Step 10 - Test your script by running it:
./Backup.ps1 -Path './webapp'

#You'll see a message similar to the one you got earlier:

<#Created backup at ./backup-<current date as YYYY-MM-DD>.zip
#>

<#You can now use parameters if the directory you want to back up isn't called ./app or if you want to put the compressed file somewhere other than the current directory.#>

#Next Steps - Adding Flow control

<#When you write scripts, they might work as intended as long as you type in reasonable values. But if time passes or someone else runs the script, it's likely that someone will enter an unintended value or that some other precondition won't be met. To avoid situations like this, you should sanitize your input; that is, you should add logic to your script to ensure it quits early if something is wrong and continues to run only if everything is fine.
#>

<#Add checks to your script parameters
You've been working with a backup script so far, and you've been adding parameters to it. You can make your script even safer to use by adding checks that ensure the script only continues if it's provided reasonable parameter inputs.

Let's look at the current script.
#>

#Step 1- Add a check for the $Path parameter by adding this code right after the Param section, then save the file:
Param(
  [string]$Path = './app',
  [string]$DestinationPath = './'
)
If (-Not (Test-Path $Path)) 
{
  Throw "The source directory $Path does not exist, please specify an existing directory"
}
$date = Get-Date -format "yyyy-MM-dd"
Compress-Archive -Path $Path -CompressionLevel 'Fastest' -DestinationPath "$($DestinationPath + 'backup-' + $date)"
Write-Host "Created backup at $($DestinationPath + 'backup-' + $date + '.zip')"

#You've added a test that checks if $Path exists. If it doesn't, you stop the script. You also explain to users what went wrong so they can fix the problem.

#Step  2 - Ensure the script works as intended by running it:
./Backup.ps1 -Path './app'

#You should see this output:
<#Throw "The source directory $Path does not exist, please specify  …
  |      ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  | The source directory ./app does not exist, please specify an
  | existing directory#>

  #Step 3 - Test that the script still works as intended. (Be sure to remove any backup files from the previous exercise before you continue.)
  ./Backup.ps1 -Path './webapp'
  
  #You should see a message that looks similar to this one: Created backup at ./backup-2024-05-17.zip
  <#If you run the script again, it will stop responding. It will notify you that the zip file already exists. Let's fix that problem. We'll add code to ensure the backup is created only if no other backup zip file from the current day exists.
  #>

  #Step 4- Modify the code
  Param(
  [string]$Path = './app',
  [string]$DestinationPath = './'
)
If (-Not (Test-Path $Path)) 
{
  Throw "The source directory $Path does not exist, please specify an existing directory"
}
$date = Get-Date -format "yyyy-MM-dd"
$DestinationFile = "$($DestinationPath + 'backup-' + $date + '.zip')"
If (-Not (Test-Path $DestinationFile)) 
{
  Compress-Archive -Path $Path -CompressionLevel 'Fastest' -DestinationPath "$($DestinationPath + 'backup-' + $date)"
  Write-Host "Created backup at $($DestinationPath + 'backup-' + $date + '.zip')"
} Else {
  Write-Error "Today's backup already exists"
}

<#You did two things here. First, you created a new variable, $DestinationFile. This variable makes it easy to check if the path already exists. Secondly, you added logic that says "create the zip file only if the file doesn't already exist." This code implements that logic:
If (-Not (Test-Path $DestinationFile)) 
{
  Compress-Archive -Path $Path -CompressionLevel 'Fastest' -DestinationPath "$($DestinationPath + 'backup-' + $date)"
  Write-Host "Created backup at $($DestinationPath + 'backup-' + $date + '.zip')"
} Else {
  Write-Error "Today's backup already exists"
}
#>

#Step 5 - Run the code to make sure the script doesn't stop responding and that your logic is applied:
./Backup.ps1 -Path './webapp'

<#You should see this output:
Write-Error: Today's backup already exists
#>


  
#Next Section adding error handling
<#you'll use a Try/Catch block to ensure the script stops responding early if a certain condition isn't met Say you've noticed that you sometimes specify an erroneous path, which causes backup of files that shouldn't be backed up. You decide to add some error management.#>
#Step 1 -  In the Param section, add a comma after the last parameter, and then add the following parameter: [switch]$PathIsWebApp
Param(
  [string]$Path = './app',
  [string]$DestinationPath = './',
  [switch]$PathIsWebApp
)
If (-Not (Test-Path $Path)) 
{
  Throw "The source directory $Path does not exist, please specify an existing directory"
}
$date = Get-Date -format "yyyy-MM-dd"
$DestinationFile = "$($DestinationPath + 'backup-' + $date + '.zip')"
If (-Not (Test-Path $DestinationFile)) 
{
  Compress-Archive -Path $Path -CompressionLevel 'Fastest' -DestinationPath "$($DestinationPath + 'backup-' + $date)"
  Write-Host "Created backup at $($DestinationPath + 'backup-' + $date + '.zip')"
} Else {
  Write-Error "Today's backup already exists"
}

<#You've added a switch parameter. If this parameter is present when the script is invoked, you perform the check on the content. After that, you can determine if a backup file should be created.#>

<#Under the Param section, add this code, then save the file: 
If ($PathIsWebApp -eq $True) {
   Try 
   {
     $ContainsApplicationFiles = "$((Get-ChildItem $Path).Extension | Sort-Object -Unique)" -match  '\.js|\.html|\.css'

If ( -Not $ContainsApplicationFiles) {
       Throw "Not a web app"
     } Else {
       Write-Host "Source files look good, continuing"
     }
   } Catch {
    Throw "No backup created due to: $($_.Exception.Message)"
   }
}

#>

Param(
  [string]$Path = './app',
  [string]$DestinationPath = './',
  [switch]$PathIsWebApp
)
If ($PathIsWebApp -eq $True) {
   Try 
   {
     $ContainsApplicationFiles = "$((Get-ChildItem $Path).Extension | Sort-Object -Unique)" -match  '\.js|\.html|\.css'

If ( -Not $ContainsApplicationFiles) {
       Throw "Not a web app"
     } Else {
       Write-Host "Source files look good, continuing"
     }
   } Catch {
    Throw "No backup created due to: $($_.Exception.Message)"
   }
}
If (-Not (Test-Path $Path)) 
{
  Throw "The source directory $Path does not exist, please specify an existing directory"
}
$date = Get-Date -format "yyyy-MM-dd"
$DestinationFile = "$($DestinationPath + 'backup-' + $date + '.zip')"
If (-Not (Test-Path $DestinationFile)) 
{
  Compress-Archive -Path $Path -CompressionLevel 'Fastest' -DestinationPath "$($DestinationPath + 'backup-' + $date)"
  Write-Host "Created backup at $($DestinationPath + 'backup-' + $date + '.zip')"
} Else {
  Write-Error "Today's backup already exists"
}

<#
The preceding code first checks if the parameter $PathIsWebApp is provided at runtime. 
If it is, the code continues to get a list of file extensions from the directory specified by $Path. 
In our case, if you run that part of the code on the webapp directory, the following code will print a list of items: (Get-ChildItem $Path).Extension | Sort-Object -Unique
#>

<#
In the full statement, we're using the -match operator. The -match operator expects a regular expression pattern. In this case, the expression states "do any of the file extensions match .html, .js, or .css?" The result of the statement is saved to the variable $ContainsApplicationFiles.

Then the If block checks whether the $ContainsApplicationFiles variable is True or False. At this point, the code can take two paths:

If the source directory is for a web app, the script writes out "Source files look good, continuing."
If the source directory isn't for a web app, the script throws an error that states "Not a web app." The error is caught in a Catch block. The script stops, and you rethrow the error with an improved error message.
#>

#Step 3 - Test the script by providing the switch $PathIsWebApp:
./Backup.ps1 -PathIsWebApp -Path './webapp'

#Test the script
<#Step 1- Using your terminal, create a directory named python-app. In the new directory, create a file called script.py:
mkdir python-app
cd python-app
touch script.py
cd ..
#>

<#Your directory should now look like this:
-| webapp/
---| app.js
---| index.html
-| python-app/
---| script.py
-| Backup.ps1
#>

#In the PowerShell shell, run the script again, but this time change the -Path value to point to ./python-app:
./Backup.ps1 -PathIsWebApp -Path './python-app'

<# Your script should now print this text: 
No backup created due to: Not a web app
The output indicates that the check failed. It should have, because there are no files in the directory that have an .html, .js, or .css extension. Your code raised an exception that was caught by your Catch block, and the script stopped early.
#>

#Completed Script to run
Param(
  [string]$Path = './app',
  [string]$DestinationPath = './',
  [switch]$PathIsWebApp
)
If ($PathIsWebApp -eq $True) {
   Try 
   {
     $ContainsApplicationFiles = "$((Get-ChildItem $Path).Extension | Sort-Object -Unique)" -match  '\.js|\.html|\.css'

If ( -Not $ContainsApplicationFiles) {
       Throw "Not a web app"
     } Else {
       Write-Host "Source files look good, continuing"
     }
   } Catch {
    Throw "No backup created due to: $($_.Exception.Message)"
   }
}
If (-Not (Test-Path $Path)) 
{
  Throw "The source directory $Path does not exist, please specify an existing directory"
}
$date = Get-Date -format "yyyy-MM-dd"
$DestinationFile = "$($DestinationPath + 'backup-' + $date + '.zip')"
If (-Not (Test-Path $DestinationFile)) 
{
  Compress-Archive -Path $Path -CompressionLevel 'Fastest' -DestinationPath "$($DestinationPath + 'backup-' + $date)"
  Write-Host "Created backup at $($DestinationPath + 'backup-' + $date + '.zip')"
} Else {
  Write-Error "Today's backup already exists"
}
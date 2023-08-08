

  <h1 align="center">Transcribe Video</h1>
<p align="center">
  Step-by-Step guide for video tutorial with open sourced tools and scripts . 
</p>


## Extract audio from video using FFmpeg
### 1. Install [Chocolatey](https://chocolatey.org/install#individual) for easy installation of FFmpeg
- Open PowerShell with administrator rights. Navigate to the start menu, search for "PowerShell", right-click, and choose "Run as administrator".
- Run command `Set-ExecutionPolicy Bypass -Scope Process -Force`
(This command temporarily bypasses certain security restrictions allowing us to install Chocolatey.)

- Run command to install chocolatey
```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```
- Run command `choco -v` to confirm success

### 2. Install [FFmpeg](https://github.com/FFmpeg/FFmpeg)
- Run command `choco install ffmpeg`
- Run command `ffmpeg -version` to confirm success

### 3. Run script to extract audio
option A (single thread):
- Default powershell command, adjust to your video folder using $rootPath = "Z:\zensive\zensive"
```
# Define the root path
$rootPath = "g:\zensive"

# Create root mp3 directory
$rootMp3Folder = Join-Path -Path $rootPath -ChildPath "mp3"
if(!(Test-Path -Path $rootMp3Folder)) {
    New-Item -ItemType directory -Path $rootMp3Folder
}

# Define all popular video formats
$videoFormats = @("*.mp4", "*.avi", "*.mkv", "*.mov", "*.wmv")

# Get all video files in all directories
$videoFiles = $videoFormats | ForEach-Object { Get-ChildItem -LiteralPath $rootPath -Recurse -Filter $_ }

# Process each file
$videoFiles | ForEach-Object {
    # Create the sub directory path
    $subDirectoryPath = $_.Directory.FullName.Replace($rootPath, $rootMp3Folder)
    # Check and create sub directory if not exists
    if(!(Test-Path -Path $subDirectoryPath)) {
        New-Item -ItemType directory -Path $subDirectoryPath
    }

    # Convert video to mp3
    $mp3Path = Join-Path -Path $subDirectoryPath -ChildPath ($_.BaseName + '.mp3')
    ffmpeg -i $_.FullName -vn -acodec libmp3lame $mp3Path
}
```

option B (multi-thread):
- install PowerShell 7, download [PowerShell-7.3.6-win-x64.msi](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.3#msi) 
- adjust to your video folder using $rootPath = "Z:\zensive\zensive"
- adjust '$threadCount = 16' to match your CPU's thread count.
```
# Define the root path
$rootPath = "g:\zensive"

# Set thread count
$threadCount = 16

# Create root mp3 directory
$rootMp3Folder = Join-Path -Path $rootPath -ChildPath "mp3"
if(!(Test-Path -Path $rootMp3Folder)) {
    New-Item -ItemType directory -Path $rootMp3Folder
}

# Define all popular video formats
$videoFormats = @("*.mp4", "*.avi", "*.mkv", "*.mov", "*.wmv")

# Get all video files in all directories
$videoFiles = $videoFormats | ForEach-Object { Get-ChildItem -LiteralPath $rootPath -Recurse -Filter $_ }

# Process each file in parallel
$videoFiles | ForEach-Object -ThrottleLimit $threadCount -Parallel {
    # Create the sub directory path
    $subDirectoryPath = $_.Directory.FullName.Replace($using:rootPath, $using:rootMp3Folder)
    # Check and create sub directory if not exists
    if(!(Test-Path -Path $subDirectoryPath)) {
        New-Item -ItemType directory -Path $subDirectoryPath
    }

    # Convert video to mp3
    $mp3Path = Join-Path -Path $subDirectoryPath -ChildPath ($_.BaseName + '.mp3')
    ffmpeg -i $_.FullName -vn -acodec libmp3lame $mp3Path
}
```
     


## Transcribe audio to text using Whipser AI library
### Install [WhisperPS](https://github.com/Const-me/Whisper/releases) 
- Extract and copy the extracted WhisperPS folder  to 'C:\Users\XXXX\Documents\WindowsPowerShell\Modules', where 'XXXX' is your windows username.
### Download a Whisper AI model  https://huggingface.co/ggerganov/whisper.cpp/tree/main 
-  I recommend using 'ggml-medium.bin' for its optimal balance of transcription quality and processing speed.
### Run powershell script after adjusting videoFolderPath = "G:\summary\mp3"
```
# Define the path to the folder containing the video files
$videoFolderPath = "g:\zensive\mp3"

# Import the Whisper module
Import-Module WhisperPS -DisableNameChecking
$ModelPath = "G:\01.Video Editing\00.AI transcribe\models\ggml-medium.bin"
$Model = Import-WhisperModel $ModelPath

# Transcribe all *.mp3 and *.m4a files in the defined folder and its subfolders
Get-ChildItem -Path $videoFolderPath -Recurse -include *.mp3, *.m4a | Transcribe-File $Model | foreach { $_ | Export-Text (Join-Path -Path $videoFolderPath -ChildPath ($_.SourceName + ".txt")) }
```
## Feed the text files to AI tool of your choice
### chatGPT
- You can prompt chatGPT to take multiple text files. Use original prompt like this 
```
Please provide an in-depth summary of the key ideas and details in the provided passage. I will give the passages in multiple prompts.  Include the following in your summary:

An introduction briefly describing the overall topic and purpose of the passage
A detailed breakdown of each main idea or section, summarizing the key points and supporting details
Examples, quotes, or specific data used in the passage to illustrate the concepts
An explanation of how ideas connect and build off each other
A conclusion restating the main purpose and highlighting the most important takeaways
Be comprehensive but concise in summarizing all relevant points and information
Use your own words as much as possible while accurately representing the original content
Please write your response in multiple paragraphs, providing as much relevant detail and expansion on the passage as you can .  If you understand, reply yes so we can continue.
```
- Then you can paste in multiple prompt, just do not exceed the token limit.

 ### Claude
 - You can just drag and drop multiple text files to claude. Claude has a much longer token length.
   
## Author

- Ed ([@ed_zensive](https://twitter.com/zensive_ed))

## License

Licensed under the [MIT license](https://github.com/steven-tey/novel/blob/main/LICENSE.md). 

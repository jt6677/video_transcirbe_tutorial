

  <h1 align="center">Video Tutorial Transcribe</h1>
<p align="center">
  Step-by-Step guide for video tutorial with opne sourced tools and scripts . 
</p>


## Extract audio from video using FFmpeg
### 1. Install [Chocolatey](https://chocolatey.org/install#individual) to easy installation of FFmpeg
- Open PowerShell with administrator rights. Navigate to the start menu, search for "PowerShell", right-click, and choose "Run as administrator".
- Run command `Set-ExecutionPolicy Bypass -Scope Process -Force`
(This command temporarily bypasses certain security restrictions allowing us to install Chocolatey.)

- Run command to install chocolatey
```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object
System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
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
$rootPath = "Z:\zensive\zensive"

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

option b(multi-thread):
- install PowerShell 7, download [PowerShell-7.3.6-win-x64.msi](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.3#msi) 
- adjust to your video folder using $rootPath = "Z:\zensive\zensive"
- adjust '$threadCount = 16' to match your CPU's thread count.
```
# Define the root path
$rootPath = "Z:\zensive\zensive"

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
     


## Extract audio from video using FFmpeg
### Install [WhisperPS](https://github.com/Const-me/Whisper/releases) 
- Extract and copy the extracted WhisperPS folder  to 'C:\Users\XXXX\Documents\WindowsPowerShell\Modules', where 'XXXX' is your windows username.
### Download a Whisper AI model  https://huggingface.co/ggerganov/whisper.cpp/tree/main 
-  I recommend using 'ggml-medium.bin' for its optimal balance of transcription quality and processing speed.
### Run powershell script after adjusting videoFolderPath = "G:\summary\mp3"
```
# Define the path to the folder containing the video files
$videoFolderPath = "G:\summary\mp3"

# Import the Whisper module
Import-Module WhisperPS -DisableNameChecking
$ModelPath = "G:\01.Video Editing\00.AI transcribe\models\ggml-medium.bin"
$Model = Import-WhisperModel $ModelPath

# Transcribe all *.mp3 and *.m4a files in the defined folder and its subfolders
Get-ChildItem -Path $videoFolderPath -Recurse -include *.mp3, *.m4a | Transcribe-File $Model | foreach { $_ | Export-Text (Join-Path -Path $videoFolderPath -ChildPath ($_.SourceName + ".txt")) }
```


## Author

- Ed ([@ed_zensive](https://twitter.com/zensive_ed))

## License

Licensed under the [MIT license](https://github.com/steven-tey/novel/blob/main/LICENSE.md).

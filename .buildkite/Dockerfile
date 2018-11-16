# Heavily based on https://github.com/StefanScherer/dockerfiles-windows/ images.
# Combines the node nano image with the Bazel Prerequisites (https://docs.bazel.build/versions/master/install-windows.html).
# msys install taken from https://github.com/StefanScherer/dockerfiles-windows/issues/30
# VS redist install taken from https://github.com/StefanScherer/dockerfiles-windows/blob/master/apache/Dockerfile

# Use a full featured Windows Server to setup files needed for the real image.
FROM microsoft/windowsservercore:1803 as download

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Install 7zip to extract msys2
RUN Invoke-WebRequest -UseBasicParsing "https://www.7-zip.org/a/7z1805-x64.exe" -OutFile 7z.exe
# For some reason the last letter in the destination directory is lost. So "/D=C:\\7zip0" will make files be
# extracted to "/D=C:\\7zip".
RUN Start-Process "c:\\7z.exe" -ArgumentList "/S", "/D=C:\\7zip0" -NoNewWindow -Wait

# Extract msys2
RUN Invoke-WebRequest -UseBasicParsing "http://repo.msys2.org/distrib/x86_64/msys2-base-x86_64-20180531.tar.xz" -OutFile msys2.tar.xz
RUN C:\7zip\7z e msys2.tar.xz -Wait
RUN C:\7zip\7z x msys2.tar -o"C:\\"

# Install Microsoft Visual C++ Redistributable for Visual Studio 2015
RUN Invoke-WebRequest -UseBasicParsing "https://download.microsoft.com/download/9/3/F/93FCF1E7-E6A4-478B-96E7-D4B285925B00/vc_redist.x64.exe" -OutFile vc_redist.x64.exe
RUN Start-Process "c:\\vc_redist.x64.exe" -ArgumentList "/Install", "/Passive", "/NoRestart" -NoNewWindow -Wait

# Needs minimum Windos Server 1803 to workaround https://github.com/nodejs/node/issues/8897#issuecomment-429601955
FROM stefanscherer/node-windows:10.13.0-nanoserver-1803
# FROM stefanscherer/node-windows:10.13.0-windowsservercore

SHELL ["cmd.exe", "/S", "/C"]

# Copy over msys and vc_redist files
COPY --from=download "C:\\msys64" "C:\\msys64"
# Copy over vc_redist files.
# Are those the only ones needed? Full list in https://docs.microsoft.com/en-us/cpp/ide/determining-which-dlls-to-redistribute?view=vs-2017
COPY --from=download "C:\\windows\\system32\\msvcp140.dll" "C:\\windows\\system32"
COPY --from=download "C:\\windows\\system32\\vcruntime140.dll" "C:\\windows\\system32"

# It looks like these paths cannot have escaped slashes, otherwise \\ will be passed on.
RUN setx /M BAZEL_SH "C:\msys64\usr\bin\bash.exe"
RUN setx /M PATH "%PATH%;c:\msys64\usr\bin"

# nanoserver seems to be missing FOLDERID_Profile: https://github.com/dotnet/core-setup/issues/1507#issuecomment-286949120
# But bazel needs it: https://github.com/bazelbuild/bazel/blob/ba6fbded11b3b40e2680c6fca27db9563c7c2193/src/main/cpp/blaze_util_windows.cc#L412
# Bazel will fallback to HOME so we can set that instead.
RUN setx /M HOME %USERPROFILE%

CMD ["cmd.exe"]

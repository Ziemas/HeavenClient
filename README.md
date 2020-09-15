# HeavenClient

HeavenClient is a custom, made-from-scratch game client for Maplestory v83.

## Supported versions

- The client is currently compatible with version 83 servers.
- The client has only been tested with [HeavenMS].
- **[Switch]**
- **[Linux]**

## Configuration

The build can be configured by editing the **MapleStory.h** file. The following options are available:

- **USE_ASIO**: Use Asio for networking (additional dependency)
- **USE_CRYPTO**: Use cryptography when communicating for the server.
- **USE_NX**: Use NX files instead of WZ files.
- **USE_DEBUG**: This suppresses the generation of the Settings file

The default settings can be configured by editing the **Configuration.h** file. These are also generated after a game session in a file called **Settings**. These can be altered in the same way as **Configuration.h**, although, these do not persist if you delete the file, unlike **Configuration.h**.

## Building
### Windows
1. Open **MapleStory.sln** in Visual Studio 2017 CE
2. Make sure to use **Windows SDK Version: 8.1** and **Platform Toolset: v140** (If you don't have these, download them)
   * [Windows 8.1 SDK]
3. Press **Build** > **Build Solution** or **Ctrl + Shift + B**
4. After a successful build, you can now run the program by pressing **Debug** > **Start Debugging** or **F5**
5. Make sure all nx files are present in the parent folder. To convert wz files to nx you can use the [NoLifeWzToNx] project.
   - Extract the zip
   - Place your wz files in the **files** folder
   - Run **start.bat**
   - See **Required Files** for a list of required nx files

### Linux
1. Run ```./build-deps.sh```. We try to build each dependency from source -- if any dependencies fail to build, you could try and find the corresponding package for your linux distro if it exists.
2. ```mkdir build```
3. ```cd build```
4. ```cmake ..```
5. ```make -j$CORES``` where $CORES is your number of CPU cores

### Mac

1. You must have GCC from brew! ```brew install gcc```
2. Run ```./build-deps.mac.sh```. We try to build each dependency from source -- if any dependencies fail to build, you could try and find the corresponding package for your linux distro if it exists.
3. ```mkdir build```
4. ```cd build```
5. Tell cmake to use compiler from brew by ``` cmake .. -DCMAKE_C_COMPILER=/usr/local/bin/gcc```
6. ```make -j$CORES``` where $CORES is your number of CPU cores

This will always be the option with the most performance, but if you are using Mac / are having issues, try out the [Vagrant](#vagrant-setup) or [Docker](#docker-setup---web-vnc) setups. 

## Required Files

*Always check **NxFiles.h** for an updated list of required nx files*

- [MapPretty.nx](https://drive.google.com/file/d/1d8HJkWY6ght5OAoJGtsAjNiG2BL1wcle/view?usp=sharing) (v167 GMS Map.wz)
- MapLatest.nx (Lastest GMS Map.wz)
- Map001.nx (Latest GMS Map001.wz)
- UI.nx (Latest GMS UI.wz)
- Everything else is from v83 GMS wz files

There is an archive of all the NX files listed above available for download [here][1] (Latest: v213.2).

# Dependencies for Linux/Mac
- Nx library:
[NoLifeNx]

- Wz library:
N/A

- Graphics:
[GLFW3](http://www.glfw.org/download.html), [GLAD](https://github.com/Dav1dde/glad), [FreeType](http://www.freetype.org/)

- Audio:
[OpenAL-soft](https://github.com/kcat/openal-soft), [Alure](https://github.com/kcat/alure)

- Networking:
[Asio](http://think-async.com/)

---

## In-Game Issues

If you experience any kind of in-game glitches, UI rendering issues, or anything else that seems out of the ordinary that other developers are not experiences; Follow these steps in order to hopefully resolve aforementioned issues.

1. Clean Solution
2. Close Visual Studio
3. Delete the following files/folders: **.vs**, **x64**, **debug.log**, **MapleStory.aps**, **Settings**
4. Open Solution
5. Rebuild Solution

Note: These steps are not applicable to linux

---

## Vagrant setup

A [Vagrantfile](./Vagrantfile) has been included in the repo to simulate a complete HeaventClient setup on a linux ubuntu/bionic virtual machine (using virtualbox provider).

One can refer to the shell scripts written within the Vagrantfile to get an insight of the complete environmental setup and dependencies required to build and run the HeavenClient binary.

### Building/Running client via Vagrant

All ssh commands must be run on a shell within context of the HeavenClient directory (this project)

1. ```vagrant up```
2. Login to the desktop environment via the virtualbox window with default vagrant credentials (vagrant:vagrant) - once the bootstrap script has completed execution
   1. This is required to start up desktop environment (xfce)
3. ```vagrant ssh -- -R 8484:localhost:8484 -R 7575:localhost:7575 -R 7576:localhost:7576 -R 7577:localhost:7577```
   1. We SSH into the VM with reverse port-forwarding; this allows us to run/dev our [server](https://github.com/ronancpl/HeavenMS) on the host machine while allowing the client to connect to it and run on the VM
4. ```cd /home/vagrant/Desktop/HeavenClient/build/```
5. ```./HeavenClient```

Note: To run the HeavenClient from the build directory; all the [relevant **.nx** files](./Util/NxFiles.h) must be available within the linux build directory

Tip: Since the binary is built on the VM with a mounted/shared volume; the binary is also available/usable by a linux host

## Docker Setup - web-vnc

A [Dockerfile](./Dockerfile) has been included in the repo to simulate a complete HeaventClient setup on a linux ubuntu/bionic OS.

The Docker setup utilizes [fcwu/docker-ubuntu-vnc-desktop](https://github.com/fcwu/docker-ubuntu-vnc-desktop) **web-based lxde VNC** solution to output HeavenClient GUI on a web browser at port **6080**.

This allows the host to rely on minimal dependencies (other than nx files) for running HeavenClient.

### Docker Pre-requisites

- All [relevant](./Util/NxFiles.h) **.nx** files must be located in the **nx** directory
- A [Settings](./Settings) file must be present in project root - instructions to create this [below](#building--runnning-client)
  - This gives user ability to configure HeavenClient outside the container - on the host
- Relevant audio passthrough device must be installed on the host
  - Mac: [pulseaudio](https://www.freedesktop.org/wiki/Software/PulseAudio/)
  - Linux: [snd-aloop](./https://www.alsa-project.org/wiki/Matrix:Module-aloop)
- [HeavenMS](https://github.com/ronancpl/HeavenMS) server running at port 8484 on host machine

### Installing sound passthrough device

#### Mac

```sh
# Install pulseaudio
brew install pulseaudio

# Run pulseaudio daemon
pulseaudio --load=module-native-protocol-tcp --exit-idle-time=-1 --daemon
```

Note: To stop sound passthrough daemon: ```pulseaudio --kill```

#### Linux

```sh
# Insert kernel module snd-aloop and specify 2 as the index of sound loop device
sudo modprobe snd-aloop index=2
```

### Building & Runnning Client

1. Create Settings with server ip pointing to host
   1. ```echo "ServerIP = host.docker.internal" > Settings```
      1. This allows the HeavenClient application within the container to point to the running hosts IP (on which the server will be running)
2. Build & Run container for your OS
   1. Mac:
      1. ```docker-compose -f docker-compose.yml -f docker-compose.mac.yml up --build```
   2. Linux:
      1. ```docker-compose -f docker-compose.yml -f docker-compose.linux.yml up --build```
3. Run HeavenClient in the browser GUI at [localhost:6080](http://localhost:6080) by double clicking the HeavenClient icon

</br>
Note: If the client fails to startup, then it is very likely something went wrong with the setup. The link doesn't show errors hence you will have to use the **LXTerminal** within the web-based GUI to run the client manually.

- The HeaventClient binary is located within the **/root** folder of the container
- You may need to restart your browser before trying to run the game after applying any fixes
- You need to ```docker-compose down``` if you move your **.nx** files into the **nx/** difectory post container start-up (since symlinks are generated only when starting up fresh container)

---

## Donations

If you feel obligated to donate, to further help and support all parties involved in the development of the HeavenClient project, you can donate using [this](https://paypal.me/pools/c/8frYNoobcY) link.

Please remember this is ONLY for the HeavenClient development and will only be used in the support of helping further develop the client. *Also please remember to support Nexon as this is not meant to replace anything Nexon offers*

Another important note to remember is that HeavenClient is a free open-sourced client developed for personal use. Do NOT pay for any services requested by anyone in regards to this client. It will always remain open and free of charge. There is no intent to publish this code with any payment in mind. If that ever changes, donations and disclaimers for donations will be removed.

[HeavenMS]:          https://github.com/ronancpl/HeavenMS
[Switch]:            https://github.com/lain3d/HeavenClientNX
[Linux]:             https://github.com/ryantpayton/HeavenClient/tree/linux
[Windows 8.1 SDK]:   https://developer.microsoft.com/en-us/windows/downloads/sdk-archive
[NoLifeWzToNx]:      https://github.com/ryantpayton/NoLifeWzToNx
[MapPretty.nx]:      https://drive.google.com/file/d/1F7nUgH3royIS75WsPlYwc7RtdiGPnzd_/view?usp=sharing
[1]:                 https://drive.google.com/file/d/1LKhOtdDWOFJs8eIvqSx8pvP1ZfHF2rpo/view?usp=sharing
[NoLifeNx]:          https://github.com/ryantpayton/NoLifeNx
[GLFW3]:             http://www.glfw.org/download.html
[GLEW]:              http://glew.sourceforge.net/
[FreeType]:          http://www.freetype.org/
[Bass]:              http://www.un4seen.com/
[Asio]:              http://think-async.com/
[commit]:            https://github.com/HeavenClient/HeavenClient/commit/a3a95007495b6cded5e34af840a1354fd012c4e0
[HeavenClient.rar]:  https://drive.google.com/file/d/1--KgFBFqrD6_-07cD-S8dpEimbq5tq0t/view?usp=sharing
[2]:                 https://paypal.me/pools/c/8frYNoobcY

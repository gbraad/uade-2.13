WebUADE
=======

	Copyright (C) 2014 Juergen Wothke

	LICENSE
		See individual files for specific licensing information (most of UADE
		uses the terms of the GNU General Public License).


This is a JavaScript/WebAudio plugin of UADE. This plugin is designed to work with version 1.0 of my 
generic WebAudio ScriptProcessor music player (see separate project). 

WebUADE is based on "uade-2.13" and this project still contains most of the original files (even if some of them are 
not used in this context). Only the various unused "frontend" subfolders have been completely removed. 

Some of the files were modified (see explanations below) but it should be easy to identify respective changes either 
by diff-ing against an original/unchanged uade-2.13 distribution or by searching for the "EMSCRIPTEN" #ifdefs which 
mark the changes. (The structure of the original code was preserved to ease comparison.)

You'll need Emscripten (I used the win installer on WinXP: emsdk-1.13.0-full-32bit.exe which could (at the time) be 
found here: http://kripken.github.io/emscripten-site/docs/getting_started/downloads.html) I did not need to perform ANY 
additions or manual changes on the installation. The below instructions assume that the uade-2.13 project folder has 
been moved into the main emscripten installation folder (maybe not necessary) and that a command prompt has been opened 
within the project's "emscripten" sub-folder, and that the Emscripten environment vars have been previously set (run 
emsdk_env.bat).


Howto build:

The original "uade-2.13" makefiles must NOT be used because they might overwrite some of the manually performed changes. 
Instead the Web version is built using the makeEmscripten.bat that can be found in this folder. The script will compile 
directly into the "emscripten/htdocs" example web folder, were it will create the backend_uade.js library. The content of 
the "htdocs" can be tested by first copying into some document folder of a web server. You then need to manually create 
an 'uade' subfolder and copy the "players" and "amigasrc" sub-folders into it - also create a 'songs' folder here to 
hold the music files. A running example can be found here: http://www.wothke.ch/webuade

Background information:

This is a short summary how the code was derived from the original "uade-2.13" (maybe this helps if ever the code 
needs to be migrated to a different version of UADE).

- I used Cygwin on WinXP to "./configure" and "make" the original "uade-2.13". This created the basic file layout that 
you'll find in this project.

- The original code uses a two process design where a "frontend" process interacts with a separate "emulator core" 
process. For the web version, the separate "frontend" is completely eliminated (and with it all the IPC based 
communication). The relevant APIs previously invoked via IPC are instead exposed such that they can be invoked 
directly from JavaScript.

- Some of the logic of the original frontend was added directly to the "emulator core" (e.g. Eagleplayer stuff).

- Any file loading is diverted to a central API (see standalonesupport.c which replaces the original unixsupport.c) - 
which is serviced from the HTML5/JavaScript side. Due to the asynchronous nature of Web based file loading, the 
respective API uses a slightly extended error handling model. Any code triggering a "file load" had to be extended 
to deal with the specific error scenario that a requested file is not yet available. To recover from such errors the 
JavaScript side player uses a "retry from scratch" approach that is triggered as soon as a loaded file becomes available.

- Whereas the original emulator permanently runs in a loop (see m68k_run_1()), the code was changed to now just run long
enough to produce the next batch of audio samples (see emu_compute_audio_samples()). The JavaScript side is in control - 
not the emulator.

- All the emulator APIs relevant for the JavaScript side are located in the new adapter.c. 

- The callback.js library provides JavaScript functionalities that are directly compilied into the emulator (mainly used 
for the interactions with the "native" JavaScript portions of the player). 




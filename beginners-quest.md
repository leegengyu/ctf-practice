# Overview
* Link to site: https://capturetheflag.withgoogle.com/#beginners/

# [easiest - task: misc] Enter Space-Time Coordinates
* There is a file for us to [download](https://storage.googleapis.com/gctf-2019-attachments/00c2a73eec8abb4afb9c3ef3a161b64b451446910535bfc0cc81c2b04aa132ed) - `00c2a73eec8abb4afb9c3ef3a161b64b451446910535bfc0cc81c2b04aa132ed.zip`. The file name is static - but looks like it could be a hash to me. Ran a check on `hash-identifier` which said that it could be a `SHA-256` or `Haval-256` hash. Used an online cracker to see if we could find anything but nothing turned up.
* *Digression: First time that I see a hash named Haval so decided to run a search on what it is - a one-way hashing algorithm designed in 1992 and apparently no successful attacks have been reported on it so far. Thus, it can apparently serve as a "drop-in" replacement of MD5.*
* Within the zip file we find 2 files - `log.txt` and `rand2`.
* In the log file we find 5 lines of names and numbers - not sure what they mean.
* `rand2` appears to be an "ELF 64-bit LSB pie executable".
* Running `rand2` requires us to enter the destination's x and y coordinate:
![](/screenshots/google-beginner-space-time/rand2Execute.jpg)
* I tried a few random ones but the output was simply that we arrived at somewhere where the flag is not at.
* I looked into using `strings`, and managed to find the flag with `strings rand2 | grep flag`!
![](/screenshots/google-beginner-space-time/flag.jpg)
* Our flag is `CTF{welcome_to_googlectf}`.
* That was really quick - indeed it was befitting of the `easiest` label.

# [easier - task: networking] Satellite
* There is a file for us to [download](https://storage.googleapis.com/gctf-2019-attachments/768be4f10429f613eb27fa3e3937fe21c7581bdca97d6909e070ab6f7dbf2fbf) - `768be4f10429f613eb27fa3e3937fe21c7581bdca97d6909e070ab6f7dbf2fbf.zip`.
* Within the zip file we find 2 files - `README.pdf` and `init_sat`.
* In the README file, it tells us that we should "load init_sat on our terminal".
![](/screenshots/google-beginner-satellite/readmePDF.jpg)
* `init_sat` appears to be an "ELF 64-bit LSB executable".
* I tried to use the previous challenge's method of searching for the `flag` string and similar words, but could not find it.
* `strings init_sat` did show a lot of results - the chunk from somewhere in the middle to the end contains a lot more readable information than the initial blocks. Googling one of the strings "regexp.(*Regexp).put" leads us to discover that it is built with `Go`.
* *Digression: I found a useful [article](https://securitytraning.com/ctf-challenge/) that talks about determining the file type and identifying strings embedded in an executable and much more.*
* When I ran `init_sat`, the prompt asked for a "name of the satellite to connect to":
![](/screenshots/google-beginner-satellite/initSatError.jpg)
* I tried several random names but they were all unrecognised. Looking back at the `README.pdf`, we see the red scribbling `osmium`. (I was stuck at this point because silly me saw the word `osmicm` instead)
* Enter `osmium` as the satellite to connect to and we are connected!
![](/screenshots/google-beginner-satellite/initSatConnected.jpg)
* We get 3 options - `a` to display config data, `b` to erase all data and `c` to disconnect.
* Let us start with the first option:
![](/screenshots/google-beginner-satellite/configData.jpg)
* A Google Docs link jumps out to us: https://docs.google.com/document/d/14eYPluD_pi3824GAFanS29tWdTcKxP_XUxx7e303-3E.
* Opening the document gives us a string `VXNlcm5hbWU6IHdpcmVzaGFyay1yb2NrcwpQYXNzd29yZDogc3RhcnQtc25pZmZpbmchCg==`.
* I had a hunch that it is a base-64 encoded string, so I entered `data:text;base64,VXNlcm5hbWU6IHdpcmVzaGFyay1yb2NrcwpQYXNzd29yZDogc3RhcnQtc25pZmZpbmchCg==` into the browser, and we got 2 lines: `Username: wireshark-rocks` and `Password: start-sniffing!`
* So I guess that's a clear hint to start using Wireshark!
* I started capturing packets the moment I re-executed `init_sat`, and found DNS queries being made before we see the TCP stream:
![](/screenshots/google-beginner-satellite/wiresharkPacketList.jpg)
* Starting with the first TCP packet, we start examining the contents of the packet, and I found  packet #10 (for me) containing the welcome message that we saw when we executed the program and was connected.
* Moving further downwards, I found the flag in packet #14, where the username and password is found. I right-clicked on the packet and selected Follow > TCP Stream:
![](/screenshots/google-beginner-satellite/flag.jpg)
* Our flag is `CTF{4efcc72090af28fd33a2118985541f92e793477f}`, found in the originally censored password field (*I find it interesting how we are able to see the password in plaintext in the packet while it is censored when we executed it in the terminal - hmm?*).
* This sounds silly, but I had always assumed that local executables had everything running locally with no connection to the Internet at all.

# [easier - task: forensics] Home Computer
* Reference link: https://ctftime.org/writeup/15941
* There is a file for us to [download](https://storage.googleapis.com/gctf-2019-attachments/86863db246859897dda6ba3a4f5801de9109d63c9b6b69810ec4182bf44c9b75) - `86863db246859897dda6ba3a4f5801de9109d63c9b6b69810ec4182bf44c9b75.zip`.
* Within the zip file we find 2 files - `family.ntfs` and `note.txt`.
* In the note file, we find that it is just a recommendation to rename the .ntfs file to .dmg if the user is on MacOS.
* Running `file family.ntfs` tells us that it is a DOS/MBR boot sector.
* Note: NTFS stands for New Technology File System, and is used by Windows.
* Since it is a file system and not a directory, we are not able to directly change our directory into it (i.e. cannot use `cd`).
* "In order to actually access the contents of the filesystem, we need to mount it to our own system at a specified mount point. Only after mounting the file system, can we actually access it as if it were a directory."
* Heading to my `/mnt` directory, I find that it is empty. I create a directory with `mkdir forensics` just for this challenge's purpose.
* The `forensics` directory is known as our mounting point. Next, we will mount the NTFS file system to the mounting point using `mount -t ntfs /root/Downloads/family.ntfs /mnt/forensics`. A successful execution of the command gives no output.
* Heading into the `forensics` directory, we see a list of directories and files:
![](/screenshots/google-beginner-home-computer/fileSystemContents.jpg)
* I navigated into `Users`, then `Family` (because that was the only directory). There were several directories in there - `Desktop`, `Documents`, `Downloads`, `Pictures` and `Videos`. I started exploring the directories from the top of the list.
* `Desktop` had nothing interesting within, but `Documents` had 3 files - `credentials.txt`, `document.pdf` and `preview.pdf`.
* `credentials.txt` tells us that pictures of the user's credentials are kept in extended attributes.
* Searching about `extended attributes NTFS` tells us that they are stored on the file system as alternate data streams "whose name is the unprefixed name of the attribute, and whose contents is the value of the attribute". They appear to be difficult to find as well.
* Run `fls -Fr family.ntfs | grep credentials.txt` to see what turns up besides the file which we looked into just now.
![](/screenshots/google-beginner-home-computer/grepCredentialsTxt.jpg)
* Note: `fls` lists file and directory names in a disk image. `-F` displays file (non-directory) entires only, and also lists the full path of the file that turns up in the result. `-r` searches directories recursively.
* We can see that there is another file `Users/Family/Documents/credentials.txt:FILE0`, which is the extended attributes file. *Though I think that it might be possible that the extended attribute file could do as a stand-alone file with just the name `FILE0`?* That way it would make it alot harder to find it.
* The identifier for this newly discovered file is `13288-128-4`.
* Next, we will use `icat`, which outputs the contents of a file based on its inode number.
* Just as a try-out first, running `icat family.ntfs 13288-128-2` shows us the one-liner content in `credentials.txt` as expected.
* Running `icat family.ntfs 13288-128-4` displays a whole bunch of data that we cannot understand, so let us redirect the contents of it to a new file with `> unknown` attached to the `icat` command.
* **Learning point**: We can run `icat family.ntfs 13288-128-4 | file -` to find out what kind of file it is without having to re-direct the output of `icat` to a new file first, then running `file`. The only unusual part here that I learnt about is `file -`, where the command `file` reads the parameter from `stdin`. I had always thought that we needed to supply the file name to `file` for the command to work - *perhaps a temporary file is created from `stdin` in its underlying implementation?*.
* `file unknown` tells us that it is a PNG image data.
* Opening up the file tells us the flag `CTF{congratsyoufoundmycreds}`!
* **Learning point**: `xdg-open unknown` allows us to open the image file straight-up without having to open up `Files`, then clicking our way to the file.
![](/screenshots/google-beginner-home-computer/grepCredentialsTxt.jpg)
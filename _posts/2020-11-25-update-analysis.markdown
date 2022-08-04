---
layout: post
title:  "Reverse Engineering Vehicle Update Files"
date:   2020-11-25 16:00:00 -0400
categories: Reverse Engineering / Analysis
---
# Backstory / Introduction
I own a relatively new Dodge vehicle, one new enough to have a touch screen infotainment system, and require system updates. Well recently my vehicle was prompted with an update, and I became curious as to what this update file actually contained. So I navigated to Dodge's update website, downloaded the respective update file, and tried to see what I could learn about it. These are some of the things I learned and was able to find out about this file.

# Update Contents
The first thing I wanted to do was see what files I had to work with. Unzipping the file presented me with the following files:
```
checkswdl.bat
KaliSWDL.log
md5deep
swdl.upd
swdl.upd.md5
```
## KaliSWDL.log
This file contains the output of what I assume is the compilation or packing of this update file. Not too much interesting information is found here. 

## md5deep.exe
This file is an executable that performs and md5 hash of a file. This is used later to check the update file against a known correct hash.

## checkswdl.bat
'checkswdl' is a batch script that runs the output of md5deep with swdl.upd and compares it against swdl.upd.md5. This ensures the hash is correct and that the file was successfully downloaded without corruption or issues. It also contains these cool ASCII art drawings to give you visual of whether or not the file passes the hash check.
### File Passed:
```
             _@_
            ((@))
            ((@))
            ((@))
 ______===(((@@@@====)
 ##########@@@@@=====))
 ##########@@@@@----))
 ###########@@@@----)
 ========-----------
:
 !!!  FILE IS GOOD  !!!!
:
```
### File Failed:
```
  .=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-.
  |                     ______                     |
  |                  .-"      "-.                  |
  |                 /            \                 |
  |     _          |              |          _     |
  |    ( \         |,  .-.  .-.  ,|         / )    |
  |     > "=._     | )(__/  \__)( |     _.=" <     |
  |    (_/"=._"=._ |/     /\     \| _.="_.="\_)    |
  |           "=._"(_          _)"_.="             |
  |               "=\__|IIIIII|__/="               |
  |              _.="| \IIIIII/ |"=._              |
  |    _     _.="_.="\          /"=._"=._     _    |
  |   ( \_.="_.="     `--------`     "=._"=._/ )   |
  |    > _.="                            "=._ <    |
  |   (_/                                    \_)   |
  |                                                |
  '-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-='
 :
  ***   FILE IS BAD   ***
 :
```

## swdl.upd.md5
This is the known good hash of the update file. This again is used in checkswdl.bat in order to ensure the update file was downloaded correctly.
```
MD5(swdl.upd)= 536fb9f643beb374bc9be69d129436b3
```

## swdl.upd
Finally that brings us to swdl.upd, this is the meat and potatoes of the whole post. This is the update file and contains the interesting stuff I really want to take a look at. The rest of this post will be the dive into figuring out how to look at the stuff found in this file. 

# Dissecting the Update
Now that we know everything contained in this file, how do I actually find the interesting stuff to view within the file?

## What kind of file is this anyway?
Well in order to find out what kind of file I am working with, I first decided to use the wonderful linux tool *file*. Here you can see the return from this tool.
```
$ file swdl.upd
swdl.upd: ISO 9660 CD-ROM filesystem data 'CDROM'
```
So it appears to be a CD-ROM file, we can actually confirm this by taking a look at the KaliSWDL.log file we found earlier. 
```
Warning: Creating ISO-9660:1999 (version 2) filesystem.
```
Near the bottom of this log we can find this line, which references the ISO-9660 filesystem being created. 

## What's in this thing?
The next step I wanted to do was to see what I could find out about the contents of this file. To do this is pretty straight forward as I used another tool called *binwalk* in order to see what it could find out about the file and its contents. Now I won't post the full output of the tool, as it is quite large. However I will provide a few snippets that allows you to see exactly what has been output from the tool. 
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
172032        0x2A000         ELF, 32-bit LSB executable, ARM, version 1 (SYSV)
245760        0x3C000         ELF, 32-bit LSB executable, ARM, version 1 (SYSV)
339800        0x52F58         Unix path: /etc/system/config/nand_partition.txt
363816        0x58D28         Unix path: /../../libgcc/../gcc
365162        0x5926A         Unix path: /../../libgcc/../gcc/config/arm
365683        0x59473         Unix path: /../../libgcc/../gcc
367524        0x59BA4         Unix path: /../../libgcc/../gcc/libgcc2.c
400331        0x61BCB         Unix path: /etc/system/config/nand_partition.txt )
427192        0x684B8         Unix path: /fs/etfs/tmp/tracelog/traces.log
427259        0x684FB         Unix path: /fs/etfs/tracescopes/links/HMI.hbtc
427321        0x68539         Unix path: /fs/etfs/usr/share/flash/scopes/scopelist
```
From this first snippet of showing the first few files discovered in the file, you can see some interesting things. Firstly there are two ELF files that appear to be compiled for ARM, these are interesting and I may have to dissect them in their own post sometime in the future. But overall you can see that the output is providinga hexadecimal offset to each different file provided in the output. As well as a description, or filepath name. As this output is 75,000 lines long I will spare you the full log. However I will be highlighting specific snippets as I pulled out data from this file. 

## "Cutting out" files
Now that I essentially had a map of the files contained within this file, I can begin cutting out files I have an interest in. To do this I will use another linux tool, *dd*. This tool functions by directly copying chunks of data from specific offsets of a file to another file. Since I have the offset of each file and the file following it. I can simply copy the middle section between those offsets in order to grab the files I desire. Let's find something I want to extract:
```
550015658     0x20C892AA      Zip archive data, at least v2.0 to extract,   
    compressed size: 1527, uncompressed size: 1821, name: res/images/logo-lrg.png
550017238     0x20C898D6      Zip archive data, at least v2.0 to extract, 
    compressed size: 1167, uncompressed size: 1473, name: res/images/logo-sml.png
```
So here we can see this *logo-lrg.png* file I desire to pull out of the update. Using *dd* and the provided offsets I should be able to pull this file out relatively easily. 
```
dd if=swdl.upd of=logo-lrg.zip bs=1 skip=$((0x20C892AA)) count=$((0x20C898D6-0x20C892AA))
```
I'll quickly explain what each option does in this command:
### if=
    You specify the input file to the tool, or the file you will be extracting from.
### of= 
    You specify the output file here, or the file you will be writing to. 
### bs=
    You can specify the block size to be copied here, you can make this larger to copy things faster. However if you make this larger you will have to divide the next options by this same value in order to account for the change in block size. I will leave this '1' for clarity's sake. 
### skip=
    Here you give the offset to the beginning of the file you wish to extract. 
### count=
    This is how many blocks of data you will be copying from the file. In order to get this value, you can subtract the next file's initial offset from the file you wish to copy. The difference between these values is the number of 'blocks' in the file. In our case since a block is a single byte, it is the number of bytes between the two offsets. 

Running this command given above provides the following output:
```
1580+0 records in
1580+0 records out
1580 bytes (1.6 kB, 1.5 KiB) copied, 0.0219515 s, 72.0 kB/s
```
And also we can see a new *logo-lrg.zip* file in our directory. So let's open that up and view our picture. 

![WinrarError](/assets/update-analysis-winrarError.png) 

Erm, what happened? I thought this would work. I can see the files within the zip, but when I attempt to extract the png to view it I keep getting this error. Well after checking... and checking... *and checking* my *dd* options to ensure I did everything correct I finally figured out you can just repair archives through Winrar. Well let's give this a shot... 

![WinrarRepair](/assets/update-analysis-winrarRepair.png) 

### TA-DA!
I successfully pulled out the logo file and prove now I can get any file I want from this update file.

![LogoFile](/assets/update-analysis-Logo.png) 

## What else can I find?
Now that we can successfully pull files out of this update, what is there for me to find within this file? Aside from random images and the two ELF binaries I found earlier, most of what I found are random text files and a whole heck of a lot of *.class* files. After digging into a bunch of text files finding nothing, I decided to figure out how to examine these *.class* files. 

## Decompiling .class files
So I got one of these *.class* files extracted, now what? Now I have never worked with Java before, however I know enough to at least recognize this is a Java. So I began digging to find out how to pull this file apart and see what's inside. Opening it directly in a text editor is no help as it just seems like a garbled mess of characters. This led me to believe that it was compiled in some way, *(again I know very little about Java)*. So I began searching for ways to decompile Java files. I came across a tool simply called *Java Decompiler*, specifically I was interested in using *JD-GUI* a standalone utility that would hopefully allow me to decompile this class file. Opening this tool, you are presented with a very easy to use GUI that just lets you navigate to the file you wish to decompile and open it. 

![JD-GUI Home](/assets/update-analysis-jdguiOpen.png) 

The file I decided to try and decompile first was a file called *SteeringWheelAngleUpdate.class*. Success! 

```
import GuiData.SensorDataUpdate;
import InternalInterface.SteeringWheelAngleUpdate;
import com.uniquesoft.tdl.system.SystemUtils;
import com.uniquesoft.tdl.system.TDLObject;

public class SteeringWheelAngleUpdate implements TDLObject {
  public SensorDataUpdate sensor_data;
  
  public void destroy() {}
  
  public boolean equals(SteeringWheelAngleUpdate copy) {
    return SystemUtils.tdlEquals(this.sensor_data, copy.sensor_data);
  }
  
  public boolean excleq(TDLObject copy) {
    return !equals(copy);
  }
  
  public boolean equals(Object copy) {
    if (getClass() == copy.getClass())
      return equals((SteeringWheelAngleUpdate)copy); 
    return false;
  }
  
  public int hashCode() {
    return 0x1 ^ ((this.sensor_data == null) ? 0 : this.sensor_data.hashCode());
  }
  
  public SteeringWheelAngleUpdate deepCopy() {
    return new SteeringWheelAngleUpdate(this);
  }
  
  public SteeringWheelAngleUpdate(SteeringWheelAngleUpdate copy) {
    this.sensor_data = copy.sensor_data;
  }
  
  public SteeringWheelAngleUpdate() {}
  
  public SteeringWheelAngleUpdate(SensorDataUpdate sensor_data) {
    this.sensor_data = sensor_data;
  }
}
```
Opening this file within JDGUI I was greeted with a decompiled version of the original Java file. I now had a way to view any of these Java files I desired. 

## That's all! *For Now...*
Well I accomplished my original goal of learning what exactly is in these update files. Now I must decide what I want to do next? I originally went into this desiring to make my own custom themes for the system. I did manage to find the theme files in the Shockwave Flash format, so there is potential there to build my own custom ones. Maybe I'll reverse engineer those ELF Binaries I found to see what I can find. For now though I am satisfied with what I have learned so far.

## Thanks for reading
I hope you found something in this post entertaining or educational! I post these in order to inspire others to work on projects of their own. If you want to share any projects with me or ask any questions feel free to shoot me an email provided at the footer of this page. Thanks!
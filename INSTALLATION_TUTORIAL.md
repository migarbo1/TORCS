# **Custom installation for TORCS**

## **Sources and references**

- [PLIB installation help](https://github.com/vog3lm/torcs-1.3.7)
- [Patch to restart races]( https://github.com/Gastd/scr-meta-restart/tree/master)
- [OpenAL compilation problem](https://unix.stackexchange.com/questions/444874/problem-with-torcs-installation-error-with-openal)

## **Installation**

### 1. Download torcs from SourceForge

The files can be found [here](https://sourceforge.net/projects/torcs/files/latest/download)

### 2. Install torcs dependencies

```sudo apt-get install mesa-utils libalut-dev libvorbis-dev cmake libxrender-dev libxrender1 libxrandr-dev zlib1g-dev libpng-dev libxxf86vm-dev```

We may need also to install `freeGlut`, so:

```sudo apt-get install freeglut3 freeglut3-dev```

And, in order to automatically select competition modes, we need the xautomation module. It can be installed using:

```sh
sudo apt-get install xautomation
```

Finally, for some more modern linux versions (+18.0.0), we may need to install also `PLIB`. In that case we will need also the following packages:

```sudo apt-get install libxmu-dev libxmu6 libxi-dev```

### 3. Install PLIB (if needed)

First of all we have to download the binaries and unzip them:

```sh
wget https://plib.sourceforge.net/dist/plib-1.8.5.tar.gz
sudo tar xfvz plib-1.8.5.tar.gz
cd plib-1.8.5
```

The next step is then to set the enviroment variables for the compilation. **If this is not set the installation may fail**.

```sh
export CFLAGS="-fPIC"
export CPPFLAGS=$CFLAGS
export CXXFLAGS=$CFLAGS
```

Now we can install PLIB. To do so, type the following:

```sh
./configure
make
sudo make install
```

In order of appearance, we are checking all the required dependencies as well as configuring all the parameters for the compilation and installation.
If this step has succeded we can delete the previous environment flags, altough this is not necesary.

### 4. TORCS code modifications

**Delete countdown**

This is  fairly easy thing to do. We only have to open the file `/src/libs/raceengineclient/raceengine.ccp` under the `torcs-1.3.7` folder and comment out the llines 619 to 625.

> **_NOTE:_**  Line numbers may differ if you have used a different source to download TORCS. In that case you only have to look for the text 'READY' or 'GO'.

**Patch to restart races**

To perform this modification we need to dig deeper into the torcs folder structure. In fact we will modify 3 core files: `scr_server.cpp` (the bot controller), `raceengine.cpp` (the race engine as it name says) and `car.h` (the car interface)

> **_NOTE:_** The routes that you need to acces for this modifications may have not been there at this step. In that case, please **INSTALL TORCS** first and then come to this section.

The first thing that we need to do is to modify the car interface to add a field that will reflect if the bot has requested a reset or not. To do so we open the file `/src/interfaces/car.h` with our favorite editor. Inside the file we are going to search the following string: `} tCarCtrl;` and above that line we are going to place our custom attribute:

```C
    int        askRestart;
```

So now we need to open the file `/src/drivers/scr_server/scr_server.cpp` and search for the following sequence: `#ifdef __DISABLE_RESTART__`. In the inmediatelly **previous** line we are going to introduce the following code:

```C
car->ctrl.askRestart = true;
std::cerr << "Restarting the race" << endl;
```

This basically tells the controller that the bot has requested the race to be restarted and stores it in a log. The next action then is to modify the controller of the race to perform this reset. To do so we are going to open the file `/src/libs/raceengine/raceengine.cpp` and just above the lines that we have commented to remove the countdown we are going to place the following code:

```C
    bool restartRequested = false;

    for(i = 0; i < s->_ncars;i++){
        if(s->cars[i]->ctrl.askRestart){
            restartRequested = true;
            s->cars[i]->ctrl.askRestart = false;
        }
    }

    if(restartRequested){
        ReRaceCleanup();
        ReInfo->_reState = RE_STATE_PRE_RACE;
        GfuiScreenActivate(ReInfo->_reGameScreen);
    }
```

In case that the code is not suficiently self-explanatory, this loops over all the agents playing and if one has requested for the race to be reset, so it is done.

**Compile and install torcs**

We are finally here! With all the modifications done we can proceed to compile and install TORCS in our PC. To do so we proceed as follows:

First of all we have to compile the `.cpp` files taht compose TORCS:

```sh
make 
```

> **_NOTE:_**  during the compilation a problem related to openAl may appear. This is a typing error that can be solved replacing the line with the issue `const char* error = '\0';` with the following line that corrects it: `const char* error = nullptr;`

With the files compiled, we can proceed to install the program:

```sh
sudo make install
sudo make datainstall
```

## **Modify server_1 default car**

In our case, we have trained our RL agent with an F1-like car. To do so we needed to perform some modifications that may not be trivial. That's why we are also displaying how to do it in this guide.

The first tricky part is to know from where is reading torcs the configuration for the different drivers. In our case, we are using the first bot, so we will find it's configuration under the following path: `/usr/local/share/games/torcs/drivers/scr_server/1`

```sh
cd /usr/local/share/games/torcs/drivers/scr_server/1
```

> **_NOTE:_** The origin of this path may differ if you have installed torcs with sudo or not. In our case, as in the commands above, we have used sudo.

There we are going to create a folder to contain the previous files, in case that we want to revert our changes. And after that we are going to move all the files inside that folder:

```sh
mkdir old
mv car1-trb1.rgb default.xml old/
```

Now we need to copy to our current folder the `.rgb` file of the desired car model. In cour case this can be done with the command:

```sh
cp /usr/local/share/games/torcs/cars/car1-ow1/car1-ow1.rgb .
```

Finally, we are going to create a `default.xml` file that will contain the default behaviour of our car. This means the basic states of the different parameters that it has (for example amplitude of front wing, gear correspondence...) This information can be observed at the desired car info file. Different cars and their recpective files are located under the folder: `/usr/local/share/games/torcs/cars/`

For completitude, we display here our `default.xml` corresponding to have an F1 car as default.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 
    file                 : car1-trb1.xml
    created              : Thu Sep 21 20:37:54 CET 2006
    copyright            : (C) 2006 by Bernhard Wymann
    email                : berniw@bluewin.ch
    version              : $Id: default.xml,v 1.1.2.1 2008/05/29 23:49:46 berniw Exp $
-->

<!--    This program is free software; you can redistribute it and/or modify  -->
<!--    it under the terms of the GNU General Public License as published by  -->
<!--    the Free Software Foundation; either version 2 of the License, or     -->
<!--    (at your option) any later version.                                   -->

<!DOCTYPE params SYSTEM "../../../../src/libs/tgf/params.dtd">
    
<params name="car1-ow1" type="template">
    
    <section name="Car">
        <attnum name="initial fuel" unit="l" min="1.0" max="100.0" val="100.0"/>
    </section>
            
    <section name="Front Wing">
        <attnum name="angle" unit="deg" min="0" max="25" val="13"/>
    </section>
    
    <section name="Rear Wing">
        <attnum name="angle" unit="deg"  min="0" max="35" val="10"/>
    </section>
        
    <section name="Gearbox">
        <attnum name="shift time" val="0.05" unit="s"/>
        <section name="gears">
            <section name="r">
                <attnum name="ratio" min="-6" max="-3.0" val="-4.0"/>
            </section>
            
            <section name="1">
                <attnum name="ratio" min="0.5" max="5" val="3.9"/>
            </section>
            
            <section name="2">
                <attnum name="ratio" min="0.5" max="5" val="2.9"/>
            </section>
            
            <section name="3">
                <attnum name="ratio" min="0.5" max="5" val="2.3"/>
            </section>
            
            <section name="4">
                <attnum name="ratio" min="0.5" max="5" val="1.87"/>
            </section>
            
            <section name="5">
                <attnum name="ratio" min="0.5" max="5" val="1.68"/>
            </section>
            
            <section name="6">
                <attnum name="ratio" min="0.5" max="5" val="1.54"/>
            </section>
            
            <section name="7">
                <attnum name="ratio" min="0.5" max="5" val="1.46"/>
            </section>
            
            
        </section>
    </section>
            
    <section name="Brake System">
        <attnum name="front-rear brake repartition" min="0.3" max="0.7" val="0.47"/>
        <attnum name="max pressure" unit="kPa" min="5000" max="60000" val="24000"/>
    </section>
    
    <section name="Rear Differential">
        <!-- type of differential : SPOOL (locked), FREE, LIMITED SLIP -->
        <attstr name="type" in="SPOOL,FREE,LIMITED SLIP" val="LIMITED SLIP"/>
        <attnum name="ratio" min="1.0" max="10" val="4.5"/>
    </section>
    
    <section name="Front Right Wheel">
        <!-- initial ride height -->
        <attnum name="ride height" unit="mm" min="50" max="150" val="90"/>
        <attnum name="toe" unit="deg" min="-2" max="2" val="0"/>
        <attnum name="camber" min="-5" max="0" unit="deg" val="-4"/>
    </section>
    
    <section name="Front Left Wheel">
        <!-- initial ride height -->
        <attnum name="ride height" unit="mm" min="50" max="150" val="90"/>
        <attnum name="toe" unit="deg" min="-2" max="2" val="0"/>
        <attnum name="camber" min="-5" max="0" unit="deg" val="-4"/>
    </section>
    
    <section name="Rear Right Wheel">        
        <!-- initial ride height -->
        <attnum name="ride height" unit="mm" min="50" max="150" val="100"/>
        <attnum name="toe" unit="deg" min="-2" max="2" val="0.15"/>
        <attnum name="camber" min="-5" max="0" unit="deg" val="-1.5"/>
    </section>
    
    <section name="Rear Left Wheel">
        <!-- initial ride height -->
        <attnum name="ride height" unit="mm" min="50" max="300" val="100"/>
        <attnum name="toe" unit="deg" min="-2" max="2" val="-0.15"/>
        <attnum name="camber" min="-5" max="0" unit="deg" val="-1.5"/>        
    </section>
    
    <section name="Front Anti-Roll Bar">
        <attnum name="spring" unit="lbs/in" min="0" max="5000" val="0"/>
    </section>
    
    <section name="Rear Anti-Roll Bar">
        <attnum name="spring" unit="lbs/in" min="0" max="5000" val="0"/>
    </section>
    
    <section name="Front Right Suspension">
        <attnum name="spring" unit="lbs/in" min="500" max="10000" val="2000"/>
        <attnum name="suspension course" unit="m" min="0" max="0.25" val="0.25"/>
        <attnum name="bellcrank" min="0.1" max="5" val="1.5"/>
        <attnum name="packers" unit="mm" min="0" max="30" val="0"/>
        <attnum name="slow bump" unit="lbs/in/s" min="20" max="1000" val="80"/>
        <attnum name="slow rebound" unit="lbs/in/s" min="20" max="1000" val="50"/>
        <attnum name="fast bump" unit="lbs/in/s" min="10" max="200" val="30"/>
        <attnum name="fast rebound" unit="lbs/in/s" min="10" max="200" val="30"/>
    </section>
    
    <section name="Front Left Suspension">
        <attnum name="spring" unit="lbs/in" min="500" max="10000" val="2000"/>
        <attnum name="suspension course" unit="m" min="0" max="0.25" val="0.25"/>
        <attnum name="bellcrank" min="0.1" max="5" val="1.5"/>
        <attnum name="packers" unit="mm" min="0" max="30" val="0"/>
        <attnum name="slow bump" unit="lbs/in/s" min="20" max="1000" val="80"/>
        <attnum name="slow rebound" unit="lbs/in/s" min="20" max="1000" val="50"/>
        <attnum name="fast bump" unit="lbs/in/s" min="10" max="200" val="30"/>
        <attnum name="fast rebound" unit="lbs/in/s" min="10" max="200" val="30"/>
    </section>
    
    <section name="Rear Right Suspension">
        <attnum name="spring" unit="lbs/in" min="500" max="10000" val="3000"/>
        <attnum name="suspension course" unit="m" min="0" max="0.25" val="0.25"/>
        <attnum name="bellcrank" min="0.1" max="5" val="1.5"/>
        <attnum name="packers" unit="mm" min="0" max="30" val="0"/>
        <attnum name="slow bump" unit="lbs/in/s" min="20" max="1000" val="80"/>
        <attnum name="slow rebound" unit="lbs/in/s" min="20" max="1000" val="150"/>
        <attnum name="fast bump" unit="lbs/in/s" min="10" max="200" val="50"/>
        <attnum name="fast rebound" unit="lbs/in/s" min="10" max="200" val="50"/>
    </section>
    
    <section name="Rear Left Suspension">
        <attnum name="spring" unit="lbs/in" min="500" max="10000" val="3000"/>
        <attnum name="suspension course" unit="m" min="0" max="0.25" val="0.25"/>
        <attnum name="bellcrank" min="0.1" max="5" val="1.5"/>
        <attnum name="packers" unit="mm" min="0" max="30" val="0"/>
        <attnum name="slow bump" unit="lbs/in/s" min="20" max="1000" val="80"/>
        <attnum name="slow rebound" unit="lbs/in/s" min="20" max="1000" val="150"/>
        <attnum name="fast bump" unit="lbs/in/s" min="10" max="200" val="50"/>
        <attnum name="fast rebound" unit="lbs/in/s" min="10" max="200" val="50"/>
    </section>
    
    <section name="Front Right Brake">
        <attnum name="disk diameter" unit="mm" min="100" max="380" val="380"/>
        <attnum name="piston area" unit="cm2" val="50"/>
        <attnum name="mu" val="0.3"/>
        <attnum name="inertia" unit="kg.m2" val="0.1241"/>
    </section>
    
    <section name="Front Left Brake">
        <attnum name="disk diameter" unit="mm" min="100" max="380" val="380"/>
        <attnum name="piston area" unit="cm2" val="50"/>
        <attnum name="mu" val="0.3"/>
        <attnum name="inertia" unit="kg.m2" val="0.1241"/>
    </section>
    
    <section name="Rear Right Brake">
        <attnum name="disk diameter" unit="mm" min="100" max="380" val="350"/>
        <attnum name="piston area" unit="cm2" val="25"/>
        <attnum name="mu" val="0.3"/>
        <attnum name="inertia" unit="kg.m2" val="0.0714"/>
    </section>
    
    <section name="Rear Left Brake">
        <attnum name="disk diameter" unit="mm" min="100" max="380" val="350"/>
        <attnum name="piston area" unit="cm2" val="25"/>
        <attnum name="mu" val="0.3"/>
        <attnum name="inertia" unit="kg.m2" val="0.0714"/>
    </section>
</params>
```

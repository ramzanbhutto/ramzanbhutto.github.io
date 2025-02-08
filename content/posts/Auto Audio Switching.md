+++
date = '2025-02-08T00:49:53+05:00'
title = 'Auto-Switch Audio in Arch Linux(LARBS): Speakers to Headphones'
tags = ["Arch","Pipewire","Pulseaudio"]
+++
### Automatically Switching Audio Output Between Speakers and Headphones on Arch Linux: A Comprehensive Guide

---

## Method 1:Using Pavucontrol(Graphical Method)

If you prefer a more user friendly environment, then go for pavucontrol.

### Step 1:Install **pavucontrol** if it's not already installed:
~~~
sudo pacman -S pavucontrol
~~~

### Step 2: Launch **pavucontrol**:
~~~
pavucontrol
~~~

### Step 3: Navigate to the `Configuration` tab.
In pavucontrol, go to the Configuration tab. This section allows you to manage audio profiles for your devices.

### Step 4:  Select `Profile`
Locate your sound card and ensure the profile is set to a mode that supports both speakers and headphones, such as **Analog Stereo Duplex**.

### Step 5: Access Output `Devices`.
Navigate to the Output Devices tab to view all available audio outputs, such as speakers and headphones.

### Step 6: Choose Output `Port`
In the Output Devices tab, select the desired output port (e.g., headphones) to direct audio accordingly.

---

## Method 2: Using the Terminal (Command-Line Method):

For those who prefer the command line, you can achieve automatic switching by following steps:

### Step 1: Ensure you have `Pulseaudio` or `Pipewire` installed, if not then achieve it using:

~~~
sudo pacman -S pulseaudio
~~~
**or**
~~~
sudo pacman -S pipewire
~~~

### Step 2: Identify available audio sinks
First, list all available audio sinks (output devices) to determine the identifiers for your loudspeaker and headphones:
~~~
pactl list short sinks
~~~

### Step 3: Set Default Sink to Headphones
Once you have identified the sink name for your headphones(e.g, ***bluez_output.56_66_98_EX_56_9X.2***), set it as default sink:
~~~
pactl set-default-sink bluez_output.56_66_98_EX_56_9X.2
~~~
Replace ***bluez_output.56_66_98_EX_56_9X.2*** with the actual sink name of your headphones.

### Step 4: Move Existing Audio Streams:
To redirect current audio streams to the newly selected default sink (headphones), execute the following script in terminal that will move all active sudio streams to the default sink you set in the previous step.
~~~
for stream in $(pactl list short sink-inputs | awk '{print $1}'); do
    pactl move-sink-input $stream $(pactl get-default-sink)
done
~~~

---

In conclusion, by leveraging PulseAudio's command-line utilities, you can effectively manage audio streams on your Arch Linux system. The provided script automates the process of moving all active audio streams to the default sink, ensuring a seamless transition when switching output devices. This approach offers a streamlined and efficient method to control audio routing directly from the terminal, enhancing your overall user experience.

---

---
layout: post
title: patching my thinkpad t450's bios
---

I own a Lenovo Thinkpad T450. This laptop, while released in 2015, is still plenty powerful for many applications, and is very repairable. Because Thinkpads were business computers, nearly every part can be repaired or replaced by someone with even minimal tech skills. The RAM, storage, keyboard, screen -- wait, what? The screen can only be replaced by a genuine Lenovo part? And this whitelist is enforced in the BIOS?

Yep, that's right. Lenovo, in their infinite wisdom, decided to whitelist several parts on their Thinkpad line of laptops. As far as I know, this includes the display, WWAN card, and wireless (Wi-Fi) card. Each of these have some degree of functionality removed when using parts that are not on the whitelist.

For WWAN and Wi-Fi cards, this means that your computer just won't boot until you remove the card. Fortunately, Wi-Fi and WWAN cards that are on the whitelist are quite cheap, and about as good as cards that aren't on the whitelist.

Arguably the bigger problem, however, is the display. If you replace a display on certain Lenovo Thinkpads, brightness controls in Windows will not work unless the EDID of that display matches one that is flashed *by Lenovo*. You can buy the exact same model number of display, from the same factory, possibly even the same batch, but unless the EDID flashed onto the display is approved by Lenovo, you can't control the brightness in Windows. This locks you in to buying replacement parts that are either stripped from other Lenovo laptops, or directly from Lenovo. Very anti-repair behavior :/. 

Actually, this possibly violates Colorado's new right-to-repair law, HB24-1121, which prohibits the use of parts pairing ("A manufacturer's practice of using software to identify component parts through a unique identifier") to "... inhibit an ... owner's ability to install or enable, the function of an otherwise functional replacement part or component of digital electronic equipment, including a replacement part or component that the manufacturer has not approved." Thanks, Colorado! 

While I generally don't use Windows, I am of the opinion that brightness control is a pretty important part of a laptop. The display I swapped into my Thinkpad doesn't allow me to use that, thanks to Lenovo's whitelist.

There are essentially three solutions to this problem:
 - Patch the hardware (by flashing the EDID of a genuine display onto the replacement display so the driver allows you to use brightness controls)
 - Patch the software (by using an operating system that doesn't care about Lenovo's whitelist, such as Linux)
 - Patch the firmware (by removing the whitelist from the BIOS)

So far, I had gotten around this problem by using Linux, where brightness controls work perfectly. Unfortunately, every time I had to boot into Windows to use some piece of software that can't run under Linux or WINE, I was flashbanged by an annoyingly bright display. So I decided to patch the firmware.

## Reading the BIOS
I set out to patch the BIOS of the Thinkpad to remove the whitelist once and for all. Fortunately, others have done this before me, so I could follow in the footsteps of those greater than me (or something like that). Specifically, I followed some of the steps of [this](https://www.reddit.com/r/thinkpad/comments/ac7u21/tutorial_t450t450sw550st550x250_screen_whitelist/) Reddit post, and if you are looking to do the same thing, I would also read that post.

**IMPORTANT: YOU CAN BRICK YOUR LAPTOP DOING THIS. ONLY ATTEMPT THIS IF YOU ARE PREPARED FOR THE CONSEQUENCES.**

I have played around with a CH341 programmer in the past, so I figured, how hard could it be? Find the flash chip, stick a clip on it, read it, patch it, flash it, right? Well... sort of. Unfortunately, cheap CH341 programmers are almost always wired incorrectly, and this can cause data loss. Many (most) flash chips require 3.3V power, and 3.3V on the data lines. The CH341 can provide 3.3V power, but CH341 programmers are wired to supply 5V on the data lines -- possibly frying your flash chip for good. People have fixed this by soldering wires onto the board to connect the correct wires to the correct places, but I had a simpler solution: the Flipper Zero.

That's right! Everyone's favorite dolphin-themed electronic multi-tool has another use: an SPI programmer. By using the [SPI Mem Manager](https://lab.flipper.net/apps/spi_mem_manager) app for the Flipper, it's easy to read and write to an SPI flash chip. I decided to use this and the SOIC-8 clip I had from the CH341 programmer to read and write the BIOS. 

I disassembled the Thinkpad, disconnected both batteries and the CMOS battery, and removed the second SO-DIMM, and there it was: a 128Mbit flash chip. Out of an abundance of caution, I desoldered the flash chip from the board (which is quite easy with a chisel tip soldering iron and some tweezers). After reading the datasheet for the chip, I connected the test clip to my Flipper Zero. Pin 1 was connected to both a 10k pull-up resistor and to Flipper pin 4, pin 2 to Flipper pin 3, pins 3 and 7 to their own 10k pull-ups, pin 4 to ground, pin 5 to Flipper pin 2, pin 6 to Flipper pin 5, and pin 8 to 3.3v. I connected attached the flash chip to the test clip, hit read flash on the Flipper, selected my chip... and it started reading!

After quite the long process (reading serial flash is not quick!), I finally had the contents of the BIOS chip, which I could now patch. 

## Patching the BIOS
Okay, so now I have the BIOS. What next?

Helpfully, others have already found the sections of the BIOS that need to be changed to remove the whitelist. Using UEFITool, I opened my dumped firmware file, selected the GUIDs that need to be patched (81334616-86CE-49C2-B6F9-1804E61C73F6 and CC71B046-CF07-4DAE-AEAD-7046845BCD8A), and replaced them with patched modules from the reddit post. Next, I saved the (modified) BIOS file and transferred it back to my Flipper. Patching the BIOS itself was actually the easiest part of this whole process!

## Writing the BIOS
Okay, everything has gone smooth so far! Now we just have to write the firmware back onto the BIOS chip, resolder it, and off we go.

I opened the SPI Mem Manager app again, selected the modded BIOS file, and hit write. After an even longer process, I disconnected the chip from the test clip, resoldered it to the Thinkpad motherboard, connected the batteries, tried to boot, and...

Nothing. When people talk about the terrible feeling of dread and the onosecond, that's what I was feeling. But what could have gone wrong? Was it the flashing? Was it the modification? Maybe the chip was fried at some point in the process?

I'm actually not sure what went wrong. It could have been any of the above possibilities, or many other things, but thankfully, I had a solution. After longer than I would like to admit trying everything I could think of (including reading the flash again and checking if it was the same as the one I tried to flash), I decided to just use another flash chip and see if that worked. I extracted a spare flash chip from a broken board I had laying around, flashed the modded BIOS to this new chip, resoldered it, tried to boot again, and...

Woohoo! It worked! Thank goodness! At least I could boot the laptop again, and it wasn't bricked. I set my BIOS settings again (as they were all reset during this process), and booted into Windows. Sure enough, my brightness controls worked again! The fix worked, and it worked well -- all your display are belong to us, Lenovo. 

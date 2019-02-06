#simple-kprobe

##Simple Kprobe Implementation For An Absolute Beginner 

Sourced from: https://opensourceforu.com/2011/04/kernel-debugging-using-kprobe-and-jprobe/

This is a simple a guide to help compile the kprobe and insert it. Nothing more. I decided to do this because I ran into a lot of trouble getting even this simple kprobe to work. While the culprit was something kind of anticlimatic, I'm writing this anyway because it might help someone. 

I did all of this on a system running Linux Mint with the 4.20.2 kernel. 

First, I built the kernel from source by doing the following:

1. Update the package manager: sudo apt-get update

2. Install the following packages: sudo apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc

3. Go to kernel.org and download the latest kernel. Extract the .tar.xz and change to the extracted folder.

4. Once inside the folder, you can either go ahead a type: make menuconfig 
and try to fumble around with the huge and slightly scary list of things you can customize (read break) your kernel. At this point, you can just save and exit if you don't know what you're doing. This command will also throw an odd error if the size of your terminal window is too small (not sure whether it will do that on Linux Mint, this only happened when I tried to run this command on a small-ish terminal on Manjaro KDE in a VM.)

   Your other option is to copy the current kernel's configuration by: cp /boot/ config-$(uname -r) .config and then running make menuconfig. This is far easier.

5. Run: make modules_install

6. Run: make install

7. Run: update-initramfs -c -k 4.7.1 so that the new kernel is used the next time you boot up.

8. Run: update-grub so that grub adds the new kernel in the /boot folder to the boot loader entries.

9. You can now run: sudo reboot and can verify that the kernel version is indeed the one you built by running: uname -r

Note: Many of these commands may not work without sudo. So if any of them throw errors and look wonky, just try running it again with sudo. 
This process will take a good bit of time, so sit tight. 

Next, I compiled and inserted the kprobe by doing the following:

1. Change the address for the ip_rcv() on line 24 into the address on my computer: sudo cat /proc/kallsyms | grep ip_rcv 
Copy the address of the ip_rcv one. I recommend rechecking the value of the address every now and then and copying the correct one to your kprobe code (maybe between reboots?) because otherwise it won't work. 

2. Now here is the hairy part: you need to compile the module through the Makefile provided. The Makefile in this repo should mostly work. But the Makefile I started from gave me a lot of problems because it was very sensitive to spaces, tabs and other fun characters. My problem was that I had not surrounded the value for KDIR on line 5 in single quotes. I do not know why this worked but it did and I found out about it after a little snooping on Stack Exchange and GitHub. So, my advice is: if this Makefile doesn't work, try looking for another (recent) kprobe repo on GitHub and see if the Makefile there works for you. By works I mean, just see if the syntax, spaces and tabs match and what not. 

   This Makefile is used to generate the object and kernel module object file from the .c file. The objects generated by the Makefile should obviously have a file name  that matches the .c file name, so if you've named it something else, change it inside the Makefile too.

   After you've checked all this, run: sudo make
3. Once you've gotten the Makefile to work, it is mostly smooth sailing from here. The output of the command should look something like this:

thenuga98@Thenuga:~/myModule$ sudo make  
make -C '/lib/modules/4.20.2/build' SUBDIRS=/home/thenuga98/myModule modules  
make[1]: Entering directory '/home/thenuga98/Downloads/linux-4.20.2'  
  CC [M]  /home/thenuga98/myModule/mod1.o  
  Building modules, stage 2.  
  MODPOST 1 modules  
  CC      /home/thenuga98/myModule/mod1.mod.o  
  LD [M]  /home/thenuga98/myModule/mod1.ko  
make[1]: Leaving directory '/home/thenuga98/Downloads/linux-4.20.2'  

  If it doesn't look like this but the command executed without errors, you will find that the object and kernel module objects have not been generated. As mentioned    before, the Makefile needs to generate the required files. Go back to step 2 and tweak your Makefile to compile and generate only the required files.

  After this is done, the folder you've stored your Makefile and kprobe code in should have all the object and kernel module objects in it. 

4. Insert the module into the kernel by: sudo insmod filename.ko

5. Check that it has been inserted by: lsmod | head -n 5

6. Great, so far, so good. Now at this point, if the address of your system call ip_rcv() has changed for some reason, the next step won't work. So recheck your code and see that the address obtained in step 1 matches the one in the code. If it doesn't, change it and compile the kprobe again. Once you get the Makefile to work, there is nothing to fear. You should be able to compile it as many times as needed.

7. Now since ip_rcv() is the target function, run: ping localhost, stop it after a while and run: dmesg to see the output of the kprobe.

Phew, that's done! Now the kprobe can be modified to anything anywhere! 

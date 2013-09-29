<h1>wheezy-kde-live</h1>

Repository containing the configuration of my <a href="http://live.debian.net/">debian-live</a> 'cd' with KDE from Debian Wheezy.
My configuration differs from the standard in a number of ways:
* It has <a href="http://backports.debian.org/">Debian Backports</a> enabled and a number of packages are installed from that repository, most notably Libreoffice 4.
* It has my preferred package selection.
* It has the persistence kernel parameter enabled by default (more on that later).

<h2>How to use it?</h2>
The best source is the excellent documentation of Debian Live itself at http://live.debian.net/manual/stable/ .

For the impatient on a Debian system you need the "live-build" package installed and superuser access.<br>
Once you have that, clone my repository and cd into that. Then do:
<pre># lb build</pre> and wait for it to finish, which can take a while depending on the speed of your system and your internet connection. When it has finished, you'll have a file named "binary.hybrid.iso" which you can burn onto a cd/dvd or put it on an usb stick with the following command:
<pre>dd if=binary.hybrid.iso of={USBSTICK} bs=1M</pre>
Where {USBSTICK} is the identifyer of your stick, like /dev/sdb. When you plug in your usb stick and do "dmesg | tail", you should see which identifyer is assigned to your stick.
<p><b>WARNING</b> The above command overrides all the data that was previously present on that disk, so backup those up before you issue that command.</p>
If you've build your image and want to change the configuration, for example add your own preferred packages, do an <pre># lb clean</pre> first, change your configuration and then do a "lb build" again.

<h2>Persistence</h2>
Normally, when you run a live system the changes you make are gone when you reboot. But you can use a feature called persistence and by using that you can save the changes you've made and use that the next time you run your live system. So, in a way, you then have a system like your normal pc that you can take with you on the go.
There are a number of things you need to do to make that work though. The official documentation of persistence can be found <a href="http://live.debian.net/manual/stable/html/live-manual.en.html#529">here</a>, but I'll describe the needed steps below.

<p>Since you'll need a device to write to, this only applies to usb sticks.
<ol>
<li>Write the iso to the usb stick as described earlier.</li>
<li>Create a new partition on that stick which will be the place where the persistent data is written to. You can do this with any partitioning tool you like. This is how to do it with "parted".
<pre>parted {USBSTICK}</pre> where {USBSTICK} is again something like /dev/sdb. 
Parted's 'print' command gives you information on your stick, like the total size it has and any partitions on it, including their size(s):
<pre># parted /dev/sdb
GNU Parted 2.3
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print                                                            
Model: Generic Flash Disk (scsi)
Disk /dev/sdf: 1992MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type     File system  Flags
 1      32.8kB  1009MB  1009MB  primary               boot, hidden

(parted) </pre>
Like you can see on the output above, my stick is 1992MB in size. The first partition is from writing the iso file in step 1 and it ends at 1009MB. We'll use the rest of the stick to create the persistence partition, after which we can quit parted:
<pre>
(parted) mkpart primary 1010M 1992M
(parted) quit
</pre>
<li>Now that we have a partition, we need to create a filesystem on it and name (label) it 'persistence' like this:
<pre># mkfs.ext4 -L persistence /dev/sdb2</pre>
<li>The last step is creating a 'persistence.conf' file in the root of the new partition (/dev/sdb2) and specify which part of the system you'd like to persist. To persist the data of (only) the /home partition, write the following line to 'persistence.conf':
<pre>/home union</pre>
If however you'd like full persistence, ie of the whole system do this:
<pre>/ union</pre>
Note that the contents of 'persistence.conf' isn't limited to a single line, so the following example also works:
<pre>
/etc union
/home union
/usr union
/var union
</pre>
Which would persist the contents of the /etc, /home, /usr and /var directories.
</ol>

Enjoy!

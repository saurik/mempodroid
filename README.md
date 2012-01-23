Today on Hacker News (where I sadly get much of my news), the post ["Linux Local Privilege Escalation via SUID /proc/pid/mem Write"] [1] hit the front page. [This article] [2] was by Jason A. Donenfeld (zx2c4), and documented how he managed to exploit CVE-2012-0056, a seemingly silly mistake that was [recently found in the Linux kernel by Jüri Aedla] [3].

  [1]: http://news.ycombinator.com/item?id=3498835
  [2]: http://blog.zx2c4.com/749
  [3]: http://git.kernel.org/?p=linux/kernel/git/torvalds/linux-2.6.git;a=commitdiff;h=e268337dfe26dfc7efd422a804dbb27977a3cccc

Obviously, I was intrigued, and then spent the next few hours learning exactly how it works and putting together an implementation of the exploit for Android. It requires the device to have Linux kernel 2.6.39 or above, which happens to include the Galaxy Nexus (one of the various phones I luckily have sitting around for testing software on ;P).

Of course, the Galaxy Nexus can be rooted quite easily (with a big thank you to Google for being awesome!), so this isn't terribly important or useful: Android 3.1 runs 2.6.36 (too early to be exploited) and there aren't any devices other than the Galaxy Nexus running Android 4.0 (other, of course, than already-rooted ones using custom installs ;P).

That said, _I_ found it interesting, and I seriously burst out laughing when I read the article by Jason A. Donenfeld, as I found this particular exploit to simply be "that awesome". There is also always the possibility that there might actually be a device out there where this ends up being useful, so I figured I'd throw it up on GitHub.

Some Details
------------

So, a major requirement for this exploit to work is to find a setuid program that writes something deterministic to a file descriptor, and it turns out that the _only_ setuid program available on stock Android (run-as) just so happens to have exactly that behavior: you give it the name of a package, and if it doesn't exist it echos it to stderr.

Unfortunately, run-as is statically linked, so you can't do any simple tricks to find the exit() symbol in the program. I therefore looked it up with a disassembler. I could probably write some kind of auto-detector for the required offsets if people find this to actually be of use (i.e., it is important or usable on some device).

Further, run-as bails early on if it is not either a) already running as root (which would defeat the purpose) or b) not running as the adb shell user, so this unfortunately cannot be integrated into a "one-click root" app. You therefore already need working adb shell access to the device in order to install/run this program and escalate to root.

Usage Instructions
------------------

Once compiled (or [downloaded pre-compiled] [4]), you should copy it to your device (using adb), mark it executable, and then run it, passing first the offset of exit(), secondly the offset of the call to setresuid() (which must take its arguments from r5, I could easily generalize this if required), and a program to spawn as root.

  [4]: http://cache.saurik.com/android/armeabi/mempodroid

On the Galaxy Nexus, exit() is at 0xd7f4 and the call to sysresuid() is at 0xad4b (these happen to be printed by mempodroid as part of its usage instructions, if you forget). So, if you thereby wanted to then use it to remount /system with read/write access, you could use it as follows (or, just use "sh" to get an immediate shell).

    $ ./mempodroid 0xd7f4 0xad4b mount -o remount,rw '' /system
    $ ./mempodroid 0xd7f4 0xad4b sh
    # 

More Resources
--------------

If people _do_ find this useful (need offsets for other devices or help with something else related to it), feel free to join irc.saurik.com/#android (or I guess one of the various Android channels on Freenode that I happen to idle in). Please, though, for the love of all that is good in this world: keep questions there fairly on-topic ;P.

Also, if you join irc.saurik.com, I will ask that you state your question with complete detail and then stay logged into the channel (as I might not actually be there for hours). If you just ask "u there" (or only wait a few mintues after asking your question, instead of some reasonably larger number of hours) you lilkely won't get a response :(.

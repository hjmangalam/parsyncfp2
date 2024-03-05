# parsyncfp2
a MultiHost parallel rsync wrapper writ in Perl. 
by Harry Mangalam <hjmangalam@gmail.com>
Released under GPL v3.

(Changes moved to the bottom of this file)

## Background

NB: If you don't want to transfer at least 10s of GB across a network, this is probably not the the tool you want.  Use rsync alone if you need or will need a sync operation, or scp if the data needs to be encrypted.


parsyncfp2 (aka pfp2) is the next generation of the family that started with [parsync](https://github.com/hjmangalam/parsync), which with Ganael LaPlanche's [fpart](http://goo.gl/K1WwtD), begat [parsyncfp](https://github.com/hjmangalam/parsyncfp) (aka pfp), which has further mutated into the MultiHost multi-send, multi-receive organism unimaginatively called parsyncfp2.

Like parsyncfp, which uses fpart to aggregate files into chunks (or partitions) to allocate to individual rsyncs, pfp2 operates similarly. The main difference between them is that pfp2 can spread the send and receive functions among multiple hosts (with a shared filesystem required on the sending side.)
As with pfp, it collects files based on aggregate size into chunkfiles which can be fed to rsync on a chunk by chunk basis.  This allows pfp to begin transferring files before the 
complete recursive descent of the source dir is complete.  This feature can save many hours of prep time on very large dir trees.  In addition, pfp2 can re-use the chunkfiles so generated so if there's an interruption, you can skip the re-generation of the chunkfile list (which is pretty fast, but for a PB filesystem can still take a long time and generate a lot of competing IO)

NB: recently fpart changed from starting its chunk files from 0 to 1, and this version of pfp2 is the first github release that tracks that change. Using fpart 1.5.1 works fine, as do the last couple of releases.

If your use involves transit over IB networks, parsyncfp requires 'perfquery' and 'ibstat', Infiniband utilities written by Hal Rosenstock < hal.rosenstock [at] gmail.com > 

pfp2 is tested on Linux.  The MacOSX port is in hibernation.

pfp2 needs to be installed only on the SOURCE end of the transfer and only works in local SOURCE -> remote TARGET mode (it won't allow remote local SOURCE <- remote TARGET, emitting an error and exiting if attempted). It requires that ssh shared keys be set up prior to operation [see here](https://goo.gl/ghCazV).  If it detects that ssh keys are NOT set up correctly, it will ask for permission to try to remedy that situation.  Check your local and remote ssh keys to make sure that it has done so correctly.  Typically, they're in your ~/.ssh dir.

It uses whatever rsync is available on the TARGET.  It uses a number of Linux-specific utilities so if you're transferring between Linux and a FreeBSD host, install pfp2 on the Linux side. 



## Installation
Installation of 'parsyncfp2' is fairly simple.  There's not yet a deb or rpm package, but the bits to make it work  that are not part of a fairly standard Linux distro are the Perl scripts *parsyncfp2*, *scut* (like cut but a bit more flexible), and *stats* (spits out descriptive statistics of whatever is fed to it).  
The rest of the dependents are listed here:

- **Debian/Ubuntu-like:**

    sudo apt install ethtool iproute2 fpart iw libstatistics-descriptive-perl infiniband-diags

    git clone https://github.com/hjmangalam/parsyncfp2 

    cd parsyncfp2; cp parsyncfp2 scut stats ~/bin


- **RHel/Centos/Rocky-like:**

    sudo yum install iw fpart ethtool iproute perl-Env.noarch  \
    perl-Statistics-Descriptive wireless-tools infiniband-diags

    git clone https://github.com/hjmangalam/parsyncfp2

    cd parsyncfp2; cp parsyncfp2 scut stats ~/bin


### Required utilities and packages
Should the above commands not fulfill the requirements or be missing from your set of repositories, the utilities are listed below.

- ethtool - query or control network driver and hardware settings. Install via repository.
- ip - show / manipulate routing, network devices, interfaces and tunnels. Install via repository.
- fpart - Sort and pack files into partitions. Now in many distro repositories, or install from [the fpart github](https://github.com/martymac/fpart); 
- scut - a more intelligent cut.  Included in the parsyncfp2 github
- stats - calculate descriptive stats from STDIN. Included in the parsyncfp2 github
- Perl::Descriptive-Statistics - basic descriptive statistical functions, but pfp will work without it.


### Recommended Utilities

- iwconfig - configure a wireless network interface. Needed only for WiFi. Install via repository.
- perfquery - query InfiniBand port counters.  Needed only for InfiniBand. Install via repository.
- udr (experimental) - utility to send packets via UDP using the UDT library.  Required for the --udr option. Install via [github](https://github.com/martinetd/UDR)



## Changes

### stats
- no change to pfp2, but added sample size estimation to stats.

### 2.59
- found and corrected the lagging print routines after launching the last rsyncs. via ..
- rearranging the data print routine as a sub, tighten up the timing and calculations; 
  moved some vars to our(vars); points to better ways of rearranging a lot of vars for this 
  and other libs.
- added a few more fpart/rsync overrun/collision detection stuff
- increase the number of BW checks before the WARN to prevent spamming the screen on startup, 
  when there will usually be a delay before fpart produces enough usable chunks.


### 2.571
- fixed the zombie --ro problem (again) that refuses to die 
- added surprising stats from GPFS to GPFS rsyncs. (~6x speedup over single rsync iwth --NP=12)

### 2.57
- verified that pfp2 now/again works across mounted filesystems, tho (probably) much slower than
across networks. Still, it increases speed of rsyncs across (parallel) FSs significantly. 
- added some more debug lines to estimate location of non-pfp2 error messages
- fixed the zombie --ro problem (again) that refuses to die 
- added a printout of actual rsync commands under VERBOSE=3 for additional debugging.  Maybe limit them to only 5 and then stop?  Not yet..

### 2.56
- minor changes, add usability.
- in Multihost mode, added code for allowing t/csh shells (as well as previous sh-like shells) on SEND hosts (so setting remote RPATHS should work)
- some help edits to clarify how things work.
- more fixes for the rsync options.  Now DON'T need double quoting - pfp2 now takes care of that internally.   And a quick followup to fix that fix (if didn't supply an --ro, would die.)
- fixed erroneous 'fix' of # of rsyncs going (should almost always be at NP)

### 2.55
- some major fixes, but not major behavioral changes.
- figured out why rsync options (--ro) were failing sometimes - they need to be '"double quoted"' (and then re-double quoted) to make it thru getopt & sending to the SEND hosts
- thanks to GabeT for the bug report and fix for creating the partial pfp2 command that goes to SEND hosts. 
- And alerting me to some other issues with that process. Prob more to come.
- fixed the case where SEND hosts (and remote servers) have users other than the originating account
- more interface cleanups, added a sub clearline to blank overwritten info lines.
- removed a bunch of zombie vars.


### 2.51
- major changes in this release.
- this is the first version that cooperates with the new version of fpart that starts numbering chunks
at 1 rather than 0.  So the current version of fpart 1.5.1 should work fine. This change in numbering allows special handling of files larger than chunk size, now in process. If you dig in the code, you'll see special handing for zillions of tiny files as well, also in progress, but also not well-debugged.  Stay away from those options.
- the scrolling output now is maintained until all rsyncs spawned by the SEND hosts have ended.  Before, pfp2 ended when all of the rsyncs were started.
- fixed a major, if largely invisible bug in the way fpart was launched.
- slowly reducing functionally duplicated variables 
- reduced # of tests that access filesystem via some primitive logic.
- the checkhost validation file (now "$HOME/.pfp-hostchecked") has been moved outside the .pfp dir tree so it survives the default deletion and renewal of ~/.pfp2
- there are some files that pfp2 is missing: a file named '2.1.1'$'\n' including  the enclosing 's and the literal '$'\n' was skipped.
- have started to doc internal functional chunks with the prefix ##: If you actuall want to get a sense of where the bits are that might address a bug you found, try "grep -n '##:' parsyncfp2" 
- trying to find the balance between helpful and annoying verbosity by merging, removing frequent or large text emissions.
- many, many bugs fixed. Many, many remain, but seem to be lower level.


### 2.44 
- added some regex to prevent the kill script from killing the remote rsync daemon in case you're using it as root. Very much NOT recommended, but some ppl require this, apparently.
- fixed the bug that caused 2 dirs for each send host to be created if you specified them with something other than the correct short host name ('hostname -s'). ie: if you spec'ed a send host as '128.200.43.11' it would create a dir named that, and later, the 'hostname -s' of the same machine.  Cost ~.5s per host, but fixes it simply.

### 2.43
- removed the option of calling it '--rsyncopts', now just --ro to prevent some regex filtering problems in the past.

### 2.42
- add --skipto  - esp useful with --reuse to skip to that chunk without wading thru all the intervening fpart chunks and rsyncs.  Could save minutes to 10s of min in restarting a huge transfer, especially if the network has significant delay and the --slowdown delay is significant. 
so --reuse and skipto are both required and just set the CUR_FPI to that number.

- collapse --reuse and --skipto into --reusechunk:i  makes more sense.

### 2.41
- debugged weird ssh/RTT errors, introduced --slowdown and internal routine to ping to REC hosts to figure out how slow the connection was and compensate with some delays in starting new ssh connections.  Seems to have addressed the problem.

### 2.40 
- Check generation of MHplotfile.sh - it doesn't pick up the user@host  designation and munges the 2 together so it drops out any data that has a user@host send or rec bc it's not parsing the data correctly (I think).
- forbid MH rsyncs on a mounted FS for now

### 2.39
- debugged the weird tailoff in bandwidth.  It's the remote firewall interpreting numerous ssh's as an attack and blacklisting me.  OK - off to iter we go..

### 2.38
- Corrected some bad logic in the main rsync startup loop.  subtracting the stride instead of adding it, and some other stuff.  Ugly.  The failing rsyncs were mostly due to a bad creation of the chunk file paths that in some cases prefixed the 'home/pfp' argument to the file path, so you ended up with /home/pfp/home/pfp/..(fqfn). Also corrected the hanging rsync error - the check is picking up 'parsync' and the --rsyncopts option. Plain stupid.

### 2.37

- test if --hosts, has to be a POD:: as target (!)

- at exit, should re-emit the complete calling commandline as a reminder.

- if specify chunk size larger than a single chunk, no erros emitted and it just runs ad infinitum.
      so needs to check if max number of chunks when FPART_DONE < number of chunks, emit WARNING.
      
- get_nbr_chunk_files() is really expensive in terms of FS activity.  Should look at this to reduce the number of calls. OK - removed 2.

### 2.3
- renamed to parsyncfp2 from pfppod and cleared deps for the name change.

### 2.00 - 2.39
- so many changes, lost track.  Mostly small thinkos but in aggregate a lot of cleanup.  thanks to ITER for hosting a lot of network testing.


### 2.00 (as pfppod)
(Multihost ITERation) Apr 22, 2021. Lots of changes...

- added multihost capability.  Can send via multiple SEND hosts to multiple REC hosts, 
with multiple REC paths if wanted.
    - with --hostcheck, multihost version pushes all required scripts to targets for better 
    reliability, checks rsync status on remote hosts.
    - stat remote ~/.ssh/config file to warn about conflicting ssh-defined hostnames
    - added --rpath to set remote PATHs (tho may deprecate)
    - the multiple hosts store their logs in /hostname/ subdirs in the cache dir so they're 
    more easily separable
- removed prohibition of using '~' to denote remote HOME targets.  I think this has no more use.
- incorporated Chris Rapier's code to remove the long ls output, leading to crashes with processing 
zillions of fpart chunk files (but left the warnings about too many of them).
- considerable code cleanup, tho you'd be hard-pressed to tell.
- mod debug() to be more useful.
- added --udr to support UDP-transport supposedly good for increasing large file transport.
works on local nets, but not much better than normal.  Need much longer rtt networks to test on.
udr is not in repositories, so users have to add it for now.
- multihost version still works as single host version
- side effect of multihost version: SEND hosts are completely independent; need to add socket 
control to suspend/kill/cleanup/pass info


### 1.72 
(California Lockdown) Dec 6, 2020, No option changes.  Intercepted rsync options to forbid 
those that increase verbosity to avoid collision with pfp's IO handling. 
Including: -v/-verbose, --version, -h/--help 

### 1.71
(GoVote) Nov 2, 2020.  No option changes.  Changed how checking for external utilities 
   works.  Separates the required from recommended utilities and now continues with 
   a WARN if it doesn't find the recommended utils.

### 1.70 
(Silverado Fire), Oct 27, 2020.  No option changes.  Fixed bug about setting up the 
   fpart command (not a problem with fpart, just coercing names with spaces to be represented 
   correctly).

### 1.69 
(Covid Synchronicity), Aug 17, 2020. No option changes, but included a significant change in the way 
    that pfp reads the chunk files that fpart provides.  Before this versio, pfp 
    checked only for the existence of the chunk files and
    could therefore launch an rsync instance on a filelist that had not been completed.  If rsync overran the files,
    This might happen when a fileset that had already been mostly transferred, 
    and so it could theoretically exit before 
    fpart finished the writing to the file, leaving some files unsync'ed.
    
pfp now uses fpart's '-W' option to run a post-file-close script to move the finished 
    and closed file to the processing directory,
    assuring that the chunk files are not read before fpart is finished writing
    to them.  Thanks again to Ganael Laplanche (fpart author) for discussion and 
    suggesting the simplest way to address the problem.
    
Also some better checking for nonsense or non-existent files/dirs.

### 1.65
- tidied, changed a lot of output routines to consolidate subs and tested on HPC on some TB sized sets
   to verify clean ending.  Also tested mostly complete rsyncs to verify the fpart keep-ahead routines.
   Seems to be good.
   
### 1.64
- fixed bug that allowed for infinite waiting for fpart to produce another chunkfile.

### 1.63
- added check for underflow in fpart chunks; warning emitted if # of chunks is < # of rsyncs requested
- updated / replaced all 'route' and 'ifconfig' bits with 'ip' equivalents.  Also re-did the multihoming 
    bits to probe, and present all routed interfaces and IP #s so user can choose the right one on a multihomed
    host. Thanks to Ryan Novosielski for suggesting / pushing on this ;).
- usual bits of code cleanup and messup in equal amounts, tho overall things should progress more smoothly.
- detected a weird output format bug(?) inconsistency in CentOS 6.9 'ps' vs recent 'ps'.  Not going to fix.
  reveals itself if exceed MAXLOAD on CentOS 6.9 - the load balancing continues to work OK afaict but the suspended PIDs aren't shown correctly.  The workaround is to set MAXLOAD high enough that it won't be invoked.
- added more utilities to check for, before startup and corrected links to them.

### 1.61
- added total raw bytes transferred as well as autoscale to MB, GB, TB in signoff stanza.

### 1.60
- figured out why the suspend/unsuspends noted below weren't tracking correctly.  Thanks to Ben Dullnig 
for his perseverance and Mikael Barfred for his code suggestions. Lots of cosmetic and interface tweaks.  
Added Bill Abbott's suggestion to allow explicit sizing to the --fromlist option. Added a warning if the
number of chunkfiles exceed 2000 (hard-coded).  Also caught a rare condition where the run ends with suspended jobs. 
Now it should continue until the suspended PIDs get unsuspended and complete.

### 1.58
- added '--fromlist' to allow explicit lists of files to be pfp'ed.  Suggested by 
Bill Abbott to support GPFS's mmapplypolicy to generate lists so that pfp can immediately 
start moving data instead of iterating thru miles of already-synced files. Thanks, Bill.

### IMPORTANT NOTE (May 31, 2019)
Thanks to the long-suffering efforts of Jeff Dullnig, I've discovered that when parsyncfp goes thru 
multiple suspend/unsuspend cycles, it fails to correctly rsync all the src files to the target.

If the '--maxload' option is kept high enough to avoid any suspensions, it syncs correctly. 

If you're using parsyncfp now, please be aware that if forked rsyncs cycle thru suspend / unsuspends
you will probably not end up with a correct target.  I'll be working on this to determine if it can 
be fixed or if that 'feature' has to be removed.

### 1.57 
- added explicit GPL v3 licence 

### 1.56
- added a better measurement of TCP bytes sent (via /proc/net/dev) 
- added attempt at measuring RDMA bytes sent by using 'perfquery'; looks like it's
doing what it's supposed to.

### 1.54
- Bungled a commit.  This one should straighten is out.

### 1.53 
- removed internal space handling for target names since this interferes with multiple
dir targets.  Have to re-think this.

### 1.50
- fixed Ken Bass' bug where trimming dir name was not constrained to front
   of the string and could lead to problems if dir was names something like
   '/data/rna/hjm/rnaseq/data/something/version/data/smith'
   if the leading name was '/data/' the condensed, trimmed dir was 
   changed to
   'rna/hjm/rnaseq/something/version/smith' ie removal of ALL 'data/'
- fixed finding top level targets with embedded spaces - had to trim spaces
   and escape filenames going into fpart.
- many verbosity fixes
- some changes to ending text to reference both rsync and fpart logs 


### 1.47
- changed format of output to add elapsed time, changed date format
- code cleanup
- fixed bandwidth speed calculation
- updated help and fixed some inaccuracies for latest version.
- added (declinable) scripted mod to ~/.ssh/config to reduce ssh warnings

### 1.46
- made it variably verbose (--verbose)
- adjusted ending conditions to be accurate
- some code cleanup.  getting there.
- added checks for multihomed devices.





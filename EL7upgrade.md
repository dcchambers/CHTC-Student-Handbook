**EL7 Upgrade Guide**

*NOTE: As of 10/4/17 we have a new Centos 7.4 Profile in Cobbler. Use this profile when rebuilding/building execute machines!*


1. Fix any hardware issues with the server, such as a bad disk or RAID configuration (only on certain machines).

2. Changes in the puppet_data repo

  * In the puppet_data repo, do a `$ git pull` to pull down any changes in the git repo upstream.
  * Create the proper .yaml file in the node directory.
    * Easiest and best way is to make a copy of a similar machine from the same datacenter, and then replace the IP and MAC address with the correct one for the node you are working on.
    * Eg., in room 3370a, if you are upgrading server e160, do `$ cp node/e151.chtc.wisc.edu.yaml node/e160.chtc.wisc.edu.yaml` and then replace the IP address and MAC address in the node/e160.chtc.wisc.edu.yaml file with the correct ones from e160.
      * You can get these from cobbler (On wid-service-1 run `$ sudo cobbler report --name=[server name]`) or from the DHCP server.
  * Create symlinks in the role_tier_0 and site_tier_0 directories.
    * `$ ln -svf ../role/htc_execute_node.yaml role_tier_0/[node].yaml`
    * `$ ln -svf ../site/[site].yaml site_tier_0/[node].yaml`
      * A site could be cs_2360.yaml, cs_3370a.yaml, cs_b240.yaml, or wid.yaml.
    * Confirm that you made the symlinks correctly by looking at them with `$ ls -la`
  * You now need to add these three files to git, make a commit, and then push them to the master repo.
    * Make sure you have a .gitconfig file with your name and email set up.
    * `$ git add [file1] [file2] [file3] ...`
      * You can run `$ git status` to see which files have been changed but are not yet staged for commit.
    * `$ git commit -m '[commit message]'` where [commit message] is a concise explaination of the changes you made. For example, "Added .yaml files to upgrade e151 to EL7"  
     * `$ git push` If you are already on the master branch (check with `$ git status`) you can just push it. <br><br>

3. Set up the disk configuration.
  * SSH into wid-service-1.
    * Change into the cobbler snippets directory `$ cd /var/lib/cobbler/snippets`
    * Edit or create a file disk_config file for the node you are upgrading.
      * It may already be created, it may not be, check for it with `$ ls`
      * Figure out how many disks the machine has you are updating. If it has one disk, you shouldn't need to make any changes. If there are 2 disks, they need be set in raid0 (for execute nodes), and 3 disks should be configured in raid5.
      * What you are really going to do is symlink the disk\_config\_[node] file to the correct configuration file.
        * If it has two disks, `$ ln -svf disk_config_exec-2disk [node disk_config file]`
        * If it has three disks, `$ ln -svf ../role/execute/disk_config_exec-3disk [node disk_config file]`
    * You will need to make these changes with `sudo` <br><br>

4. Update the cobbler system object for the node.
  * You will need to enable netboot, set the profile to be CentOS_7_4_exec, and set the hostname.
  * SSH into wid-service-1.
    * `$ sudo cobbler system report --name=[node]` to view the current cobbler configuration for this server.
    * `$ sudo cobbler system edit --name=[node] --netboot-enabled=true --profile=CentOS_7_4_exec --hostname=[hostname]`
    * Check that your changes were made correctly by running another report.
    * `$ sudo cobbler sync` when you are done.
  * If you are upgrading a batch of nodes, you can run this small script:
    > for i in [Space delimited node list]; do <br>
    > sudo cobbler system edit --name=$i <br>
    > --profile=CentOS_7_4_exec \ <br>
    > --kopts='' \ <br>
    > --ksmeta='' \ <br>
    > --netboot-enabled=true <br>
    > done

  * Make sure to sync when done.<br><br>

5. Update the HTCondor Configuration for the node. Do not do this until you are really ready to upgrade as it will cause HTCondor to stop working on the target server.
  * If you don't have it already, `$ git clone` the chtc_htcondor_config repo or cd into the chtc_htcondor_config directory. If you already have the repo cloned, do a `$ git pull` to update your branch.
  * In the hosts directory, open up the file of the server you are working on.
    * Add the line `include : $(INCLUDE_CONF)/opsys_el7.conf` to the end of that file.
  * Add the file to git (`$ git add [file]`), you can check with `$ git status` to see if it has been added (red means it has been changed but not yet staged for commit. Green means you have added it correctly.)
  * `$ git commit -m [commit message]`
  * `$ git push` if you're on the master branch already. <br><br>

6. Rebuild the server.
  * If this is a broken server you are upgrading, go ahead and reboot it and netboot to run the upgrade process.
  * If this is a working server you are upgrading, you can use the kexec_boot.sh script found in the infscripts git repo (in the util directory).
    * Get this file onto your local machine and then scp/sftp it onto the target machine. Run the script, and it should do everything a PXE boot does without the need for a reboot.
    * At this point, you should be able to ssh into the host as root and do a `$ tmux attach-session -d` to attach to the anaconda installer tmux session. CTL-b + 2 will put you into the shell window. You can chroot into the install directory with "chroot /mnt/sysimage /bin/bash" if you want.
    * It should automatically run puppet (it does this late in the install period, however).
      * If you need to manually run puppet, the r10k_puppet.sh script will do an r10k-pull and a puppet run on the chosen environment (eg, master):
        * `/usr/local/bin/r10k_puppet.sh master`
      * Networking will need to be working in order to run puppet. If you did all the above steps correctly (especially setting up the role and site files/symlinks) then networking should be automatically configured.
      * If the git server is down or having issues, puppet won't run.

If you have any questions, refer to the [EL7 upgrade ticket. (#51784)]( https://crt.cs.wisc.edu/rt/Ticket/Display.html?id=51784)

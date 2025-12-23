+++
title = "Troubleshooting a small boot partition"
tags = [
    "linux"
]
date = "2025-12-23"
toc = true
+++

# This one will be quick

Recently stumbled across an interesting problem... 

Against recommendations I have multiple long-lived Kali Linux VMs. I use these for CTFs as well as general security research and my day job, so having them always available and set up the way I want is quite valuable to me. This, of course, means I also need to keep them updated.

As I update these VMs I of course get updated packages, but I also get updated kernels. While the compiled kernel, for Debian/Kali, is quite small with the total package coming in around 250MB, Kali still defaults the installations to a 500MB boot partition which of course means that it's a stretch to have more than a single kernel version installed for rollback purposes (or even completing single updates).

Since I screw around with these machines a lot, it's important that I also be able to recover from problematic updates, so having current-1 kernels available for rollback is super helpful and has saved my bacon more than once. Yes, I also back up the VMs and take snapshots where possible, however it's much easier to just run a few commands than to completely reset the state of the machine.

# The problem

So it turns out that when running an update via `apt` that will drop a new kernel version to /boot, it won't actually check that there's enough space for the output of `initramfs`, and as a result you can wind up with a successfully installed kernel but having 0 bytes free on `/boot`. Your bootloader will have been updated to point to the newer version, but if you attempt to boot to it you'll just get kernel panics.

# The fix

This has been [posted all over the internet](https://www.reddit.com/r/Ubuntu/comments/163pd22/guide_removing_old_unused_kernels/), but here it is again since it's useful dammit!

```
uname -r # Get your current (presumably working) version
dpkg -l | tail -n +6 | grep -E 'linux-image-[0-9]+' # List all installed kernels
sudo apt purge linux-image-VERSION_TO_REMOVE # Remove specific version(s)
sudo apt autoremove && sudo apt autoclean # Cleanup
```

For a more manual version of this, you can also do some surgery on `/boot`. Do this at your own peril - you can cause some serious problems. Replace `VERSION_GOES_HERE` with the actual version (e.g. `6.12.33+kali-amd64`):

```
ls -la /boot # See what all is in /boot
sudo rm -f /boot/initrd.img-VERSION_GOES_HERE
sudo rm -f /boot/vmlinuz-VERSION_GOES_HERE
sudo rm -f /boot/System.map-VERSION_GOES_HERE
sudo rm -f /boot/config-VERSION_GOES_HERE
sudo dpkg --configure -a
sudo apt purge linux-image-VERSION_GOES_HERE
sudo apt purge linux-headers-VERSION_GOES_HERE
sudo update-initramfs -u -k all
sudo update-grub
```

**Don't do this unless you know what those commands mean**. Do some research before bricking your system and complaining about this blog post :P

# And automation to help prevent the problem

Of course now that this is fixed I don't want it to happen again - specifically I want to only ever keep one single previous kernel version available for rollback rather than the (default?) unlimited number that will otherwise appear.

We can do this with an `apt` config file edit, however why do that manually across systems when we could use Ansible instead. So, [here's a playbook for it](https://github.com/maclarel/ansible_playbooks/blob/master/vm_manage/playbooks/kernel_upgrade_retention.yml):

```

---
- name: Enforce unattended-upgrades kernel retention policy
  hosts: all
  become: true
  gather_facts: false

  tasks:
    - name: Ensure unattended-upgrades is installed
      apt:
        name: unattended-upgrades
        state: present
        update_cache: true

    - name: Enforce unattended-upgrades cleanup policy
      copy:
        dest: /etc/apt/apt.conf.d/50unattended-upgrades
        owner: root
        group: root
        mode: '0644'
        content: |
          // Managed by Ansible
          Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
          Unattended-Upgrade::Remove-New-Unused-Dependencies "true";
          Unattended-Upgrade::Remove-Unused-Dependencies "true";
          Unattended-Upgrade::MinimalSteps "true";

    - name: Ensure unattended-upgrades service is enabled and running
      systemd:
        name: unattended-upgrades
        enabled: true
        state: started
```

Good luck troubleshooting if this happens to you!

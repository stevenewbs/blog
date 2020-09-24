# libvirtd / qemu Blu Ray drive passthrough

A quick and dirty guide to get your blu ray drive showing up in a VM (for example, using MakeMKV in a vm). 

## Pre-flight checks
* SATA Blu Ray drive - USB ones are easier to pass through, so I'm lead to believe from what I have seen while working out how to get this to work.
* Get libvirtd and qemu set up.
* Establish a method of managing your VMs (e.g, virt-manager, locally or remote).
* Install `lsscsi` if not already. (TLDR; `$ sudo apt install lsscsi`)


## Apparmor

Thanks to [this ask ubuntu post](https://askubuntu.com/questions/1034181/how-can-i-grant-kvm-read-access-to-dev-sg0) which got this fixed.

Apparmor may block access to your disk drive so you need to whitelist it. 
1. Check if Apparmour is blocking access:

   `dmesg | grep /dev/sg0` OR `dmesg | grep /dev/sr0`

   Check for "DENIED" entries for /dev/s{r,g}0 

2. Add lines to `/etc/apparmor.d/libvirt/TEMPLATE.qemu` to allow access to qemu:
   ```
   profile LIBVIRT_TEMPLATE flags=(attach_disconnected) {
     #include <abstractions/libvirt-qemu>
     /dev/sg* rwk,
     /dev/sr* rwk,
   }
   ```

3. Add the `/dev...` from above to `/etc/apparmor.d/usr.lib.libvirt.virt-aa-helper` under the # for hostdev section:
   ```
   ...
     # for hostdev
     /dev/sr* rwk,
     /dev/sg* rwk,
   ...
   ```

4. Reload apparmor:

   `sudo systemctl restart apparmor`


## Libvirt / qemu config

[This Unraid thread](https://forums.unraid.net/topic/33851-blu-ray-dvd-rom-passthrough/) gave some heavy hints for this section.

1. Create the VM as you wish (make a note of the name). 

2. List SCSI devices to get the ID of your Blu Ray drive:
   ```
   $ lsscsi
    [2:0:0:0]    cd/dvd  ATAPI    iHOS104          WL0F  /dev/sr0 
    [3:0:0:0]    disk    ATA      WDC WD30EFRX-68E 0A82  /dev/sdb 
    [4:0:0:0]    disk    ATA      KINGSTON SV300S3 BBF2  /dev/sdc 
    ...
   ```
   
2. On the host, edit the VM's config:
   `sudo virsh edit <VM NAME>`
   
3. Add a section for a SCSI controller and a new hostdev device. These need to be within the `<devices>` tags - the file will re-generate with your changes afterwards anyway. 

   A: A section similar to this may exist already. If not, add it. 
      ```
      <controller type='scsi' index='0' model='virtio-scsi'>
         <address type='pci' domain='0x0000' bus='0x00' slot='0x0c' function='0x0'/>
      </controller>
      ```
      
   B: Add a new `<hostdev>` section for the drive. 
   ```
   <hostdev mode='subsystem' type='scsi' managed='no'>
     <source>
       <adapter name='scsi_host2'/>
       <address bus='0' target='0' unit='0'/>
     </source>
     <readonly/>
     <address type='drive' controller='0' bus='0' target='0' unit='0'/>
   </hostdev>
   ```
   
   Where 'scsi_host2' has the number changed to whatever the first digit was from the `lsscsi` output earlier. 
   
   C: Save and exit the editor, hit `i` to try to ignore warnings about parsing the XML. 

4. Boot up the VM.


  

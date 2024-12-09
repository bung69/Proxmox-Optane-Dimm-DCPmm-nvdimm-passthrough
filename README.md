usefull links:

https://www.intel.com/content/www/us/en/developer/articles/training/provision-intel-optane-dc-persistent-memory-for-kvm-qemu-guests.html
https://docs.pmem.io/persistent-memory/getting-started-guide/creating-development-environments/virtualization/qemu
https://github.com/qemu/qemu/blob/master/docs/nvdimm.txt

ipmctl
ndctl

args: -machine nvdimm=on -m slots=2,maxmem=1T -object memory-backend-file,id=mem1,share,mem-path=/pmemfs0/pmem0,size=100G,align=2M -device nvdimm,memdev=mem1,id=nv1,label-size=2M



TLDR:
use impctl to configure optane dimms

use ndctl to configure an fdax namespace

format to xfs and mount with dax option

build qemu with nvdimm support


    git clone --recursive git://git.proxmox.com/git/pve-qemu.git
    cd pve-qemu/
    apt build-dep .
    nano /debian/rules



Backend File Setup Example
--------------------------

Here are two examples showing how to setup these persistent backends on
linux using the tool ndctl [3].

A. DAX device

Use the following command to set up /dev/dax0.0 so that the entirety of
namespace0.0 can be exposed as an emulated NVDIMM to the guest:

    ndctl create-namespace -f -e namespace0.0 -m devdax

The /dev/dax0.0 could be used directly in "mem-path" option.

B. DAX file

Individual files on a DAX host file system can be exposed as emulated
NVDIMMS.  First an fsdax block device is created, partitioned, and then
mounted with the "dax" mount option:

    ndctl create-namespace -f -e namespace0.0 -m fsdax
    (partition /dev/pmem0 with name pmem0p1)
    mount -o dax /dev/pmem0p1 /mnt
    (create or copy a disk image file with qemu-img(1), cp(1), or dd(1)
     in /mnt)

Then the new file in /mnt could be used in "mem-path" option.

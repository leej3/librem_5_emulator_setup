# More verbose instuctions for emulating the librem 5 OS (using debian operating system)

I had lots of trouble with the official [setup](https://developer.puri.sm/Librem5/Development_Environment/Boards/qemu.html#linux-environments). Perhaps a lot of it is obvious to others. I'm documenting it here for others and my future self. I've left out the various required software you will need as you work through it. Off the top of my head they include virt-manager and vmdebootstrap. You can install them using `apt install`.

## Summary
This is mean to complement the pre-existing instructions which are reasonably extensive. 

Figure out when there was a successful build last on their [CI](https://arm01.puri.sm/job/Images/job/Image%20Build/).
You are specifically looking for a build that created a qcow2 image. You can download the image at the link provided for their build artifacts. 
To run it a nice way is to use qemu as they suggest in their docs. 
I didn't find the boot option especially helpful. Enabling kvm is probably a good idea though. And specifying 3G of memory. And I decided to not give all my cpus to the phone.

Additionally it is nice to have ssh access, and to mount your local directory so that you can work on the same files inside and outside the emulated device.
This all results in the following command:

```
qemu-system-x86_64 --smp 2 ./qemu-x86_64.qcow2 --enable-kvm -vga virtio -m 3G -enable-kvm -device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp::5555-:22 -usb -virtfs local,path=$PWD,mount_tag=host0,security_model=passthrough,id=host0
```

Once you've run the above and logged in (pw 123456), you can ssh (from a new terminal console) to the phone with:

```
ssh -p 5555 purism@localhost
```

At this point to use the mounted volume you can add the following to /etc/fstab (you'll need to use sudo)

```
host0   /my_share    9p      trans=virtio,version=9p2000.L   0 0
```

Create your directory within the emulator:

```
sudo mkdir /my_share
```

And finally execute:

```
sudo mount -a
```

You now have a ssh connection to your emulated librem 5 with a shared directory


### Building the qemu image from source

Instead of downloading a 2.5Gb image each time you want to update the phone software version you can build the image from source. To do this, checkout a recent successful commit (assessed from the ci) in the [image builder repository](https://source.puri.sm/Librem5/image-builder).
For example here is a [successful one](https://arm01.puri.sm/job/Images/job/Image%20Build/4865/)
So run the following: 

```
git clone https://source.puri.sm/Librem5/image-builder
cd image-builder
git checkout 9a6319908fefd3bf98afd45a9502c001b6c3edaf
```

Run the image build using the make build tool, you need to know type, distro, and board
For the phone image (using bash):

```
BOARD=qemu-x86_64 TYPE=plain DIST=amber-phone make
```

It takes about 20 minutes to build the librem 5 image.




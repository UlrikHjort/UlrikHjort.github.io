## Run a Cortex-A53 (Raspberry Pi 3b+) in QEMU

### Get or build QEMU

In this examle qemu-system-aarch64 version 8.0.50 is used with the slirp network module,

If building QEMU yourself remember to configure slirp network module:

```
./configure --enable-slirp
```

### Prepare image

#### Download image

```
wget https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2023-05-03/2023-05-03-raspios-bullseye-arm64-lite.img.xz
```

#### Resize image:

```
qemu-img resize ./2023-05-03-raspios-bullseye-arm64-lite.img 8G
```

#### Mount kernel partition

Find start block of the kernel partition


```                                                                                                                                                                                                              
$ fdisk -l  2023-05-03-raspios-bullseye-arm64-lite.img

Disk 2023-05-03-raspios-bullseye-arm64-lite.img: 8 GiB, 8589934592 bytes, 16777216 sectors

Units: sectors of 1 * 512 = 512 bytes

Sector size (logical/physical): 512 bytes / 512 bytes

I/O size (minimum/optimal): 512 bytes / 512 bytes

Disklabel type: dos

Disk identifier: 0x544c6228                                                                                                     
                                                                                                                            


Device                                      Boot  Start     End Sectors  Size Id Type

2023-05-03-raspios-bullseye-arm64-lite.img1        8192  532479  524288  256M  c W95 FAT32 (LBA)

2023-05-03-raspios-bullseye-arm64-lite.img2      532480 4104191 3571712  1,7G 83 Linux



offset=8192 * sector size = 8192*512 = 4194304
```
#### Mount the kernel partition:                                                                                                          
                                                                                
```
mount -o loop,offset=4194304 2022-04-04-raspios-bullseye-arm64-lite.img boot                                                    
```


#### Create a default userconf file (user:pi , passwd:raspberry) :   

```
echo -n 'pi:' > userconf

echo 'raspberry' | openssl passwd -6 -stdin >> userconf

sudo cp userconf /mnt/
```

#### Enable ssh:

Create an empty file "ssh" in the image:

```
sudo touch /mnt/ssh
```

#### Start the machine:


```
qemu-system-aarch64 -m 1024 -M raspi3b -kernel /mnt/kernel8.img \

                    -dtb /mnt/bcm2710-rpi-3-b-plus.dtb \
                    
                    -sd ../A53/2023-05-03-raspios-bullseye-arm64-lite.img \
                    
                    -append "console=ttyAMA0 root=/dev/mmcblk0p2 rw rootwait rootfstype=ext4" \
                    
                    -nographic -device usb-net,netdev=net0 \
                    
                    -netdev user,id=net0,hostfwd=tcp::2222-:22
                    
```


Log in with: pi, raspberry.

                                                                                                                                                                                                                  
Now you can login from a client terminal with:

```
ssh -p2222 pi@localhost
```

And copy files with:

```
scp -P 2222 <FILE> pi@localhost:/home/pi/                                           
```
#### Remember:
Remember to expand the filesystem (use raspi-config) to get above 1.5G disk 

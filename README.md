# Create a 128T router template in EVE-NG

## **prepare EVE server**

1. create the directory on EVE server
2. SCP your license from your computer to EVE-NG
3. change into 128T directory
4. download 128T installer ISO file
5. rename install ISO to the filename EVE is expecting
6. create an 8 GB hard disk for the new template

```sh
mkdir /opt/unetlab/addons/qemu/128T-5.1.3
scp 128license.pem root@eve:/opt/unetlab/addons/qemu/128T-5.1.3/128license.pem
cd /opt/unetlab/addons/qemu/128T-5.1.3
curl -O --cert 128license.pem https://yum.128technology.com/isos/128T-5.1.4-1.el7.OTP.v1.x86_64.iso
mv 128T-5.1.4-1.el7.OTP.v1.x86_64.iso cdrom.iso
/opt/qemu/bin/qemu-img create -f qcow2 virtioa.qcow2 8G
```

## **create our new template image in EVE GUI**

1. add a new node to an empty lab, select Juniper 128T
2. add a network that will allow your device to talk on your LAN (cloud1 in my case maps to my second NIC on my EVE server)
3. before starting the router, set the flags to `-machine type=pc-1.0,accel=kvm -cpu host -vga std -usbdevice tablet -boot order=cd` and make sure that vnc is selected for console under router options
4. start router, click on device to open VNC console, select Install 128T Routing Software VGA Console (fourth option from menu)
5. install will take up to an hour, be patient
6. when prompted to shutdown, select yes
7. find your Lab ID by clicking on "Lab Details" on the left menu and copy the value of ID; you'll need this soon

## **finalize template image**

change into your lab's working directory (replace the UUID with your Lab ID found above), the last directory may be a 0 or 1
we need to commit your changes made to the disk image
remove the installer ISO file

```sh
cd /opt/unetlab/tmp/0/03108a19-9aec-4428-a60c-272be78cbcfa/1
/opt/qemu/bin/qemu-img commit virtioa.qcow2
rm /opt/unetlab/addons/qemu/128T-5.1.3/cdrom.iso
```

## **final step, promise**

the defaults for 128T definition need tweaking, specifically the CPU flags used to power up the VM. 

if this step is skipped, you will not be able to access the VM after it boots, you'll just get a black screen after GRUB completes; I believe this is due to the fact that 128T boots with the console baud set to 115200 and EVE isn't smart enough to adapt.

1. edit the 128T device profile
2. update the value set for qemu_options to match the line below
3. update the value set for console to match the line below

```sh
nano /opt/unetlab/html/templates/intel/128T.yml
```

```yaml
qemu_options: -machine type=pc-1.0,accel=kvm -cpu host -vga std -usbdevice tablet
console: vnc
```

run the fixpermissions script

```sh
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

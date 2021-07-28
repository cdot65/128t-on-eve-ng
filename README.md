# Create a 128T router template in EVE-NG

## **prepare EVE server**

create the directory on EVE server

- [ ] - `mkdir /opt/unetlab/addons/qemu/128T-5.1.3`

SCP your license from your computer to EVE-NG

- [ ] - `scp 128license.pem root@eve:/opt/unetlab/addons/qemu/128T-5.1.3/license.pem `

change into 128T directory

- [ ] - `cd /opt/unetlab/addons/qemu/128T-5.1.3`

download 128T installer ISO file

- [ ] - `curl -O --cert lic.pem https://yum.128technology.com/isos/128T-5.1.4-1.el7.OTP.v1.x86_64.iso `

rename install ISO to the filename EVE is expecting

- [ ] - `mv 128T-5.1.4-1.el7.OTP.v1.x86_64.iso cdrom.iso `

create an 8 GB hard disk for the new template

- [ ] `/opt/qemu/bin/qemu-img create -f qcow2 virtioa.qcow2 8G `

## **create our new template image in EVE GUI**

- add a new node to an empty lab, select Juniper 128T
- add a network that will allow your device to talk on your LAN (cloud1 in my case maps to my second NIC on my EVE server)
- before starting the router, set the flags to -machine type=pc-1.0,accel=kvm -cpu host -vga std -usbdevice tablet -boot order=cd and make sure that vnc is selected for console under router options
- start router, click on device to open VNC console, select Install 128T Routing Software VGA Console (fourth option from menu)
- install will take up to an hour, be patient
- when prompted to shutdown, select yes
- find your Lab ID by clicking on "Lab Details" on the left menu and copy the value of ID; you'll need this soon

## **finalize template image**

change into your lab's working directory (replace the UUID with your Lab ID found above), the last directory may be a 0 or 1

- [ ] `cd /opt/unetlab/tmp/0/03108a19-9aec-4428-a60c-272be78cbcfa/1`

we need to commit your changes made to the disk image

- [ ] `/opt/qemu/bin/qemu-img commit 0/virtioa.qcow2`

remove the installer ISO file

- [ ] `rm /opt/unetlab/addons/qemu/128T-5.1.3/cdrom.iso`

## **final step, promise**

the defaults for 128T definition need tweaking, specifically the CPU flags used to power up the VM. if this step is skipped, you will not be able to access the VM after it boots, you'll just get a black screen after GRUB completes; I believe this is due to the fact that 128T boots with the console baud set to 115200 and EVE isn't smart enough to adapt.

edit the 128T device profile

- [ ] `nano /opt/unetlab/html/templates/intel/128T.yml`

update the value set for qemu_options to match the line below

- [ ] `qemu_options: -machine type=pc-1.0,accel=kvm -cpu host -vga std -usbdevice tablet`

update the value set for console to match the line below

- [ ] `console: vnc`

run the fixpermissions script

- [ ] `/opt/unetlab/wrappers/unl_wrapper -a fixpermissions`

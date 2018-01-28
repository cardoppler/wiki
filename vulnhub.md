## Convert VMware to VirtualBox
```
PS C:\Program Files (x86)\VMware\VMware Player\OVFTool> .\ovftool.exe C:\Users\cardo\Downloads\KVM3\KioptrixVM3\KioptrixVM3.vmx C:\Users\cardo\Downloads\KVM3\KioptrixVM3\KioptrixVM3.ovf
Opening VMX source: C:\Users\cardo\Downloads\KVM3\KioptrixVM3\KioptrixVM3.vmx
Opening OVF target: C:\Users\cardo\Downloads\KVM3\KioptrixVM3\KioptrixVM3.ovf
Writing OVF package: C:\Users\cardo\Downloads\KVM3\KioptrixVM3\KioptrixVM3.ovf
Transfer Completed
Completed successfully
```
Then, in VirtualBox, File > Import Appliance > C:\Users\cardo\Downloads\KVM3\KioptrixVM3\KioptrixVM3.ovf

## Home lab network configuration

Kali:
- Adapter 1: NAT
- Adapter 2: Host-only

Target VM:
- Adapter 1: Host-only

So they can see each other and kali can browse the internet.

Add static pics:
![host-only_vm_network_configuration](https://github.com/cardoppler/cardoppler.tech/blob/master/static/host-only_vm_network_configuration.PNG?raw=true "host-only_vm_network_configuration")
![virtualbox_network_modes](https://github.com/cardoppler/cardoppler.tech/blob/master/static/virtualbox_network_modes.PNG?raw=true "virtualbox_network_modes")
![host_network_manager_adapter](https://github.com/cardoppler/cardoppler.tech/blob/master/static/host_network_manager_adapter.PNG?raw=true "host_network_manager_adapter")
![host_network_manager_DHCP_server](https://github.com/cardoppler/cardoppler.tech/blob/master/static/host_network_manager_DHCP_server.PNG?raw=true "host_network_manager_DHCP_server")

Other
- https://www.virtualbox.org/manual/ch06.html
- Bridged Adapter https://stackoverflow.com/questions/31922055/bridged-networking-not-working-in-virtualbox-under-windows-10

### Find the MAC address of the vulnhub box
VirtualBox > Select vm > Settings > Network > Advanced. Then compare the MAC Address value to what `netdiscover` finds
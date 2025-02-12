# Debian NAS

Ansible playbook to configure my Debian NAS, forked from Jeff Geerling's arm-nas repo

  - [Giga-Byte H97N-Wifi Pentium NAS](#ssd-nas)

## Hardware

### <a name="ssd-nas"></a>Debian NAS

The Debian NAS contains the following hardware:

  - (Motherboard) [Giga-Byte H97N-Wifi](https://www.gigabyte.com/au/Motherboard/GA-H97N-WIFI-rev-10/support#support-dl-driver)
  - (Case / PSU) [Just a junk case and PSU from eWaste](https://etse.me/insert-photo-here.jpg)
  - (CPU) [Intel Pentium G3450](https://www.intel.com/content/www/us/en/products/sku/80792/intel-pentium-processor-g3450-3m-cache-3-40-ghz/specifications.html)
  - (RAM) [8GB DDR3 1600 DIMM](https://etse.me)
  - (SSDs) [4x Random 2.5" SATA SSDs](https://etse.me)

## Preparing the hardware

The Debian NAS was a base install of Debian 9 netinst followed by the configuration below:

  1. Configure the network bond based on [this article](https://www.snel.com/support/lacp-bonding-on-debian-7-8-9/)
  2. `apt-get install ifenslave`
  3. Update the `/etc/network/interfaces` 

     ``` bash
      # This file describes the network interfaces available on your system
      # and how to activate them. For more information, see interfaces(5).

      source /etc/network/interfaces.d/*

      # The loopback network interface
      auto lo
      iface lo inet loopback

      # The primary network interface
      allow-hotplug eno1
      iface eno1 inet manual
              mtu 9000

      allow-hotplug enp2s0
      iface enp2s0 inet manual
              mtu 9000

      auto bond0
      iface bond0 inet dhcp
              slaves eno1 enp2s0
              #mtu 9000 # this wasn't working for some reason.  Moved to the post-up below
              bond-mode 802.3ad
              bond-miimon 100
              bond-downdelay 200
              bond-updelay 200
              post-up ip link set $IFACE mtu 9000
     ```

  4. Reboot
  5. Configure static DHCP assignment in UniFi Controller
  6. Run ansible onboarding script & ensure the Ansible host can contact the Debian NAS (ssh hostkey)

Confirm the SATA drives are recognized with `lsblk`.

## Running the playbook

Ensure you have Ansible installed, and can SSH into the NAS using `ssh user@nas-ip-or-address` without entering a password, then run:

``` bash
ansible-playbook main.yml
```

## Benchmarks

Using Jeff's Disk Benchmark script [`disk-benchmark.sh` script](https://github.com/geerlingguy/pi-cluster/blob/master/benchmarks/disk-benchmark.sh).

You can run it by copying it to the server, making it executable, and running it with `sudo`:

``` bash
wget https://raw.githubusercontent.com/geerlingguy/pi-cluster/master/benchmarks/disk-benchmark.sh
chmod +x disk-benchmark.sh
sudo MOUNT_PATH=/nvmepool/mercury TEST_SIZE=20g ./disk-benchmark.sh
```

## License

GPLv3 or later

## Author

Jeff Geerling

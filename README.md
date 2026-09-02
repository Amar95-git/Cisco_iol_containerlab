# Cisco IOL with Containerlab on Linux

This guide provides the Linux equivalent of Marc's Tech Blog article
**"Add Cisco IOL to Containerlab on macOS"**.

The goal is to:

1.  mount the Cisco CML REFPLAT ISO;
2.  extract the Cisco IOL L3 and IOL-L2 binaries;
3.  build Docker images using `srl-labs/vrnetlab`;
4.  use those images with Containerlab.

> **Important**
>
> This guide does not provide any Cisco binaries or software. You must
> use a legally obtained Cisco CML/REFPLAT image and comply with Cisco
> licensing terms.

------------------------------------------------------------------------

## 1. Prerequisites

You will need:

-   Linux;
-   Docker;
-   Git;
-   Make;
-   Containerlab;
-   a Cisco CML REFPLAT ISO image.

On Ubuntu/Debian:

``` bash
sudo apt update
sudo apt install -y git make docker.io
```

Check Docker:

``` bash
docker --version
```

Check Containerlab:

``` bash
containerlab version
```

Check your system architecture:

``` bash
uname -m
```

On an x86-64 Linux host, the expected result is:

``` text
x86_64
```

------------------------------------------------------------------------

## 2. Mount the REFPLAT ISO

Assume the ISO file is:

``` text
refplat-20260409-fcs.iso
```

Go to the directory containing the ISO:

``` bash
cd ~/Downloads
```

Create a mount point:

``` bash
sudo mkdir -p /mnt/refplat
```

Mount the ISO read-only:

``` bash
sudo mount -o loop,ro refplat-20260409-fcs.iso /mnt/refplat
```

Verify the contents:

``` bash
ls /mnt/refplat
```

Then:

``` bash
ls /mnt/refplat/virl-base-images/
```

On Linux, this replaces the macOS step:

``` text
open refplat-....iso
```

The macOS path:

``` text
/Volumes/REFPLAT/
```

becomes the following on Linux:

``` text
/mnt/refplat/
```

------------------------------------------------------------------------

## 3. Locate the IOL Images

Find the IOL images available in your REFPLAT ISO:

``` bash
find /mnt/refplat/virl-base-images \
  -maxdepth 2 \
  \( -name 'iol-*.tar.gz' -o -name 'ioll2-*.tar.gz' \)
```

For example, with a recent REFPLAT image you may see:

``` text
/mnt/refplat/virl-base-images/iol-xe-17-18-02/iol-xe-17-18-02.tar.gz
/mnt/refplat/virl-base-images/ioll2-xe-17-18-02/ioll2-xe-17-18-02.tar.gz
```

Create working directories:

``` bash
mkdir -p ~/cml-refplat/{iol,ioll2}
```

------------------------------------------------------------------------

# Extract IOL-L2

## 4. Extract the IOL-L2 Archive

Extract the archive:

``` bash
tar -xvf \
  /mnt/refplat/virl-base-images/ioll2-xe-17-18-02/ioll2-xe-17-18-02.tar.gz \
  -C ~/cml-refplat/ioll2
```

Enter the directory:

``` bash
cd ~/cml-refplat/ioll2
```

List the blobs by size:

``` bash
ls -lSh blobs/sha256/
```

The largest blob normally contains the IOL binary.

You can select it automatically:

``` bash
L2_BLOB=$(find blobs/sha256 -type f -printf '%s %p\n' \
  | sort -nr \
  | head -1 \
  | cut -d' ' -f2-)
```

Verify it:

``` bash
echo "$L2_BLOB"
file "$L2_BLOB"
```

Inspect its contents:

``` bash
tar -tf "$L2_BLOB"
```

You should find a file similar to:

``` text
x86_64_crb_linux_l2-adventerprisek9-ms.iol
```

Extract it:

``` bash
tar -xvf "$L2_BLOB" \
  x86_64_crb_linux_l2-adventerprisek9-ms.iol
```

Verify the file:

``` bash
file x86_64_crb_linux_l2-adventerprisek9-ms.iol
```

You can also check the IOS version:

``` bash
strings x86_64_crb_linux_l2-adventerprisek9-ms.iol \
  | grep '^Cisco IOS Software' \
  | head
```

------------------------------------------------------------------------

# Extract IOL L3

## 5. Extract the IOL L3 Archive

Extract the archive:

``` bash
tar -xvf \
  /mnt/refplat/virl-base-images/iol-xe-17-18-02/iol-xe-17-18-02.tar.gz \
  -C ~/cml-refplat/iol
```

Enter the directory:

``` bash
cd ~/cml-refplat/iol
```

Identify the largest blob:

``` bash
L3_BLOB=$(find blobs/sha256 -type f -printf '%s %p\n' \
  | sort -nr \
  | head -1 \
  | cut -d' ' -f2-)
```

Verify it:

``` bash
echo "$L3_BLOB"
file "$L3_BLOB"
```

Inspect its contents:

``` bash
tar -tf "$L3_BLOB"
```

You should find:

``` text
x86_64_crb_linux-adventerprisek9-ms.iol
```

Extract it:

``` bash
tar -xvf "$L3_BLOB" \
  x86_64_crb_linux-adventerprisek9-ms.iol
```

Check the IOS version:

``` bash
strings x86_64_crb_linux-adventerprisek9-ms.iol \
  | grep '^Cisco IOS Software' \
  | head
```

------------------------------------------------------------------------

## 6. Unmount REFPLAT

Once both binaries have been extracted:

``` bash
cd ~
sudo umount /mnt/refplat
```

------------------------------------------------------------------------

# Build the Containerlab Images

## 7. Clone srl-labs/vrnetlab

Create a directory for your images:

``` bash
mkdir -p ~/images
cd ~/images
```

Clone the vrnetlab fork used by Containerlab:

``` bash
git clone https://github.com/srl-labs/vrnetlab.git
```

The Cisco IOL build directory is:

``` bash
cd ~/images/vrnetlab/cisco/iol
```

> **Note**
>
> Use the `srl-labs/vrnetlab` fork, which is the vrnetlab implementation
> used by Containerlab.

------------------------------------------------------------------------

## 8. Copy and Rename the IOL-L2 Binary

Copy the L2 binary:

``` bash
cp ~/cml-refplat/ioll2/x86_64_crb_linux_l2-adventerprisek9-ms.iol \
  ~/images/vrnetlab/cisco/iol/cisco_iol-L2-17.18.02.bin
```

------------------------------------------------------------------------

## 9. Copy and Rename the IOL L3 Binary

Copy the L3 binary:

``` bash
cp ~/cml-refplat/iol/x86_64_crb_linux-adventerprisek9-ms.iol \
  ~/images/vrnetlab/cisco/iol/cisco_iol-17.18.02.bin
```

Verify both files:

``` bash
cd ~/images/vrnetlab/cisco/iol
ls -lh *.bin
```

You should have:

``` text
cisco_iol-17.18.02.bin
cisco_iol-L2-17.18.02.bin
```

> Replace `17.18.02` if your REFPLAT ISO contains a different version.

------------------------------------------------------------------------

## 10. Build the Docker Images

On an x86-64 Linux host:

``` bash
cd ~/images/vrnetlab/cisco/iol
make docker-image
```

If Docker requires root privileges:

``` bash
sudo make docker-image
```

Verify the resulting images:

``` bash
docker images vrnetlab/cisco_iol
```

You should get something similar to:

``` text
REPOSITORY           TAG
vrnetlab/cisco_iol   17.18.02
vrnetlab/cisco_iol   L2-17.18.02
```

### ARM64 Hosts

Check your architecture:

``` bash
uname -m
```

If the result is:

``` text
aarch64
```

you may need to force the amd64 platform:

``` bash
DOCKER_DEFAULT_PLATFORM=linux/amd64 make docker-image
```

On a native Linux `x86_64` host, this is normally unnecessary.

------------------------------------------------------------------------

# Use the Images with Containerlab

## 11. Create a Lab

Create a directory:

``` bash
mkdir -p ~/labs/iol
cd ~/labs/iol
```

Create:

``` text
iol.clab.yml
```

Example topology:

``` yaml
name: iol

topology:
  nodes:

    r1:
      kind: cisco_iol
      image: vrnetlab/cisco_iol:17.18.02

    r2:
      kind: cisco_iol
      image: vrnetlab/cisco_iol:17.18.02

    sw1:
      kind: cisco_iol
      image: vrnetlab/cisco_iol:L2-17.18.02
      type: L2

  links:
    - endpoints: ["r1:Ethernet0/1", "sw1:Ethernet0/1"]
    - endpoints: ["r2:Ethernet0/1", "sw1:Ethernet0/2"]
```

For an IOL-L2 switch, it is important to specify:

``` yaml
type: L2
```

------------------------------------------------------------------------

## 12. Deploy the Lab

Deploy it:

``` bash
sudo containerlab deploy -t iol.clab.yml
```

or:

``` bash
sudo clab deploy -t iol.clab.yml
```

Inspect the topology:

``` bash
sudo clab inspect -t iol.clab.yml
```

Give the Cisco devices a little time to finish booting.

------------------------------------------------------------------------

## 13. Connect to the Devices

Containerlab uses `Ethernet0/0` as the IOL management interface.

The default credentials documented for the Containerlab IOL integration
are:

``` text
Username: admin
Password: admin
```

Connect using SSH:

``` bash
ssh admin@<management-ip>
```

You can also inspect a container directly:

``` bash
docker exec -it <container-name> bash
```

------------------------------------------------------------------------

## 14. IOL Interface Mapping

The interface mapping starts as follows:

``` text
eth0 -> Ethernet0/0   Management
eth1 -> Ethernet0/1   Data
eth2 -> Ethernet0/2   Data
eth3 -> Ethernet0/3   Data
eth4 -> Ethernet1/0   Data
eth5 -> Ethernet1/1   Data
```

`Ethernet0/0` is therefore used for management.

For Containerlab data links, you will normally start with:

``` text
Ethernet0/1
```

------------------------------------------------------------------------

## 15. Destroy the Lab

Destroy the topology:

``` bash
sudo clab destroy -t iol.clab.yml
```

To also clean up the Containerlab-generated directory:

``` bash
sudo clab destroy -t iol.clab.yml --cleanup
```

------------------------------------------------------------------------

# macOS vs Linux

The main differences between the original macOS procedure and native
Linux are:

  -------------------------------------------------------------------------------------
  macOS Procedure                         Native Linux
  --------------------------------------- ---------------------------------------------
  `open refplat.iso`                      `mount -o loop,ro refplat.iso /mnt/refplat`

  `/Volumes/REFPLAT/`                     `/mnt/refplat/`

  `orb push -m clab ...`                  Not required

  `orb -m clab`                           Not required

  OrbStack Linux VM                       Not required

  `DOCKER_DEFAULT_PLATFORM=linux/amd64`   Normally not required on x86-64

  Eject ISO from Finder                   `sudo umount /mnt/refplat`
  -------------------------------------------------------------------------------------

In short, the Linux workflow is more direct:

``` text
REFPLAT ISO
    |
    v
mount /mnt/refplat
    |
    +---- IOL L3
    |
    +---- IOL L2
           |
           v
    srl-labs/vrnetlab
           |
           v
       Docker
           |
           v
     Containerlab
```

------------------------------------------------------------------------

# Troubleshooting

## `mailcap` Error When Opening the ISO

If you get:

``` text
Error: no "view" mailcap rules found for type "application/x-iso9660-image"
```

do not use:

``` bash
open refplat-20260409-fcs.iso
```

That step comes from the macOS procedure.

On Linux, mount the ISO instead:

``` bash
sudo mkdir -p /mnt/refplat

sudo mount -o loop,ro \
  refplat-20260409-fcs.iso \
  /mnt/refplat
```

Then:

``` bash
ls /mnt/refplat/virl-base-images/
```

## Check the Docker Images

``` bash
docker images | grep cisco_iol
```

## Check Running Containers

``` bash
docker ps
```

## Open a Shell Inside a Container

``` bash
docker exec -it <container-name> bash
```

------------------------------------------------------------------------

# References

-   Marc's Tech Blog --- Add Cisco IOL to Containerlab on macOS\
    https://marcstech.blog/archives/add-cisco-iol-containerlab-macos/

-   Containerlab --- Cisco IOL\
    https://containerlab.dev/manual/kinds/cisco_iol/

-   Containerlab --- vrnetlab\
    https://containerlab.dev/manual/vrnetlab/

-   srl-labs/vrnetlab\
    https://github.com/srl-labs/vrnetlab

------------------------------------------------------------------------

## Legal Notice

Cisco IOS, IOS-XE, CML, and IOL are Cisco technologies and software.

This guide only documents the process of using software to which you
already have legitimate access. It does not distribute Cisco images,
binaries, licenses, or other proprietary software.

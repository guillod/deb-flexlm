# Debian package for FLEXlm

## Description

Debian rules to generate a Debian package for [FLEXlm](https://www.openlm.com/what-is-flexlm-what-is-flexnet/) for amd64 architecture.
The FLEXlm installer `mathworks_network_license_manager_glnxa64.zip` provided on MATLAB website corresponding to the version in changelog has to be provided to build the package. Note that the version numbers correspond to MATLAB version and not to the FlexNet version.

The generated Debian package:
- installs FLEXlm executables in `/usr/sbin`
- installs vendor daemon provided in `daemons` in `/usr/sbin/`
- adds flexlm user
- adds systemd service

## Package build

1. Define the desired version:

        export VERSION=2022b

2. Clone the git repository containing the packages rules in an empty directory:

        git clone -b $VERSION https://github.com/guillod/deb-flexlm.git flexlm-$VERSION
        cd flexlm-$VERSION

3. Download the FLEXlm installer from [MATLAB website](https://fr.mathworks.com/support/install/license_manager_files.html) and put it in the current directory (`flexlm-$VERSION`):

        wget https://ssd.mathworks.com/supportfiles/downloads/R${VERSION}/license_manager/R${VERSION}/daemons/glnxa64/mathworks_network_license_manager_glnxa64.zip

4. Optionally, put other vendor daemons (like `maplelmg` for Maple) in the `daemons/` directory.
5. Generate the package for example with:

        debuild --no-lintian -i -b -uc -us

    or `sbuild`.

## Installation

Once generated, the package `flexlm_$VERSION_amd64.deb` can be installed for example with:

    sudo apt install ../flexlm_$VERSION_amd64.deb


License file has to be provided in `/etc/flexlm/`. To each license file, the corresponding vendor daemon executable (defined in the license file with the line starting with `VENDOR` or `DAEMON`) has to be provided in `/usr/sbin/` if not already included at package build.
The daemon executable can be located elsewhere but then the `VENDOR` or `DAEMON` line should include the full path in the following way:

    DAEMON|VENDOR daemon-name /path/to/daemon/daemon-name

Once licenses and daemons are added, the flexlm service can be restarted using:

    sudo systemctl restart flexlm

The port `27000` is used by FLEXlm so has to be opened, for example with:

    sudo ufw allow from 127.0.1.0/24 port 27000 to any

In addition, the ports used by vendor daemons need to be opened too.

## Setup of a MATLAB license

For MATLAB the daemon `MLM` is installer by the package. It might be necessary to add or modify the following lines at the beginning of the license file (just after the comments):

    SERVER hostname-of-license-server mac-address-of-license-server 27000
    DAEMON MLM PORT=27001

so that the MATLAB daemon always listen on port `27001` (which has to be opened).    

## Setup of a Maple license

For Maple the daemon `maplelmg` has to be in `/usr/sbin/`. It might be necessary to modify the following line of the license file:

    VENDOR maplelmg PORT=27002

so that the Maple daemon always listen on port `27002` (which has to be opened).  

## Debugging

The log of flexlm are handled by journald, so can be shown using:

    journalctl -u flexlm

The status of the license server can be checked by running `lmutil lmstat`.

## Contact

For comments, issues, bug-reports and requests, please use the issue tracker of the current repository. Otherwise the principal author can be reached at:

    Julien Guillod
    julien.guillod CHEZ sorbonne-universite.fr
    https://guillod.org/
    Department of Mathematics
    Sorbonne University
    France

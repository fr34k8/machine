# Docker Machine

Machine makes it really easy to create Docker hosts on local hypervisors and cloud providers. It creates servers, installs Docker on them, then configures the Docker client to talk to them.

It works a bit like this:

    $ machine create -d virtualbox dev
    [info] Downloading boot2docker...
    [info] Creating SSH key...
    [info] Creating VirtualBox VM...
    [info] Starting VirtualBox VM...
    [info] Waiting for VM to start...
    [info] "dev" has been created and is now the active host. Docker commands will now run against that host.

    $ machine ls
    NAME  	ACTIVE   DRIVER     	STATE 	URL
    dev   	*    	virtualbox 	Running   tcp://192.168.99.100:2375

    $ export DOCKER_HOST=`machine url` DOCKER_AUTH=identity

    $ docker run busybox echo hello world
    Unable to find image 'busybox' locally
    Pulling repository busybox
    e72ac664f4f0: Download complete
    511136ea3c5a: Download complete
    df7546f9f060: Download complete
    e433a6c5b276: Download complete
    hello world

    $ machine create -d digitalocean --digitalocean-access-token=... staging
    [info] Creating SSH key...
    [info] Creating Digital Ocean droplet...
    [info] Waiting for SSH...
    [info] "staging" has been created and is now the active host. Docker commands will now run against that host.

    $ machine ls
    NAME      ACTIVE   DRIVER         STATE     URL
    dev                virtualbox     Running   tcp://192.168.99.108:2376
    staging   *        digitalocean   Running   tcp://104.236.37.134:2376

Machine creates Docker hosts that are secure by default. The connection between the client and daemon is encrypted and authenticated using new identity-based authentication. If you'd like to learn more about this, it is being worked on in [a pull request on Docker](https://github.com/docker/docker/pull/8265).

## Try it out

Machine is still in its early stages. If you'd like to try out a preview build, download it here:

 - Mac OS X: https://github.com/docker/machine/releases/download/0.0.1/darwin
 - Linux: https://github.com/docker/machine/releases/download/0.0.1/linux

You will also need a version of Docker with identity authentication. Builds are available here:

 - Mac OS X: https://bfirsh.s3.amazonaws.com/docker/darwin/docker-1.3.1-dev-identity-auth
 - Linux: https://bfirsh.s3.amazonaws.com/docker/linux/docker-1.3.1-dev-identity-auth

## Drivers

### VirtualBox

Creates machines locally on [VirtualBox](https://www.virtualbox.org/). Requires VirtualBox to be installed.

Options:

 - `--virtualbox-boot2docker-url`: The URL of the boot2docker image. Defaults to the latest available version.
 - `--virtualbox-disk-size`: Size of disk for the host in MB. Default: `20000`
 - `--virtualbox-memory`: Size of memory for the host in MB. Default: `1024`

### Digital Ocean

Creates machines on [Digital Ocean](https://www.digitalocean.com/). You need to create a personal access token under "Apps & API" in the Digital Ocean Control Panel and pass that to `machine create` with the `--digitalocean-access-token` option.

Options:

 - `--digitalocean-access-token`: Your personal access token for the Digital Ocean API.
 - `--digitalocean-image`: The name of the Digital Ocean image to use. Default: `docker`
 - `--digitalocean-region`: The region to create the droplet in. Default: `nyc3`
 - `--digitalocean-size`: The size of the Digital Ocean driver. Default: `512mb`

### Microsoft Azure

Create machines on [Microsoft Azure](http://azure.microsoft.com/).

You need to create a subscription with a cert. Run these commands:

    $ openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.pem -out mycert.pem
    $ openssl pkcs12 -export -out mycert.pfx -in mycert.pem -name "My Certificate"
    $ openssl x509 -inform pem -in mycert.pem -outform der -out mycert.cer

Go to the Azure portal, go to the "Settings" page, then "Manage Certificates" and upload `mycert.cer`.

Grab your subscription ID from the portal, then run `machine create` with these details:

    $ machine create -d azure --azure-subscription-id="SUB_ID" --azure-subscription-cert="mycert.cer"

Options:

 - `--azure-subscription-id`: Your Azure subscription ID.
 - `--azure-subscription-cert`: Your Azure subscription cert.

### VMware Fusion

Creates machines locally on [VMware Fusion](http://www.vmware.com/products/fusion). Requires VMware Fusion to be installed.

Options:

 - `--fusion-boot2docker-url`: The URL of the boot2docker image. Defaults to the latest available version.
 - `--fusion-disk-size`: Size of disk for the host in MB. Default: `20000`
 - `--fusion-memory`: Size of memory for the host in MB. Default: `1024`

### VMware vSphere

Creates machines on a [VMware vSphere](http://www.vmware.com/products/vsphere) Virtual Infrastructure. Requires a working vSphere installation.

Options:

 - `--vsphere-boot2docker-url=""`: The URL of the boot2docker image. Defaults to the latest available version.
 - `--vsphere-cpu=2`: Number of CPUs for the host. Default: `2`
 - `--vsphere-memory=2048`: Size of memory for the host in MB. Default: `2048`
 - `--vsphere-disk-size=20000`: Size of disk for the host in MB. Default: `20000`
 - `--vsphere-vcenter=""`: IP/hostname for vCenter host.
 - `--vsphere-username=""`: vCehter username.
 - `--vsphere-password=""`: vCenter password.
 - `--vsphere-datacenter=""`: vSphere Datacenter for the host.
 - `--vsphere-compute-ip=""`: ESXi compute host IP where the host docker VM will be instantiated.
 - `--vsphere-datastore=""`: vSphere Datastore for the host.
 - `--vsphere-network=""`: vSphere network where the host will be attached
 - `--vsphere-pool=""`: vSphere resource pool for the host. Optional.


### VMware vCloud Air

Creates machines on the [vCloud Air](http://vcloud.vmware.com) subscription service. You need an account within an existing subscription of vCloud Air VPC or Dedicated Cloud.

 - `--vcloudair-username=""`: vCloud Air username
 - `--vcloudair-password=""`: vCloud Air user password.
 - `--vcloudair-computeid=""`: vCloud Air Compute ID (if using Dedicated Cloud)
 - `--vcloudair-vdcid=""`: vCloud Air VDC ID
 - `--vcloudair-name=""`: vCloud Air vApp host name. Default: `<autogenerated>`
 - `--vcloudair-catalog="Public Catalog"`: The vCloud Air Catalog. Default: `Public Catalog`
 - `--vcloudair-catalogitem="Ubuntu Server 12.04 LTS (amd64 20140927)`: The vCloud Air Catalog Item. Default: `Ubuntu Server 12.04 LTS (amd64 20140927)`
 - `--vcloudair-orgvdcnetwork=""`: vCloud Air Org VDC Network. Default: `<vdcid>-default-routed`
 - `--vcloudair-edgegateway=""`: vCloud Air Org Edge Gateway. Default: `<vdcid>`
 - `--vcloudair-publicip=""`: vCloud Air Org Public IP to use
 - `--vcloudair-cpu-count=1`: vCloud Air VM Cpu Count. Default: `1`
 - `--vcloudair-mem-size=2048`: vCloud Air VM Memory Size in MB. Default: `2048` 
 - `--vcloudair-provision=true`: vCloud Air Install Docker binaries. Default: `true`
 - `--vcloudair-ssh-port=22`: vCloud Air SSH port
 - `--vcloudair-docker-port=2376`: vCloud Air Docker port. Default: `2376` 

## Contributing

Want to hack on Machine? [Docker's contributions guidelines](https://github.com/docker/docker/blob/master/CONTRIBUTING.md) apply.

To build, run:

    $ go get github.com/docker/machine
    $ cd $GOPATH/src/github.com/docker/machine
    $ script/build
    $ ./machine

## Creators

**Ben Firshman**

- <https://twitter.com/bfirsh>
- <https://github.com/bfirsh>


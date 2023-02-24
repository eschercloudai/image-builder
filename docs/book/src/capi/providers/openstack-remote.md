# Building Images on OpenStack

## Hypervisor

The image is built using Openstack.

### Prerequisites for Openstack builds

Check any prerequisites over on the [Packer docs for the Openstack builder](https://developer.hashicorp.com/packer/plugins/builders/openstack).

Also ensure you have a [Ubuntu 20.04](https://cloud-images.ubuntu.com/focal/current/) or [Ubuntu 22.04](https://cloud-images.ubuntu.com/jammy/current/) cloud image available in your Openstack instance before continuing as it will need to be referenced.
This also supports Flatcar - so far only Stable has been tested.

#### Note
> Other OS's could be supported adn additions are welcome.

## Setup Openstack authentication
Ensure you have set up your method of authentication - [examples here](https://docs.openstack.org/python-openstackclient/zed/cli/authentication.html).
You can also check out the [packer builder](https://developer.hashicorp.com/packer/plugins/builders/openstack#configuration-reference) for more information on authentication.

You should be able to run commands against openstack before running this builder otherwise it will fail.
You can test with a simple command such as `openstack image list`.

## Building Images

The build [prerequisites](../capi.md#prerequisites) for using `image-builder` for
building Openstack images are managed by running:

```bash
cd image-builder/images/capi
make deps-openstack
```

### Define variables for Openstack build
Using the [Openstack packer provider](https://developer.hashicorp.com/packer/plugins/builders/openstack), an instance will be deployed and an image built from it.
A certain set of environment variables (example below) must be defined in a josn file  and reference it during `make build-openstack-ubuntu-xxxx`. Please replace xxxx with 2004 or 2204.

Replace UPPERCASE variables with your values.
```json
{
  "source_image": "OPENSTACK_SOURCE_IMAGE_ID",
  "networks": "OPENSTACK_NETWORK_ID",
  "flavor": "OPENSTACK_INSTANCE_FLAVOR_NAME",
  "floating_ip_network": "OPENSTACK_FLOATING_IP_NETWORK_NAME"
}
```

#### Note:
> The following Kuberentes versions have been tested: 1.23, 1.24, 1.25 and 1.26. <br>
The following crictl versions have been tested: 1.23.0, 1.24.0, 1.25.0 and 1.26.0.

Check out `images/capi/packer/openstack/packer.json` for more variables such as allowing the use of floating IPs and config drives.

### Building Image on Openstack

From the `images/capi` directory, run `PACKER_VAR_FILES=var_file.json make build-openstack-<DISTRO>`.

An instance is built in Openstack from the source image defined and once completed, the instance is shutdown and the image gets created that can then be used afterwards.
This image will default to private however this can be controlled via `image_visibility `.

For building a ubuntu-2204 based capi image with Kubernetes 1.25.3, run the following commands:

#### Example
```bash
$ git clone https://github.com/kubernetes-sigs/image-builder.git
$ cd image-builder/images/capi/
$ make deps-openstack
$ make build-openstack-ubuntu-2204
```

The resulting image will be named `ubuntu-2204-kube-v1.25.3` based on the following format: `ubuntu-XXXX-kube-KUBERNETES_SEMVER`.

This can be modified by overriding the `image_name` variable if required.

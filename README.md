# Test Podman nested in unprivileged containers

(Unfinished) Project to help developing and testing Podman images.
One of the unique features of Podman is to run application containers rootless. Therefore the aim for this Vagrant environment is to run rootless podman containers running nested in unprivileged LXD container, with LXD storage backend ZFS and podman storage backend fuse-overlayfs. All this happens in a virtual machine that is configured by Vagrant via Ansible.


# Usage

ssh-keygen -t rsa -b 2048 -C "developer" -f sshkey/id_rsa_dev -P '' -q
vagrant up
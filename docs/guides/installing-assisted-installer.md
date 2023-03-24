# Install OKD/OCP on OCI using agnostic method

Install an agnostic OCP/OKD Cluster using Assisted-Installer.

## Pre-requisites

- Install ansible and dependencies:

```bash
pip install -r requirements.txt
```

- Install the Collections

```bash
ansible-galaxy collection install -r requirements.yml
```

Ensure the `aicli` is installed:

```bash
$ pip list | grep aicli
aicli                   99.0.202303091315
```

Ensure the Ansible collection `karmab.aicli` is installed:

```bash
$ ansible-galaxy collection list | grep "karmab.aicli"
karmab.aicli         1.0.0
```

Set your offline token in your environment, you will find it on
https://cloud.redhat.com/openshift/token:

```bash
export AI_OFFLINETOKEN=<token>
```

If you plan you use the Assisted Installer from staging or integration, set the
following environment variables accordingly:

```bash
export AI_INTEGRATION=1
# or
export AI_STAGING=1
```

## Usage

### Configuration

Here is a configuration sample:

```yaml
# Specify that we want to use the Assisted-Installer
installation_method: assisted_installer

# Specify basic cluster configuration as you would do using UPI
cluster_name: my-cluster
config_base_domain: my-domain.com

config_ssh_key: my-public-key
config_pull_secret_file: path-to-my-pull-secret

cluster_profile: ha

# You can list the versions available in the Assisted-Installer with aicli:
# aicli info service
config_cluster_version: 4.12

# with the OCI provider, a custom image will be created from the discovery ISO generated by the Assisted Installer
os_mirror: yes
os_mirror_from: stream_artifacts

os_mirror_to_provider: oci
os_mirror_to_oci:
  compartment_id: ...
  bucket: qcow
  image_type: QCOW2
```

### Create cluster in Assisted Installer

Create the cluster in the Assisted-Installer, by default, platform of the
cluster will be set to `none` and a minimal discovery ISO will be provided:
```bash
ansible-playbook mtulio.okd_installer.config -e mode=create -e @./vars-my-config.yaml
```

You should be able to review the cluster on the UI on
https://console.redhat.com/openshift/assisted-installer/clusters/ (for
production).

### Install cluster

Run the following playbook:
```bash
ansible-playbook mtulio.okd_installer.config -e mode=install -e @./vars-my-config.yaml
```

It will wait for all the hosts to be discovered and for all validations to
pass, then it will assign the roles (master or workers) to the hosts, start
the installation and wait for its completion.
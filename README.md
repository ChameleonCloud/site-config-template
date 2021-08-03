# Site configuration for chi-in-a-box

This repo is both a template for your site configuration, as well as the recommended method for keeping track of changes.

## Initial Setup

### Clone the Repo

First, clone this repo to your deploy host, and pick a destination. For these examples, we'll use `/opt/site-config`
```shell
git clone https://github.com/ChameleonCloud/site-config-template.git /opt/site-config
cd /opt/site-config
```

### Create a new branch for your site

When starting, you will create a new branch via

```shell
git checkout -b chi_example_production
```

## Make initial changes

Set the `CC_ANSIBLE_SITE` environment variable to the path you chose.
This can be done per-command, or made persistent in your shell.

```shell
export CC_ANSIBLE_SITE=/opt/site-config
```
### Generate and encrypt secrets

Running the following will create an ansible vault_password file, generate passwords for all templated values in `passwords.yml`, and then encrypt that file.

```
./cc-ansible --site /opt/site-config init
```
### Configure inventory

1. In `inventory/host_vars`, create a file named `<hostname>.yml`, where hostname is the hostname of your deploy machine.
    Ensure it looks like the following:
    ```yaml
    # we assume that this is running on the deploy host
    ansible_connection: local

    # Name of the interface used for the public API endpoints
    kolla_external_vip_interface: public_interface_name

    # Name of interface used for private API endpoints
    network_interface: private_interface_name
    ```
1. Modify `inventory/hosts.ini` to replace `<host>` in each section with the hostname used in part 1. The top should look like:
    ```ini
    [control]
    hostname_here

    [network]
    hostname_here

    [compute]
    # No compute node; this is a baremetal-only cluster.

    [monitoring]
    hostname_here

    [storage]
    hostname_here

    [deployment]
    localhost ansible_connection=local
    ```
    Ansible will look up specific variables for each host in `host_vars/hostname.yml`.

### Configure `defaults.yml`

defaults.yml contains all of your site-wide configuration, and is overridden by configurations in inventory.

For a minimal working example, meaning a baremetal control-plane with no nodes yet, the following configurations should work.

```yaml
# Name of your site
openstack_region_name: CHI@XYZ
chameleon_site_name: xyz
keystone_idp_client_id: chi-xyz

# URL resolving to the public API address
kolla_external_fqdn: chi.xyz.fqdn
# Address for HAProxy virtual IP, for public APIs
kolla_external_vip_address: "100.0.0.1"
# Address for HAProxy virtual IP, for private APIs
kolla_internal_vip_address: "10.0.0.1"

# If you have a cert, move it to `/certificates/haproxy.pem`, and set this to yes
kolla_enable_tls_external: no

# If you received a client ID from Chameleon for federation, comment these lines
enable_keystone_federation: no
enable_keystone_federation_openid: no
```

## Commit your changes
Finally, add the changed files and commit.

```shell
git add inventory defaults.yml passwords.yml
git commit -m "initial config for site <sitename>"
```

## Handling Secrets

This method encrypts the contents of passwords.yml. It's important to note that the
contents of `certificates/`, and `vault_password`, should not be committed to your repository.

Be sure to keep regular backups, so you don't lose these.

## Remote storage

You can store your configuration in a git remote, just as with any git repo. Again, we
recommend using a private repository as an extra layer of security against leaking secrets.

### Rename and add your remote
```shell
git remote rename origin chameleon
git remote add origin <your_repo_url>
git push --set-upstream origin/branchname
```

## Bringing in updates and new features

We periodically add new features, many of which include accompanying configuration.
To audit and include these changes, you can use standard git diff and merge tools.

```
git diff chameleon/main
```
```
git merge chameleon/main
```
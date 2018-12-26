# DCOS-Local-Package-Registry
This registry covers the process to deploy a Local Package Registry using local storage on Enterprise DC/OS 1.12.  It covers accounting for Airgapped Clusters, Creating & Attaching the Registry, managing Packages in Repo, and some Lessons Learned

## Airgap Cluster Preparation
If deploying an airgapped cluster that will have no access to the outside world, you will want to remove reference to Internet-based catalogs

Remove Default Catalog entry (as it cannot be accessed)

`dcos package repo remove Universe`

## Create and Attach Registry

Add the Local Bootstrap Registry

`dcos package repo add "Bootstrap Registry" https://registry.component.thisdcos.directory/repo`

Install Enterprise CLI

`dcos package install --yes dcos-enterprise-cli`

Creates Service Account Keys

`dcos security org service-accounts keypair private-key.pem public-key.pem`

Create Service Account

`dcos security org service-accounts create -p public-key.pem -d "dcos_registry service account" registry-account`

Create Service Account Secret

`dcos security secrets create-sa-secret --strict private-key.pem registry-account registry-private-key`

Grant permissions to "registry-account" Service Account

`dcos security org users grant registry-account dcos:adminrouter:ops:ca:rw full`

Create registry.options.json file

`echo '{"registry":{"service-account-secret-path":"registry-private-key"}}' > registry-options.json`

Install Package Registry 

`dcos package install package-registry --options=registry-options.json --yes`

## Connect Local Registry to DC/OS Cluster

Connect Local Registry to DC/OS 

`dcos package repo add --index=0 Registry https://registry.marathon.l4lb.thisdcos.directory/repo`

Execute on ALL AGENT NODES for Docker to trust local registry
```
sudo mkdir -p /etc/docker/certs.d/registry.marathon.l4lb.thisdcos.:443
sudo cp /run/dcos/pki/CA/ca-bundle.crt /etc/docker/certs.d/registry.marathon.l4lb.thisdcos.directory:443/ca.crt
sudo systemctl restart docker
```

## Manage Packages in DC/OS Package Registry

Below link contains references to supported package registries:

`https://downloads.mesosphere.com/universe/packages/packages.html`

Download a Package (eg Jenkins)

`wget https://downloads.mesosphere.com/universe/packages/jenkins/3.5.2-2.107.2/jenkins-3.5.2-2.107.2.dcos`

Search Packages in Local Package Registry

`dcos package search`

Add Package (eg Jenkins):

`dcos registry add --dcos-file jenkins-3.5.2-2.107.2.dcos`

Describe Package (eg Jenkins):

`dcos registry describe --package-name jenkins --package-version 3.5.2-2.107.2`

Remove Package (eg Jenkins)

`dcos registry remove --package-name jenkins --package-version 3.5.2-2.107.2`

## Lessons Learned:

**Package Versioning**

When deploying packages of the same name, the Local Package Registry will save them under the same heading and allow you to select the version to be deployed from a drop-down box.


**Deploying Packages**

Local registry packages can be deployed as normal from both the GUI and the CLI interfaces.


**Default Packages**

When uploading packages, the most recent one uploaded will be treated as the default even if the version number is lower than previous ones.


**Deploying Kubernetes**

When Deploying Kubernetes Cluster, make sure to select "Use Agent Docker Certs" (Service Section, borrom of page) or installation will crashloop and stall at the Control Plane deployment.

![IMAGE](quiver-image-url/69FB11EFAEE4B847A8C33B3FEC5ABDFC.jpg =309x179)

  

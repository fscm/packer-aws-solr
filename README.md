# Apache Solr AMI

AMI that should be used to create virtual machines with Apache Solr
installed.

## Synopsis

This script will create an AMI with Apache Solr installed and with all of
the required initialization scripts.

The AMI resulting from this script should be the one used to instantiate a
Solr server (standalone or cluster).

## Getting Started

There are a couple of things needed for the script to work.

### Prerequisites

Packer and AWS Command Line Interface tools need to be installed on your local
computer.
To build a base image you have to know the id of the latest Debian AMI files
for the region where you wish to build the AMI.

#### Packer

Packer installation instructions can be found
[here](https://www.packer.io/docs/installation.html).

#### AWS Command Line Interface

AWS Command Line Interface installation instructions can be found [here](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)

#### Debian AMI's

This AMI will be based on an official Debian AMI. The latest version of that
AMI will be used.

A list of all the Debian AMI id's can be found at the Debian official page:
[Debian official Amazon EC2 Images](https://wiki.debian.org/Cloud/AmazonEC2Image/)

### Usage

In order to create the AMI using this packer template you need to provide a
few options.

```
Usage:
  packer build \
    -var 'aws_access_key=AWS_ACCESS_KEY' \
    -var 'aws_secret_key=<AWS_SECRET_KEY>' \
    -var 'aws_region=<AWS_REGION>' \
    -var 'solr_version=<SOLR_VERSION>' \
    [-var 'option=value'] \
    solr.json
```

#### Script Options

- `aws_access_key` - *[required]* The AWS access key.
- `aws_ami_name` - The AMI name (default value: "Solr").
- `aws_ami_name_prefix` - Prefix for the AMI name (default value: "").
- `aws_instance_type` - The instance type to use for the build (default value: "t2.micro").
- `aws_region` - *[required]* The regions were the build will be performed.
- `aws_secret_key` - *[required]* The AWS secret key.
- `java_build_number` - Java build number (default value: "11").
- `java_major_version` - Java major version (default value: "8").
- `java_token` - Java link token (default version: "d54c1d3a095b4ff2b6607d096fa80163").
- `java_update_version` - Java update version (default value: "131").
- `solr_version` - *[required]* Solr version.
- `system_locale` - Locale for the system (default value: "en_US").
- `zookeeper_version` - Zookeeper version (default value: "3.4.9").

### Instantiate a Cluster

In order to end up with a functional Solr Cluster some configurations have to
be performed after instantiating the servers.

To help perform those configurations a small script is included on the AWS
image. The script is called **solr_config**.

A Zookeeper instance or cluster (for production environments) is required to
instantiate a Solr cluster.

If required, a Zookeeper server/node can be started within this image (see
below for more information) however, for production environments, it is
recommended to use a dedicated and separated Zookeeper instance or cluster.

#### Solr Configuration Script

The script can and should be used to set some of the Solr options as well as
setting the Solr service to start at boot.

```
Usage: solr_config [options]
```

##### Options

* `-a <ADDRESS>` - Sets the Solr node address exposed to the cluster (default value is 'localhost').
* `-D` - Disables the Solr service from start at boot time.
* `-E` - Enables the Solr service to start at boot time.
* `-m <MEMORY>` - Sets Solr heap size. Values should be provided following the same Java heap nomenclature.
* `-S` - Starts the Solr service after performing the required configurations (if any given).
* `-W <SECONDS>` - Waits the specified amount of seconds before starting the Solr service (default value is '0').
* `-z <ENDPOINT>` - Sets a Zookeeper server endpoint to be used by the Solr node (defaut value is 'localhost:2181'). Several Zookeeper endpoints can be set by either using extra `-z` options or if separated with a comma on the same `-z` option.

#### Configuring a Solr node

To prepare an instance to act as a Solr node the following steps need to be
performed.

Run the configuration tool (*solr_config*) to configure the instance.

```
solr_config -a solr01.mydomain.tld -E -S -z zookeeper01.mydomain.tld:2181
```

After this steps a Solr node (for either a single instance or a cluster setup)
should be running and configured to start on server boot.

More options can be used on the instance configuration, see the
[Configuration Script](#solr-configuration-script) section for more details

### Instantiate Zookeeper

Is it possible to use the included Zookeeper installation to instantiate a
Zookeeper node (standalone or as part of a cluster).

In order to end up with a functional Zookeeper node some configurations have to
be performed after instantiating the servers.

To help perform those configurations a small script is included on the AWS
image. The script is called **zookeeper_config**.

#### Zookeeper Configuration Script

The script can and should be used to set some of the Zookeeper options as well
as setting the Zookeeper service to start at boot.

```
Usage: zookeeper_config [options]
```

##### Options

* `-D` - Disables the Zookeeper service from start at boot time.
* `-E` - Enables the Zookeeper service to start at boot time.
* `-i <ID>` - Sets the Zookeeper broker ID (default value is '1').
* `-m <MEMORY>` - Sets Zookeeper maximum heap size. Values should be provided following the same Java heap nomenclature.
* `-n <ID:ADDRESS>` - The ID and Address of a cluster node (e.g.: '1:127.0.0.1'). Should be used to set all the Zookeeper nodes. Several Zookeeper nodes can be set by either using extra `-n` options or if separated with a comma on the same `-n` option.
* `-S` - Starts the Zookeeper service after performing the required configurations (if any given).
* `-W <SECONDS>` - Waits the specified amount of seconds before starting the Zookeeper service (default value is '0').

#### Configuring a Zookeeper Node

To prepare an instance to act as a Zookeeper node the following steps need to
be performed.

Run the configuration tool (*zookeeper_config*) to configure the instance.

```
zookeeper_config -E -S
```

After this steps a Zookeeper node (for a standalone setup) should be running
and configured to start on server boot.

For a cluster with more than one Zookeeper node other options have to be
configured on each instance using the same configuration tool
(*zookeeper_config*).

```
zookeeper_config -E -i 1 -n 1:zookeeper01.mydomain.tld -n 2:zookeeper02.mydomain.tld,3:zookeeper03.mydomain.tld -S
```

After this steps, the first node of the Zookeeper cluster (for a three node
cluster) should be running and configured to start on server boot.

More options can be used on the instance configuration, see the
[Configuration Script](#zookeeper-configuration-script) section for more details

## Services

This AMI will have the SSH service running as well as the Solr services. The
Zookeeper service may also be running on this AMI.
The following ports will have to be configured on Security Groups.

| Service      | Port      | Protocol |
|--------------|:---------:|:--------:|
| SSH          | 22        |    TCP   |
| Zookeeper    | 2181      |    TCP   |
| Zookeeper    | 2888:3888 |    TCP   |
| Solr         | 8983      |    TCP   |

## Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request

## Versioning

This project uses [SemVer](http://semver.org/) for versioning. For the versions
available, see the [tags on this repository](https://github.com/fscm/packer-aws-solr/tags).

## Authors

* **Frederico Martins** - [fscm](https://github.com/fscm)

See also the list of [contributors](https://github.com/fscm/packer-aws-solr/contributors)
who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE)
file for details

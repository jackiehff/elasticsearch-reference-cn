# Repositories
Repositoriesedit

On this page
APT
YUM
Elasticsearch Reference:
Getting Started
Setup
Configuration
Running as a Service on Linux
Running as a Service on Windows
Directory Layout
Repositories
Upgrading
Breaking changes
API Conventions
Document APIs
Search APIs
Aggregations
Indices APIs
cat APIs
Cluster APIs
Query DSL
Mapping
Analysis
Modules
Index Modules
Testing
Glossary of terms
Release Notes
We also have repositories available for APT and YUM based distributions. Note that we only provide binary packages, but no source packages, as the packages are created as part of the Elasticsearch build.

We have split the major versions in separate urls to avoid accidental upgrades across major version. For all 2.x releases use 2.x as version number, for 3.x.y use 3.x etc…

We use the PGP key D88E42B4, Elasticsearch Signing Key, with fingerprint

4609 5ACC 8548 582C 1A26 99A9 D27D 666C D88E 42B4
to sign all our packages. It is available from https://pgp.mit.edu.

APTedit

Download and install the Public Signing Key:

wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
Save the repository definition to /etc/apt/sources.list.d/elasticsearch-2.x.list:

echo "deb http://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list
Warning
Use the echo method described above to add the Elasticsearch repository. Do not use add-apt-repository as it will add a deb-src entry as well, but we do not provide a source package. If you have added the deb-src entry, you will see an error like the following:

Unable to find expected entry 'main/source/Sources' in Release file (Wrong sources.list entry or malformed file)
Just delete the deb-src entry from the /etc/apt/sources.list file and the installation should work as expected.

Run apt-get update and the repository is ready for use. You can install it with:

sudo apt-get update && sudo apt-get install elasticsearch
Warning
If two entries exist for the same Elasticsearch repository, you will see an error like this during apt-get update:

Duplicate sources.list entry http://packages.elastic.co/elasticsearch/2.x/debian/ ...`
Examine /etc/apt/sources.list.d/elasticsearch-2.x.list for the duplicate entry or locate the duplicate entry amongst the files in /etc/apt/sources.list.d/ and the /etc/apt/sources.list file.

Configure Elasticsearch to automatically start during bootup. If your distribution is using SysV init, then you will need to run:

sudo update-rc.d elasticsearch defaults 95 10
Otherwise if your distribution is using systemd:

sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
YUMedit

Download and install the public signing key:

rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
Add the following in your /etc/yum.repos.d/ directory in a file with a .repo suffix, for example elasticsearch.repo

[elasticsearch-2.x]
name=Elasticsearch repository for 2.x packages
baseurl=http://packages.elastic.co/elasticsearch/2.x/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
And your repository is ready for use. You can install it with:

yum install elasticsearch
Configure Elasticsearch to automatically start during bootup. If your distribution is using SysV init, then you will need to run:

Warning
The repositories do not work with older rpm based distributions that still use RPM v3, like CentOS5.

chkconfig --add elasticsearch
Otherwise if your distribution is using systemd:

sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service

# cldemo-automation-puppet




Configuring the puppetmaster
----------------------------
We will be installing our puppetmaster on `oob-mgmt-server`. We are going to
download the demo package, install the latest version of puppet, and create a
link in the puppet configuration directory to point to our puppet code instead.

    git clone https://github.com/cumulusnetworks/cldemo-automation-puppet
    cd cldemo-automation-puppet
    sudo su
    wget https://apt.puppetlabs.com/puppetlabs-release-pc1-trusty.deb
    dpkg -i puppetlabs-release-pc1-trusty.deb
    apt-get update
    apt-get install puppetserver -qy
    rm -rf /etc/puppetlabs/code/environments/production
    ln -s  /home/cumulus/cldemo-automation-puppet/production /etc/puppetlabs/code/environments/production

If running this on a virtual machine, open the file `/etc/default/puppetserver`
and configure the JVM to use 512 MB of RAM.

    JAVA_ARGS="-Xms512m -Xmx512m"

Start the puppetmaster.

    service puppetserver restart






Configuring the puppet agents
-----------------------------
Then configure the puppet agents on the switches and servers. Note that the
wget is to a different URL. Keep the puppetmaster open in a seperate terminal
window.

    sudo su
    wget https://apt.puppetlabs.com/pool/cumulus/PC1/p/puppetlabs-release-pc1/puppetlabs-release-pc1_0.9.4-1cumulus_all.deb
    dpkg -i puppetlabs-release-pc1_0.9.4-1cumulus_all.deb
    apt-get update
    apt-get install puppet-agent -qy

Edit the file `/etc/puppetlabs/puppet/puppet.conf` and add the line

    server = oob-mgmt-server.lab.local

Run

    /opt/puppetlabs/bin/puppet agent --test

This will create a certificate on the server that needs to be signed.

    # on the puppetmaster
    /opt/puppetlabs/bin/puppet cert list
    # find the name of the node
    /opt/puppetlabs/bin/puppet cert sign leaf01.lab.local

Back on the puppet agent, run the test command again.

    /opt/puppetlabs/bin/puppet agent --test

Assuming everything worked, run the following command to enable the puppet agent
in the background. Roughly every 30 minutes, the puppet agent will attempt to
query the puppetmaster for updated configuration. To force a client to update,
run the test command again.

    sudo service puppet start
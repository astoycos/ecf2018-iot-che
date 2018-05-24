
Once you have the playbooks, setting up the cluster is simple:

    ansible-playbook -i ansible-hosts.cfg openshift-ansible/playbooks/prerequisites.yml
    ansible-playbook -i ansible-hosts.cfg openshift-ansible/playbooks/deploy_cluster.yml

Then, create the initial password file with your desired user

    htpasswd -c /etc/origin/master/htpasswd iot

You should be able to login with that user on the openshift console.

Also create a set of PVs. In order to do so copy the `pvcreate` script to the master
and execute:

    ./pvcreate 20Gi {00..04}
    ./pvcreate 10Gi {05..09}
    ./pvcreate 1Gi {10..99}

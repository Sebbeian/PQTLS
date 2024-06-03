# PQTLS

OQS EXPERIMENT

NB! To use NREC you must first apply for ann account through NREC.no

Files for this test:

1. Create a VM in NREC or set up another environment on a self-chosen service
  1.1 Build a Instance with the following credential
       Source/OS: GOLD Ubuntu Linux 9
       Flavor: m1.small
       Networks: dualStack
       Security Group: SSH_and_ICMP
       Key Pair: Create Key Pair: SSH





- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

DIGICERT EXPERIMENT

NB! 
- To use NREC you must first apply for ann account through NREC.no
- You need to use a variant of Rocky Linux 
- Have a security group that allows SSH-connectrivity (my NREC setup enables this)

Files for this test:
- quantum_demo (the certificate files)
- linux-install-docker (instalaltion of the docker environment)
- linux-quantom-demo (the script that applies certificates)


1. Create two VM's in NREC or set up another client-server environment.

1.1 For a NREC you must make two instances

1.1.1 Instance 1: Ansible controller
- Source/OS: GOLD Rocky Linux 9
- Flavor: m1.small
- Networks: dualStack
- Security Group: SSH_and_ICMP
- Key Pair: Create Key Pair: SSH

1.1.2 Instance 2: Connector
- Source/OS: GOLD Rocky Linux 9
- Flavor: m1.large
- Networks: dualStack
- Security Group: SSH_and_ICMP
- Key Pair: Use same as Instance 1
   
1.2 Instance 1 is used for part 2-6 and Instance 2 is used for part 7


2. After booting up Instance 1, install necessary packages

  2.1 Div packages, run "sudo yum update"
  
  2.2 Epel, run "sudo yum install epel-release"

  2.3 Ansible, run "sudo yum install ansible"


3. Download all the necessary files from the git reposatory

4. Create a inventory file and add:

  4.1 Run: `vi linux_inventory.yml`

  4.2 Add this: 

    `linux_hosts:

       hosts:

         quantom:

           ansible_host: 158.39.77.74

           ansible_user: rocky

           ansible_ssh_private_key_file: /Users/YourUserName/.ssh/openstack`


5. Install docker

   5.1 Run: `ansible-playbook -i linux_inventory.yml linux-install-docker.yml`


6. Collect certificates and start the nginx 

   6.1 Run: `ansible-playbook -i linux_inventory.yml linux-quantom-demo.yml`
   NB! If you want to use other certificates, you need replace them in the quantum_demo folder,
   this is were the playbook collects them and put them into their place.


7. Test connectiong ssh rocky@instance_ip

   7.1 Run: `ssh rocky@instance_ip`


8. Run the TLS-handshake command (remeber to add own credentials (instance_ip, port and algorithm)

   8.1 Run: `docker run -it -v /root/server-pki/ca-cert.pem:/ca-cert.pem openquantumsafe/curl curl https://instance:port -vvi --curves kyber768 --cacert ./ca-cert.pem`

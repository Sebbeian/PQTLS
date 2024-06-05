# PQ TLS EXPERIMENTS

## THESIS EXPERIMENT

NB! To use NREC you must first apply for ann account through NREC.no

Files for this test:
None

<br><br>
**1. Create a VM in NREC or set up another environment on a self-chosen service**

- Build a Instance with the following credential
  - Source/OS: GOLD Ubuntu Linux 9
  - Flavor: m1.small
  - Networks: dualStack
  - Security Group: SSH_and_ICMP
  - Key Pair: Create Key Pair: SSH

<br><br>
**2. Install the dependencies**<br><br>
- Run `sudo apt update` to check that everything is up-to-date.

- This command installs development tools like Git, compiler tools, and necessary libraries.
  - Run `sudo apt -y install git build-essential perl cmake autoconf libtool zlib1g-dev` to install essential.

- These commands configure the environment, creating a workspace and directories for building software.
  - Run `export WORKSPACE=~/quantumsafe` to set a work directory.
  - Run `export BUILD_DIR=$WORKSPACE/build` to build the artifacts.
  - Run `mkdir -p $BUILD_DIR/lib64` to make the directory.
  - Run `ln -s $BUILD_DIR/lib64 $BUILD_DIR/lib`

<br><br>
**3. Install OpenSSL**

- Go to the workspace by running `cd $WORKSPACE`
- Clone the Git reposatory for OpenSSL by running `git clone https://github.com/openssl/openssl.git`
- Go to OpenSSL directory by `cd openssl` and run the following command:
 ```bash
./Configure \
    --prefix=$BUILD_DIR \
    no-ssl no-tls1 no-tls1_1 no-afalgeng \
    no-shared threads -lm`
```

- This builds OpenSSL with specific configurations to ensure compatibility with quantum-safe algorithms.
  - Run `make -j $(nproc)`
  - Run `make -j $(nproc) install_sw install_ssldirs`

<br><br>
**4. Install liboqs**

- Go back to the workspace `cd $WORKSPACE`<br>
- Clone the Git reposatory for LibOQS with `git clone https://github.com/open-quantum-safe/liboqs.git`.<br>
- Go into the LibOQS directory cd liboqs and build a "build"-directory with `mkdir build` and enter it `cd build`.<br>
- Make<br>
```bash
cmake \
  -DCMAKE_INSTALL_PREFIX=$BUILD_DIR \
  -DBUILD_SHARED_LIBS=ON \
  -DOQS_USE_OPENSSL=OFF \
  -DCMAKE_BUILD_TYPE=Release \
  -DOQS_BUILD_ONLY_LIB=ON \
  -DOQS_DIST_BUILD=ON \
  ..
```
- Compile the Software
  - Run `make -j $(nproc)`

- Install the Software
  - Run `make -j $(nproc) install`

<br><br>
**5. Install OQS provider for OpenSSL 3**

- Go back to the workspace `cd $WORKSPACE`

- Clone the Git reposatory for OQS provider for OpenSSL 3 with `git clone https://github.com/open-quantum-safe/oqs-provider.git`.

- Enter the OQS-provider `cd oqs-provider` and run:
```bash
liboqs_DIR=$BUILD_DIR cmake \
  -DCMAKE_INSTALL_PREFIX=$WORKSPACE/oqs-provider \
  -DOPENSSL_ROOT_DIR=$BUILD_DIR \
  -DCMAKE_BUILD_TYPE=Release \
  -S . \
  -B _build
cmake --build _build
```

- Now use this command to manually copy the files into the build dir; `cp _build/lib/* $BUILD_DIR/lib/`

- We need to edit the openssl config to use the OQS-provider. This is done by running:
```bash
sed -i "s/default = default_sect/default = default_sect\noqsprovider = oqsprovider_sect/g" $BUILD_DIR/ssl/openssl.cnf &&
sed -i "s/\[default_sect\]/\[default_sect\]\nactivate = 1\n\[oqsprovider_sect\]\nactivate = 1\n/g" $BUILD_DIR/ssl/openssl.cnf`
```

- Change environment variables for the oqsprovider, so that it is used when using OpenSSL. Run:
```bash
export OPENSSL_CONF=$BUILD_DIR/ssl/openssl.cnf
export OPENSSL_MODULES=$BUILD_DIR/lib
$BUILD_DIR/bin/openssl list -providers -verbose -provider oqsprovider
```

<br><br>
**6. Install and run cURL with quantum-safe algorithms**

- Go to the workspace by running `cd $WORKSPACE`

- Clone the Git reposatory for LibOQS with `git clone https://github.com/curl/curl.git`.

- Go to the Git directory `cd curl` and run:
```
autoreconf -fi
  ./configure \
    LIBS="-lssl -lcrypto -lz" \
    LDFLAGS="-Wl,-rpath,$BUILD_DIR/lib64 -L$BUILD_DIR/lib64 -Wl,-rpath,$BUILD_DIR/lib -L$BUILD_DIR/lib -Wl,-rpath,/lib64 -L/lib64 -Wl,-rpath,/lib -L/lib" \
    CFLAGS="-O3 -fPIC" \
    --prefix=$BUILD_DIR \
    --with-ssl=$BUILD_DIR \
    --with-zlib=/ \
    --enable-optimize --enable-libcurl-option --enable-libgcc --enable-shared \
    --enable-ldap=no --enable-ipv6 --enable-versioned-symbols \
    --disable-manual \
    --without-default-ssl-backend \
    --without-librtmp --without-libidn2 \
    --without-gnutls --without-mbedtls \
    --without-wolfssl --without-libpsl`
```
- Compile the Software
  - Run `make -j $(nproc)`

- Install the Compiled Software
  - Run `make -j $(nproc) install`

<br><br>
**7. Running the tests**
- Test Download of CA Certificate
  - Run 
```
$BUILD_DIR/bin/curl -vk https://test.openquantumsafe.org/CA.crt --output $BUILD_DIR/ca.cert`
```
- Test Quantum-Safe Secure Communication
  - Run (remeber to change algorithm for key exchange (algorithm) and signatures (xxxx) to match your test)
     -- Eks `mlkem768` against `htps://test.openquantumsafe.org:6137/` for a TLS-handshake with ML-KEM and ML-DSA
```
$BUILD_DIR/bin/curl -v --curves ke_algorithm --cacert $BUILD_DIR/ca.cert htps://test.openquantumsafe.org:xxxx/`
```

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

## DIGICERT EXPERIMENT

NB! 
- To use NREC you must first apply for ann account through NREC.no
- You need to use a variant of Rocky Linux 
- Have a security group that allows SSH-connectrivity (my NREC setup enables this)

Files for this test:
- quantum_demo (the certificate files)
- linux-install-docker (installaltion of the docker environment)
- linux-quantom-demo (the script that applies certificates)
<br><br>

**1. Create two VM's in NREC or set up another client-server environment.**
 -  For a NREC you must make two instances

- Instance 1: Ansible controller
  - Source/OS: GOLD Rocky Linux 9
  - Flavor: m1.small
  - Networks: dualStack
  - Security Group: SSH_and_ICMP
  - Key Pair: Create Key Pair: SSH

- Instance 2: Connector
  - Source/OS: GOLD Rocky Linux 9
  - Flavor: m1.large
  - Networks: dualStack
  - Security Group: SSH_and_ICMP
  - Key Pair: Use same as Instance 1
   
- Instance 1 is used for part 2-6 and Instance 2 is used for part 7

<br><br>
2. After booting up Instance 1, install necessary packages
<br><br>
  - Div packages, run `sudo yum update` to update all 
  - Epel, run `sudo yum install epel-release` to install
  - Ansible, run `sudo yum install ansible` to install

<br><br>
3. Download all the necessary files from the git reposatory (this reposatory)

<br><br>
4. Create a inventory file and add:
<br><br>
  - Run `vi linux_inventory.yml`

  - Add this to the file (remember to use own credentials): 
```
linux_hosts:<
  hosts:
    quantom:
        ansible_host: 158.39.77.74
        ansible_user: rocky
        ansible_ssh_private_key_file: /Users/YourUserName/.ssh/openstack`
```

<br><br>
5. Install docker from the playbook
   - Run: `ansible-playbook -i linux_inventory.yml linux-install-docker.yml`

<br><br>
6. Collect certificates and start nginx 
   - Run: `ansible-playbook -i linux_inventory.yml linux-quantom-demo.yml`
   NB! If you want to use other certificates, you need replace them in the quantum_demo folder,
   this is were the playbook collects them and put them into their place.

<br><br>
7. Test connectiong ssh rocky@instance_ip
   - Run: `ssh rocky@instance_ip` to test the SSH-connection to the other "part" (Instance 2)

<br><br>
8. Run the TLS-handshake command (remeber to add own credentials (instance_ip, port and algorithm)
   - Run: `docker run -it -v /root/server-pki/ca-cert.pem:/ca-cert.pem openquantumsafe/curl curl https://instance:port -vvi --curves kyber768 --cacert ./ca-cert.pem`

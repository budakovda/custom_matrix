# REAME.md

## on ansible host

```sh
# package
yum -q -y install epel-release >>/dev/null && wait; yum -q -y install ansible git sshpass 2 >>/dev/null

# ssh
net='192.168.100.0'
net_prefix=$(echo $net | awk -F. '{print $1"."$2"."$3"."}'); 
cat /dev/zero | ssh-keygen -q -N ""
for i in 117; do
    sshpass -p vagrant ssh root@"$net_prefix""$i" "if ! [ -d ~/.ssh ]; then mkdir ~/.ssh; fi";
    sshpass -p vagrant scp ~/.ssh/id_rsa.pub root@"$net_prefix""$i":~/.ssh/authorized_keys;
done

# repo
cd /vagrant
git clone https://github.com/spantaleev/matrix-docker-ansible-deploy.git 

# inventory
cd matrix-docker-ansible-deploy
mkdir inventory/host_vars/matrix.home.org
cp examples/hosts inventory/
cp examples/vars.yml inventory/host_vars/matrix.home.org/vars.yml

sed -i '/ansible_host/c matrix.home.org ansible_host=192.168.100.117 ansible_ssh_user=root'  inventory/hosts
sed -i '/matrix_domain: /c matrix_domain: home.org' roles/matrix-base/defaults/main.yml
sed -i '/matrix_domain: /c matrix_domain: home.org' inventory/host_vars/matrix.home.org/vars.yml
sed -i '/matrix_ssl_lets_encrypt_support_email: /c matrix_ssl_lets_encrypt_support_email: '\''example@mail.org'\''' inventory/host_vars/matrix.home.org/vars.yml
sed -i '/matrix_coturn_turn_static_auth_secret: /c matrix_coturn_turn_static_auth_secret: '\''12345678'\''' inventory/host_vars/matrix.home.org/vars.yml
sed -i '/matrix_synapse_macaroon_secret_key: /c matrix_synapse_macaroon_secret_key: '\''12345678'\''' inventory/host_vars/matrix.home.org/vars.yml
sed -i '/matrix_postgres_connection_password: /c matrix_postgres_connection_password: '\''12345678'\''' inventory/host_vars/matrix.home.org/vars.yml
echo 'matrix_ssl_retrieval_method: self-signed' >> inventory/host_vars/matrix.home.org/vars.yml
echo 'matrix_synapse_enable_registration: true'  >> inventory/host_vars/matrix.home.org/vars.yml 

# ansible
ansible-playbook -i inventory/hosts setup.yml --tags="setup-all" --tags="always"
```

## on matrix host

```sh
ssh <matrix-host>
systemctl start matrix-nginx-proxy.service
systemctl start matrix-client-element.service 
systemctl start matrix-ma1sd.service
```

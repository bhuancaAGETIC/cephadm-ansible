*** servidores *****
192.168.24.50 ceph0
192.168.24.51 ceph1
192.168.24.52 ceph2

******* para cada nodo ceph **********
sudo apt update
sudo apt-get install chrony -y

sudo sed 's/^pool/#&/' -i /etc/chrony/chrony.conf
sudo echo -e "pool 192.168.24.50 iburst \nallow 192.168.24.0/26" | sudo tee -a /etc/chrony/chrony.conf

sudo systemctl enable chrony
sudo systemctl restart chrony

sudo curl --silent --remote-name --location https://github.com/ceph/ceph/raw/quincy/src/cephadm/cephadm
sudo chmod +x cephadm
sudo ./cephadm install
sudo ./cephadm install  ceph-common

sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install -y docker-ce docker-ce-cli containerd.io





*****************solo para ceph0**************************

sudo ./cephadm bootstrap --mon-ip 192.168.24.50 --initial-dashboard-user "admin" --initial-dashboard-password "ceph" --dashboard-password-noupdate --cluster-network=192.168.24.0/26 --allow-fqdn-hostname

sudo ssh-copy-id -f -i /etc/ceph/ceph.pub root@ceph1
sudo ssh-copy-id -f -i /etc/ceph/ceph.pub root@ceph2


sudo ceph orch host add ceph1 192.168.24.51
sudo ceph orch host add ceph2 192.168.24.52


sudo ceph orch apply mon --placement="ceph0,ceph1,ceph2"
sudo ceph orch apply mgr --placement="ceph0,ceph1,ceph2"


sudo ceph orch host label add ceph0 osd-node
sudo ceph orch host label add ceph1 osd-node
sudo ceph orch host label add ceph2 osd-node


sudo ceph orch host label add ceph0 mon
sudo ceph orch host label add ceph1 mon
sudo ceph orch host label add ceph2 mon


sudo ceph orch host label add ceph0 mgr
sudo ceph orch host label add ceph1 mgr
sudo ceph orch host label add ceph2 mgr


sudo ceph orch apply osd --all-available-devices
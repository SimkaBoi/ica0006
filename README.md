# ICA0006

Grupp 9

Grupi liikmed: Siim Ristjõe, Georg Veevo, Johann Buschmann

## Ülesanne

Igale grupile eraldatakse 3 füüsilist serverit ning 1 virtuaalne server.

Kasutades HP serverite ILO kaughaldusliidest seadistada laboris riistvara peale RAID grupp operatsioonisüsteemi jaoks, installida endale sobiv serveri operatsioonisüsteem (3 füüsilist serverit)

Serveritele paigaldada horisontaalselt skaleeruv tarkvara (CEPH, ScaleIO, Microsoft Storagespaces, MinIO, Lustre, LizardFS jms)

Installeerida terraformi kasutades virtuaalserver ülikooli laborisse.

Kasutajanimed ning paroolid on kõik samad nii ILO kui virtuaalserverisse sisse logimisel.

Provisioneerida andmesalvestus pind virtuaalserverile üle IP võrgu (kas plokina, jagatud failisüsteemina või objektide salvestuskohana). Et muuta ülesanne realistlikumaks, panna seda pinda kasutama kas andmebaas või mingi veebirakendus. Veenduda süsteemi tõrkekindluses, lülitades välja suvaline füüsiline server.

### Ülesannete jaotus

| Ülesanne                                                | Määratud tudeng(id)                         |
| ------------------------------------------------------- | ------------------------------------------- |
| Raidi seadistamine                                      | Siim Ristjõe, Georg Veevo, Johann Buschmann |
| Operatsioonisüsteemi installimine                       | Siim Ristjõe, Georg Veevo, Johann Buschmann |
| SSH seadistamine                                        | Siim Ristjõe, Georg Veevo, Johann Buschmann |
| Cephi seadistamine                                      | Siim Ristjõe                                |
| Virtuaalserveri installeerimine                         | Georg Veevo, Johann Buschmann               |
| Minecraft serveri seadistamine                          | Georg Veevo                                 |
| VM-i cephi külge mountimine                             | Georg Veevo                                 |
| Minecrafti serveri failide lisamine Cephi               | Siim Ristjõe, Georg Veevo                   |
| Minecrafti serveri teenus ja enable boot peale panemine | Georg Veevo                                 |
| Tõrkekindluse tagamine                                  | Siim Ristjõe, Georg Veevo                   |
| Powerpoint slaidid                                      | Johann Buschmann                            |

## Serverid ning kasutajaandmed

### Serverid

| Serveri nimi | Serveri IP     | ILO IP        |
| ------------ | -------------- | ------------- |
| server1      | 192.168.185.21 | 192.168.185.1 |
| server2      | 192.168.185.22 | 192.168.185.2 |
| server3      | 192.168.185.23 | 192.168.185.3 |
| vm           | 192.168.180.26 |               |

### Kasutajaandmed

Lihtsuse mõttes otsustasime kasutada kõikjal sama kasutajat ning parooli.

| Nimi    | Parool      |
| ------- | ----------- |
| student | student1234 |

## RAIDi seadistamine

Selleks, et ILO kaughaldusliidesele ligi pääseda, tuleb kasutada Microsoft Edge'i ning Internet Exploreri compatibility mode'i. Samuti peab olema ühendatud kooli sisevõrku.

Seejärel tuleb serverid käima panna ja kui pilt tuleb ette, vajutada `F8` nuppu. Avaneb menüü, kus tuleb taas vajutada `F8` nuppu.

![alt text](/images/image.png)

Pärast seda avaneb uus menüü, kust tuleb lahkuda.

![alt text](/images/image-1.png)

Lähtuvalt ülesande tekstist lõime operatsioonisüsteemile eraldi `RAID 1+0` grupi, mis koosneb kahest 300GB kettast. Ülejäänud 900GB kettad panime eraldi `RAID 0` gruppidesse.

![alt text](/images/image-2.png)

## Operatsioonisüsteemi installimine

Otsustasime operatsioonisüsteemiks valida [Ubuntu server 24.04](https://ubuntu.com/download/server).

Installi käigus määrasime serveritele staatilise IP koos subnet'i ja default gateway'ga.

| Serveri nimi | Serveri IP     |
| ------------ | -------------- |
| server1      | 192.168.185.21 |
| server2      | 192.168.185.22 |
| server3      | 192.168.185.23 |

Subnet: `255.255.252.0`
Default gateway: `192.168.187.254`

Kasutaja loomisel määrasime kasutajanimeks `student` ning parooliks `student1234`.

Pärast operatsioonisüsteemi installi uuendasime servereid.

```bash
sudo apt update && sudo apt upgrade
```

## SSH seadistamine

Pärast operatsioonisüsteemi installi lisasime kasutajasse `student` grupi liikmete avalikud SSH võtmed.

```bash
curl -s https://github.com/{simkaboi,jbuschtal,geveev}.keys > ~/.ssh/authorized_keys
```

Seejärel veendusime, et ligipääs serveritesse läbi SSH töötab.

```bash
ssh student@192.168.185.2x
```

## Cephi seadistamine

Otsustasime andmete hoiustamiseks kasutada [Cephi](https://ceph.io/en/) ning selle ametlikku tööriista [Cephadm](https://docs.ceph.com/en/latest/cephadm/).

### Cephadm'i install

Cephadm'i installimiseks kasutatud dokumentatsioon asub [siin](https://docs.ceph.com/en/latest/cephadm/install/#install-cephadm).

Alustuseks tuli välja valida server, kuhu installime Cephadm'i. See server hakkab Cephi manageerima. Valisime selleks serveriks `server1`.

Cephadm'i installimiseks kasutasime järgnevat käsku:

```bash
sudo apt install -y cephadm
```

Pärast installi veendusime et Cephadm töötab:

```bash
cephadm
```

### Ceph klastri loomine

Ceph klastri loomiseks kasutatud dokumentatsioon asub [siin](https://docs.ceph.com/en/latest/cephadm/install/#bootstrap-a-new-cluster).

Olles `server1` terminalis, jooksutasime järgnevat käsku:

```bash
sudo cephadm bootstrap --mon-ip 192.168.185.21
```

Pärast käsu käivitamist kuvatakse terminalis linki https://192.168.185.21:8443/, kust saab ligi Ceph'i dashboardile.

### Ceph CLI käskude lubamine

Ceph CLI käskude lubamiseks kasutatud dokumentatsioon asub [siin](https://docs.ceph.com/en/latest/cephadm/install/#enable-ceph-cli).

Praegu on võimalik Ceph'i käske jooksutada ainult Cephadm'i loodud konteinerite sees. Selleks, et käske saaks väljaspool kasutada, pidime installima `ceph-common` paketi.

```bash
cephadm add-repo --release squid
cephadm install ceph-common
```

Kontrolliks kasutasime järgnevat käsku:

```bash
ceph -v
```

### Hostide lisamine Cephi klastrisse

Hostide lisamiseks kasutatud dokumentatsioon asub [siin](https://docs.ceph.com/en/latest/cephadm/host-management/#adding-hosts).

Hetkel koosneb Ceph'i klaster ainult ühest serverist (`server1`), kuid serverid `server2` ja `server3` tuleb ka lisada.

Enne serverite lisamist Cephi klastrisse peavad serveritel olema installitud järgnevad [paketid](https://docs.ceph.com/en/latest/cephadm/install/#requirements):

- Python 3
- Systemd
- Podman/Docker
- Chrony
- LVM2

Kuna enamik pakette tuli koos Ubuntu installiga kaasa, oli meil ainult [dockerit](https://docs.docker.com/engine/install/ubuntu/) vaja installida.

Selleks jooksutasime järgnevaid käske:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Pärast installi veendusime, et Docker töötab:

```bash
sudo docker run hello-world
```

Nüüd kui eeldused on täidetud, on vaja `server1` Ceph'i avalik võti lisada teistele serveritele.

Kuna `ssh-copy-id` käsk nõuab parooliga sisselogimist, on vaja teistes serverites parooliga sisselogimine ajutiselt lubada. Lisaks, kuna avalik võti peab olema kasutaja `root` kaustas, lubame ajutiselt `root` kasutajasse sisselogimise parooliga.

Selleks lõime uue faili `/etc/ssh/sshd_config.d/00-custom.conf`

```bash
sudo vim /etc/ssh/sshd_config.d/00-custom.conf
```

ning lisasime sinna järgnevad read:

```
PasswordAuthentication yes
PermitRootLogin yes
```

Seejärel tegime SSH teenusele restarti, et muudatused hakkaksid tööle

```bash
sudo systemctl restart ssh
```

ning veendusime, et muudatused oleksid paigaldatud:

```bash
sudo sshd -T | grep -E 'passwordauthentication|permitrootlogin'
```

Pärast SSH konfiguratsiooni muutmist saime lisada Ceph'i avaliku võtme:

```bash
ssh-copy-id -f -i /etc/ceph/ceph.pub root@192.168.185.22
ssh-copy-id -f -i /etc/ceph/ceph.pub root@192.168.185.23
```

Pärast avaliku võtme lisamist saime failis `/etc/ssh/sshd_config.d/00-custom.conf` read välja kommenteerida, SSH teenusele restarti teha ning veenduda, et muudatused läksid täide.

Viimase sammuna lisasime serverid `server2` ja `server3` Ceph'i klastrisse ning veendusime, et serverid said edukalt lisatud.

```bash
sudo ceph orch host add server2 192.168.185.22
sudo ceph orch host add server3 192.168.185.23
sudo ceph orch host ls --detail
```

### Ketaste lisamine ja volüümi loomine

Esmalt veendusime, et igal serveril oleksid 2x900GB kettad tühjad:

```bash
sudo wipefs -a /dev/sdb
sudo wipefs -a /dev/sdc
```

Seejärel veendusime, et Ceph näeks kõiki saadaval kettaid:

```bash
sudo ceph orch device ls
```

Siis lisasime kõik kettad Cephi klastrisse:

```bash
sudo ceph orch apply osd --all-available-devices
```

Viimasena lõime volüümi ning veendusime, et kõik töötaks:

```bash
sudo ceph fs volume create cephfilesystem
sudo ceph -s
```

```
student@server1:~# sudo ceph -s
  cluster:
    id:     f8f1d956-1df3-11f0-99e7-3b8bca07a27c
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum server1,server2,server3 (age 3h)
    mgr: server1.jadojs(active, since 5h), standbys: server2.esbgns
    mds: 1/1 daemons up, 2 standby
    osd: 6 osds: 6 up (since 26m), 6 in (since 27m)

  data:
    volumes: 1/1 healthy
    pools:   2 pools, 272 pgs
    objects: 22 objects, 2.3 KiB
    usage:   1.7 GiB used, 4.9 TiB / 4.9 TiB avail
    pgs:     272 active+clean
```

## Virtuaalserveri installeerimine

### Terraformi installimine Windowsile

Terraformi installimiseks Windowsile laadisime selle HashiCorpi ametlikult veebilehelt.

1. Laadisime alla `terraform_1.11.4_windows_amd64.zip\` ja pakkisime kausta `C:\Terraform\`.
2. Lisasime `C:\Terraform\` keskkonnamuutujasse `Path\`.
3. Pärast PowerShelli taaskäivitamist kontrollisime:

   ```powershell
   terraform --version
   ```

   ```
   Terraform v1.11.4
   on windows_amd64
   ```

### Projekti seadistamine

Lõime Terraformi projekti jaoks kausta ning faili `main.tf` virtuaalmasina loomise haldamiseks.

```powershell
mkdir $env:USERPROFILE\TerraformProjects\ica0006
cd $env:USERPROFILE\TerraformProjects\ica0006
New-Item main.tf
```

Avasime `main.tf` Notepadis ja lisasime järgiva skripti, mis loob virtuaalmasina vSphere keskkonnas:

```hcl
provider "vsphere" {
  user                 = "UNI-ID@intra.ttu.ee"
  password             = "PASSWORD"
  vsphere_server       = "192.168.184.253"
  allow_unverified_ssl = true
}

data "vsphere_datacenter" "dc" {
  name = "Datacenter_407"
}

data "vsphere_resource_pool" "pool" {
  name          = "HPE BladeSystem Gen8 - Rack 3/Resources"
  datacenter_id = data.vsphere_datacenter.dc.id
}

data "vsphere_datastore" "datastore" {
  name          = "Hitachi_LUN1"
  datacenter_id = data.vsphere_datacenter.dc.id
}

data "vsphere_network" "network" {
  name          = "VM Network"
  datacenter_id = data.vsphere_datacenter.dc.id
}

data "vsphere_virtual_machine" "template" {
  name          = "Ubuntu 22.04 vanilla template"
  datacenter_id = data.vsphere_datacenter.dc.id
}

resource "vsphere_virtual_machine" "demo" {
  name             = "grupp_9"
  num_cpus         = 2
  memory           = 4096
  datastore_id     = data.vsphere_datastore.datastore.id
  resource_pool_id = data.vsphere_resource_pool.pool.id
  guest_id         = data.vsphere_virtual_machine.template.guest_id
  scsi_type        = data.vsphere_virtual_machine.template.scsi_type
  folder           = "ICA0006"

  network_interface {
    network_id = data.vsphere_network.network.id
  }

  disk {
    label              = "vm-one.vmdk"
    size               = 50
    eagerly_scrub      = false
    thin_provisioned   = true
  }

  clone {
    template_uuid = data.vsphere_virtual_machine.template.id
  }
}
```

### Terraformi initsialiseerimine ja virtuaalmasina loomine

Initsialiseerisime Terraformi:

```powershell
terraform init
```

```text
- Installed hashicorp/vsphere v2.12.0 (signed by HashiCorp)
Terraform has been successfully initialized!
```

Kontrollisime plaani:

```powershell
terraform plan
```

```text
Plan: 1 to add, 0 to change, 0 to destroy.
```

Rakendasime konfiguratsiooni:

```powershell
terraform apply --auto-approve
```

```text
vsphere_virtual_machine.demo: Creation complete after 57s [id=42166ee0-72ed-e251-4ca3-ad92d1631f9a]
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

Kontrollisime virtuaalmasina detaile:

```powershell
terraform state show vsphere_virtual_machine.demo
```

````text
# vsphere_virtual_machine.demo:
resource "vsphere_virtual_machine" "demo" {
    default_ip_address = "192.168.180.26"
    name               = "grupp_9"
    memory             = 4096
    num_cpus           = 2
}
```EOF

echo "Markdown file 'ICA0006.md' has been generated."
````

Siit näeme virtuaalmasina automaatselt määratud IP aadressi ja meie pandud nime "grupp_9".

## Minecrafti serveri seadistamine virtuaalmasinas

Pärast virtuaalmasina loomist Terraformiga seadistasime Minecrafti serveri virtuaalmasinas (IP: `192.168.180.26`), et täita ülesande nõuet panna andmesalvestusala kasutama veebirakendus.
Minecrafti serveri jaoks kasutatud dokumentatsioon asub [siin](https://idroot.us/install-minecraft-server-ubuntu-24-04/).

### Kasutaja loomine

Minecrafti serveri turvaliseks haldamiseks on soovitatav luua eraldi kasutajakonto. See aitab hoida Minecrafti protsessi isoleerituna ja vähendada turvariske.

Logisime sisse serverisse ja tegime uus kasutaja nimega minecraft:
`sudo adduser minecraft`

Logisime kasutajakontole sisse:
`su - minecraft`

### Minecrafti serveri allalaadimine

Logisime virtuaalmasinasse kasutajaga `minecraft` ja lõime kausta `minecraft_server` serverifailide jaoks. Laadisime alla PaperMC serveri JAR-faili versioonile 1.21.4:

```bash
mkdir minecraft_server
cd minecraft_server
wget https://api.papermc.io/v2/projects/paper/versions/1.21.4/builds/226/downloads/paper-1.21.4-226.jar
```

### Minecrafti serveri käivitamine

Käivitasime serveri:

```bash
java -Xmx1024M -Xms1024M -jar paper-1.21.4-226.jar nogui
```

Server käivitus, kuid peatus, kuna pidime nõustuma EULA-ga:

Väljund (lühendatud):

```
[18:49:09 WARN]: Failed to load eula.txt
[18:49:09 INFO]: You need to agree to the EULA in order to run the server. Go to eula.txt for more info.
```

### EULA ja serveri seadete muutmine

Muutsime `eula.txt` faili, et nõustuda EULA-ga, ja kohandasime `server.properties` faili vastavalt vajadusele (nt mängurežiim, port jne):

```bash
nano eula.txt
nano server.properties
```

Failis `eula.txt` asendasime rea `eula=false` reaga `eula=true`.

![alt text](/images/image-3.png)

### Serveri uuesti käivitamine

Käivitasime serveri uuesti:

```bash
java -Xmx1024M -Xms1024M -jar paper-1.21.4-226.jar nogui
```

Seekord server käivitus edukalt ning kõik vajalikud failid said loodud.

## VM-i cephi külge mountimine

Selleks et VM Cephiga ühendada on vaja installida VM-ile ceph-common

```bash
sudo apt update
sudo apt install -y ceph-common
```

Kopeerisime Server-1lt cephi confi ja ceph admin keyringi

```
root@server1:~# scp /etc/ceph/ceph.conf student@192.168.180.26:/tmp/ceph.conf
student@192.168.180.26's password:
ceph.conf                                                                             100%  283   363.0KB/s   00:00
minecraft@lab:/home/student$ sudo mv /tmp/ceph.conf /etc/ceph/

root@server1:~# scp /etc/ceph/ceph.client.admin.keyring student@192.168.180.26:/tmp/ceph.client.admin.keyring
student@192.168.180.26's password:
ceph.client.admin.keyring                                                             100%  151   207.6KB/s   00:00
minecraft@lab:~$ sudo mv /tmp/ceph.client.admin.keyring /etc/ceph/ceph.client.admin.keyring
```

Cephi mountimiseks:

```bash
student@192.168.180.26:~$ sudo mount -t ceph 192.168.185.21,192.168.185.22,192.168.185.23:/ /mnt/cephfs -o name=admin,secret=<key>
```

Lisaks lisasime /etc/fstab faili automaatse mount'imise taaskäivitamisel.

```bash
student@192.168.180.26:~$ echo "192.168.185.21,192.168.185.22,192.168.185.23:/ /mnt/cephfs ceph name=admin,secret=<key> 0 0" | sudo tee -a /etc/fstab
```

Märkus: <key> väärtus võeti failist /etc/ceph/ceph.client.admin.keyring.

## Minecrafti serveri failide lisamine Cephi

Minecraftis on kolme tüüpi mappe World, Nether ja End. Iga map on vaja lisada cephi ja nendega kaasa tuleks ka lisada Server Log file.

Failide liigutamine Cephi

```bash
sudo mv ~/minecraft_server/world /mnt/cephfs/world
sudo mv ~/minecraft_server/world_nether /mnt/cephfs/world_nether
sudo mv ~/minecraft_server/world_the_end /mnt/cephfs/world_the_end
sudo mv ~/minecraft_server/logs /mnt/cephfs/logs
```

Synlinkide loomine:

```bash
ln -s /mnt/cephfs/world ~/minecraft_server/world
ln -s /mnt/cephfs/world_nether ~/minecraft_server/world_nether
ln -s /mnt/cephfs/world_the_end ~/minecraft_server/world_the_end
ln -s /mnt/cephfs/logs ~/minecraft_server/logs
```

Dashboardis failide olemasolu kinnitamine:

![alt text](/images/image-4.png)

## Minecrafti serveri teenus ja enable boot peale panemine

Kui VM-ile restart teha siis oleks vaja et minecrafti server käivituks automaatselt.

Kuna Minecraft user ja directory oli valmis siis oli vaja luua uus /etc/systemd/system/minecraft.service

Minecraft.service:
![alt text](/images/image-5.png)

System daemonite reloadimine ja boot enable

```bash
sudo systemctl daemon-reload
sudo systemctl enable minecraft.service
```

## Tõrkekindluse tagamine

Selleks, et kindel olla Minecrafti serveri ja andmete liikumine toimib peale SRV1/2/3 sulgemisel peatasime ükshaaval SRV1/2/3

Minecraft server toimimas peale, SRV2 sulgemist:
![alt text](/images/image-6.png)

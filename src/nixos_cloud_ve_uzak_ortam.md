# NixOS'u Cloud ve Uzak Ortamlara Deploy Etmek

İlk yazımızda bahsettiğim `nix-gernerator` aracından tekrar bahsederek konuya girmek istiyorum.

Tek bir konfigürasyonu kullanarak [nix-generator](https://github.com/nix-community/nixos-generators) aracı ile alttaki bütün ortamlara çıktı üretebilirsiniz.

| format               | description                                                                                                                             |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| amazon               | Amazon EC2 image                                                                                                                        |
| azure                | Microsoft azure image (Generation 1 / VHD)                                                                                              |
| cloudstack           | qcow2 image for cloudstack                                                                                                              |
| do                   | Digital Ocean image                                                                                                                     |
| docker               | Docker image (uses systemd to run, probably only works in podman)                                                                       |
| gce                  | Google Compute image                                                                                                                    |
| hyperv               | Hyper-V Image (Generation 2 / VHDX)                                                                                                     |
| install-iso          | Installer ISO                                                                                                                           |
| install-iso-hyperv   | Installer ISO with enabled hyper-v support                                                                                              |
| iso                  | ISO                                                                                                                                     |
| kexec                | kexec tarball (extract to / and run /kexec_nixos)                                                                                       |
| kexec-bundle         | same as before, but it's just an executable                                                                                             |
| kubevirt             | KubeVirt image                                                                                                                          |
| linode               | Linode image                                                                                                                            |
| lxc                  | create a tarball which is importable as an lxc container, use together with lxc-metadata                                                |
| lxc-metadata         | the necessary metadata for the lxc image to start, usage: `lxc image import $(nixos-generate -f lxc-metadata) $(nixos-generate -f lxc)` |
| openstack            | qcow2 image for openstack                                                                                                               |
| proxmox              | [VMA](https://pve.proxmox.com/wiki/VMA) file for proxmox                                                                                |
| proxmox-lxc          | LXC template for proxmox                                                                                                                |
| qcow                 | qcow2 image                                                                                                                             |
| raw                  | raw image with bios/mbr. for physical hardware, see the 'raw and raw-efi' section                                                       |
| raw-efi              | raw image with efi support. for physical hardware, see the 'raw and raw-efi' section                                                    |
| sd-aarch64           | Like sd-aarch64-installer, but does not use default installer image config.                                                             |
| sd-aarch64-installer | create an installer sd card for aarch64. For cross compiling use `--system aarch64-linux` and read the cross-compile section.           |
| vagrant-virtualbox   | VirtualBox image for [Vagrant](https://www.vagrantup.com/)                                                                              |
| virtualbox           | virtualbox VM                                                                                                                           |
| vm                   | only used as a qemu-kvm runner                                                                                                          |
| vm-bootloader        | same as vm, but uses a real bootloader instead of netbooting                                                                            |
| vm-nogui             | same as vm, but without a GUI                                                                                                           |
| vmware               | VMWare image (VMDK)                                                                                                                     |

[^1]

[^1]: [Tablo Kaynağı](https://github.com/nix-community/nixos-generators)

Sayfasında detaylarına bakabilirsiniz. Uygulama olarak Nix paket yöneticisi ile kurulum yapıp kullanılabiliyor veya flake dosyasına konfigürasyon olarak da girilebiliyor. Yani illa NixOS sahibi olmak zorunda değiliz. Yani örneğin bir Ubuntu üzerinde Nix paket yöneticisini kurup bu aracı kullanabiliriz.

Elimizdeki Konfigürasyon üzerinden bu aracı kullanarak ISO, Docker Image ve VM oluşturmak çok vakit alcaktır. Bu nedenle Github sayfamda önceki yazılarımızda kullandığımız repo'da ilk oluşturduğumuz base konfigürasyonu kullanarak uygulamaları yapacağız.

Bunun için alttaki komutla base brach'i klonlayalım.

```bash
git clone -b base https://github.com/muratcabuk/nixos-sample-dotfiles.git
```

## NixOS-Generator ile ISO, Docker Image ve VM oluşturma

**ISO Oluşturmak**

- `Configuration.nix` dosyasında `boot.loader.grub.enable = true;` satırını `boot.loader.grub.enable = pkgs.lib.mkForce true;` olarak değiştiriyoruz.
- `Configuration.nix` dosyasında ` networking.networkmanager.enable = true;` satırını ` networking.networkmanager.enable = false;` olarak değiştiriyoruz.
- Daha sonra `nix profile install github:nix-community/nixos-generators` komutuyla kurulumu yapıyoruz. Son olarak `nixos-generate -f install-iso -c configuration.nix` komutu ile ISO dosyamızı oluşturuyoruz.

Bu biraz vakit alcaktır. İşlem bittiğinde `/nix/store/4sqxkbx1im3wsrdfrzxcyklkaynq3mpy-nixos-23.11pre536306.12bdeb01ff9e-x86_64-linux.iso/iso/nixos-23.11pre536306.12bdeb01ff9e-x86_64-linux.iso` adresine benzer şekilde ISO dosyasının adresini görebilirsiniz.

**Ddocker Image Oluşturmak**

Mesela docker image oluşturmak için `nixos-generate -f docker -c configuration.nix` komutunu kullanmak yeterli olacaktır.

komutu çalıştırdığınızda `/nix/store/5bmzn6rrjqjb3j1vn7h09vplfjn960r2-tarball/tarball/nixos-system-x86_64-linux.tar.xz` adresine benzer şekilde docker image dosyasının adresini görebilirsiniz. Bu paketi load etmek için alttaki komutu çalıştırabilirsiniz.

```bash
docker import  /nix/store/5bmzn6rrjqjb3j1vn7h09vplfjn960r2-tarball/tarball/nixos-system-x86_64-linux.tar.xz

sha256:31f06f10c13b90698c2764c16f0b2a9c50d52113f3b8ddf9771c3c4c1070dcfe

docker image ls
REPOSITORY                                                          TAG                         IMAGE ID       CREATED         SIZE
<none>                                                              <none>                      31f06f10c13b   8 seconds ago   1.07GB
<none>                                                              <none>                      925f5d1664a6   4 days ago      1.11GB
<none>                                                              <none>                      bb873de60531   4 days ago      3.02GB
<none>                                                              <none>                      1b6b71ea5c5c   4 days ago      966MB

```

- Docker için doğrudan Nix paket yöneticisinde de bir çok paket var. Kendi [resmi developer sayfasında](https://nix.dev/tutorials/nixos/building-and-running-docker-images.html) da konu ilgili bir başlık var.

- Nix paket yöneticisi içinde de docker ile ilgili bir çok örnek bulabilirsiniz. Örneğin [şu sayfada](https://github.com/NixOS/nixpkgs/blob/master/pkgs/build-support/docker/examples.nix) aklınıza gelebilecek hemen hemen bir çok örnek var.

- Bir de [şu harika örneği](https://fasterthanli.me/series/building-a-rust-service-with-nix/part-11) inceleyebilirsiniz. Baştan sona bir uygulamanın dev ortamının ve paketin oluşturulması ve uygulamanın docker image'ı haline getirilmesi sürecinin tamamını Nix ile anlatıyor.

- Nixpkgs içinde Docker image'larını oluştururken çok daha detaylı çalışabileceğiniz [dockerTools](https://nixos.org/manual/nixpkgs/stable/#sec-pkgs-dockerTools) adında ir paket bulunuyor. Buna da bakmanızı tavsiye ederim.

- Diğer bir araç da [Arion](https://docs.hercules-ci.com/arion/). Bu aracı daha doğrusu paketi kullanarak Docker Compose'daki gibi aynı anda bir den fazla uygulamayı hem container yapabilir hem de çalıştırabilirsiniz.

**VM Oluşturmak**

Virtualbox virtual Machine oluşturmak için öncelikle `configuration.nix` dosyasındaki disk bilgisini açıklama satırına alarak devre dışı bırakıyoruz.

```nix
  # fileSystems."/" =
  #   { device = "/dev/disk/by-uuid/5b06bc26-f363-4810-bc0d-a44a9c7cdc12";
  #     fsType = "ext4";
  #   };
```

Daha sonra `nixos-generate -f virtualbox -c configuration.nix` komutunu kullanarak VirtualBox için sanal bir makine oluşturmuş olacağız.

İşlem bittiğinde `/nix/store/7qy9wvm93p4ycwq2rxf6184xj8bna32c-nixos-ova-23.11pre536306.12bdeb01ff9e-x86_64-linux/nixos-23.11pre536306.12bdeb01ff9e-x86_64-linux.ova` şeklinde bir dizinde VirtualBox image'ımızın oluştuğunu görebiliriz.

## NixOS'u Uzak Sistemlere ve Cloud'a Kurmak

Burada karşımıza alttaki araçlar çıkıyor. Bir iki araç daha var ama ya geliştirmeleri durmuş ya da çok popüler değil o yüzden listeye eklemedim.

- [NixOS Everywhere](https://github.com/nix-community/nixos-anywhere): Yıldız Sayısı 950. Temel amacı SSH ile bağlanılabilen bir sisteme elinizdeki Flake ile kurulum yapmak. Nix Everywhere'de cloud kısmı yok ancak [Disco](https://github.com/nix-community/disko) eklentisi sayesinde disk formatlama ve biçimlendirme işlerini de yapabiliyor.
  - Detaylar için [resmi dokümanına](https://github.com/nix-community/nixos-anywhere/blob/main/docs/howtos/INDEX.md) bakabilirsiniz.
  - **Yetenekleri**
    - Disk formatlama ve bölümlendirme
    - NixOS'un kurulumu ve konfigürasyonu
    - Uygulama kurabilir
    - Ayrıca ekstra dosya konulacaksa onları ekleyebilir
  - **Nasıl Çalışır?**
    - SSH ile bağlanılabiliyor olması lazım.
    - SSh ile ayarları sayfasında bulabilirsiniz.
    - Karşı makinede NixOS kurulu olmalı yada installer çalışıyor olmalı. Eğer o yoksa kexec ile makine boot edilmiş olmalı.
    - şu komutla sisteme kurulum yapılır `nix run github:nix-community/nixos-anywhere -- --flake '.#myconfig' nixos@remote-host-ip-address`
- [deploy-rs](https://github.com/serokell/deploy-rs/): Yıldız Sayısı (1100)
  - **Yetenekleri**
    - Birden fazla profili deploy edebiliyor
    - Rust diliyle yazılmış çok hızlı bir araç
    - Kurulum işlemleri ters giderse sistemi temiz bir şekilde rollback yapabiliyor.
    - Temel amacı bir flake üzerideki profili alıp tanımlanan makine (node) üzerine kurmaktır.
    - Blok sitesinden baştan sona [bir örnek](https://serokell.io/blog/deploy-rs). Diğer bir örnek Github sayfasındaki tam sistem kurulumu gösteren [örnek](https://github.com/serokell/deploy-rs/tree/master/examples/system).
  - **Nasıl Çalışır**
    - Flake içinde node (makine) tanımı yapılır (ip, ssh port, kullanıcı adı, parola, ssh key, vb.)
    - daha sonra deploy-rs cli kullanılarak `deploy-rs --flake ./flake.nix --profile defaultPackage --node myNode1 --node myNode2` komutu ile kurulum yapılır.
  - [colmena](https://github.com/zhaofengli/colmena): Yıldız sayısı 930. Rust diliyle yazılmış diğer bir deployment aracı.
    - **Yetenekleri**
      - Node'lara tag vererek gruplandırmak mümkün.
      - Nixops network konfigurayonu ile uyumlu değişiklik yapmadan kullanılabilir.
      - Local makine deployment'ı da yapabiliyor.
      - Paralel deployment yapabiliyor. Zaten Rust ile yazılı olduğu için de hızlı.
      - Cloud deployment yeteneği yok. Sadece local ve uzak makinelere kurulum yapabiliyor.
      - **Kullanımı** Nixops'a çok benziyor. YAni bir flake deployment'ı yok daha çok sunucu kurulumlar için uygun görünüyor.
- [Morph](https://github.com/DBCDK/morph): Yıldız Sayısı 700
  - Temel amacı birden fazla host üzerinde NixOS'u yönetmek. Kurulum yapıp health check yapabiliyor.
  - Daha önce Ansible, Chef veya Puppet gibi araçlar kullandıysanız tam olarak onların yaptığı için NixOS ve Nix paket yöneticisini kullanarak yapıyor.
  - Çok basit bir kullanımı var. Bir nix dosyasına makinelerin bilgilerini yazdıktan morph komutu ile elinizdeki konfigürasyonu toplu halde makinelerinize uygulayabiliyorsunuz.
- [Nixops](https://github.com/NixOS/nixops): Yıldız Sayısı 1700. Araçlardan en dikkat çekeni Nixops. Temelde Nixops bir flake'i alıp onu belirtilen makineye kurmaya odaklanmıyor. Bunu Nix Everywhere veya Deploy-rs ile yapabiliriz. Nixops amacı çok farklı ortamlardaki makinelere bağlanmak ve onlara Nixops konfigürasyonunda belirttiğimiz paketleri ve uygulamaları kurmaktır. Toplu halde paket kurmak, toplu halde sistemleri destroy etmek gibi işlemleri yapabilir. Yani daha çok sunucu işlemleri ve konfigürasyonu için kullanılır.

- Nixops'un yeni versiyonu Nixops 2.0'da bazı değişikliliklere gidilmiş. Yeni versiyonla ilgili olarak da elimizdeki tek kaynak da [resmi dokümanlar](https://nixops.readthedocs.io/en/latest/introduction.html). Ancak bu versiyondaki bazı yapısal problemlerden dolayı da Rust diliyle yeniden yazmaya karar verilmiş. Bu yeni versiyonu da tamamen yeni bir Github hesabından devam ettiriyorlar. [Şu linkten](https://github.com/nixops4/nixops4) takip edebilirsiniz. Mevzu halen [şu issue'da](https://github.com/NixOS/nixops/issues/1574) tartışılıyor.
  - Elimizdeki çalışan en temiz versiyon olarak 1.7'nin [resmi dokümanlarına da şu linkten](https://releases.nixos.org/nixops/nixops-1.7/manual/manual.html) erişebilirsiniz.

  - Konu ile ilgili Türkçe olarak bulabileceğiniz belki tek içerik [şu Youtube videosu](https://www.youtube.com/watch?v=VIBPfXmkvr8&t=4309s) olacaktır. Bunu da izlemenizi tavsiye ederim. Çünkü videonun yarısında Nix paket yöneticisi ve NixOS hakkında çok güzel bilgiler veriliyor.

  - Baştan sona tüm detaylarıyla Nixops'un anlatıldığı [bir makale](https://www.thedroneely.com/posts/nixops-towards-the-final-frontier/).

  - **Yetenekleri**
    - Local veya uzak makinelere kurulum yapabilirsiniz.
    - Hetzner fiziksel makinelere kurulum yapabilirsiniz.
    - Libvirt (Qemu) üzerinde VM oluşturabilirsiniz.
    - VirtualBox üzerinde çalışan bir NixOS üzerinde kurulum ve değişiklik yapabilirsiniz. Ancak buraya dikkat çekmek istiyorum Nixops Virtualbox üzerinden makine oluşturmaz sadece vorolan makşneye bağlanablir.
    - AWS (Amazon Web Services), GCE (Google Compute Engine), Azure (Microsoft Azure), Digital Ocean gibi cloud ortamlarına kurulum yapabilir hatta network, storage, IP, firewall gibi
    - [Şu kaynakta da](https://nixops.readthedocs.io/en/latest/manual/migrating.html) Versiyon 1'den 2'ye geçişle alakalı konulara değinilmiş.

  Son tahlilde eğer 1.7 kullanmak istiyorsak özellikle ilgili NixOS versiyonunu bulup onun üzerinden nixops'u yüklememiz gerekiyor. Nixops 1.7 versiyonunun bulunduğu Nixpks'nin en son versiyonu [NixOS 23.05'de](https://github.com/NixOS/nixpkgs/blob/nixos-23.05/pkgs/tools/package-management/nixops/default.nix) bulabiliyoruz. O zaman registry (channel) olarak 23.05 versiyonunu registry'lere ekleyip bu versiyon üzeridnen Nixops'u kurabiliriz.

  ```bash
       nix registry add nixpkgs2305 github:NixOS/nixpkgs/nixos-23.05

       export NIXPKGS_ALLOW_INSECURE=1

       nix profile install flake:nixpkgs2305#nixops

       nixops --version
  ```

  Nixops'u kullanırken flake içine veya flake'den bağımsız olarak bir dosya içine konfigürasyonu yazabiliriz. Bu dosya içinde birden fazla makine tanımlayabiliriz. Bu makinelere bağlanarak NixOS konfigürasyonunu uygulayabiliriz.

  Flake içinde tanımlama yaparken alttaki gibi bir template kullanmalısınız. `nixopsConfigurations` bloğuna bakacak olursanız `network`, `defaults` ve `machine` bloklarını görebilirsiniz. `network` bloğunda genel network ayarları yapılırken `machine` bloğunda ise tekbir makinenin tanımını gösteriyor. Defaults bloğu ise tüm makinelere uygulanacak genel ayarları gösteriyor. Konfigürasyondaki databasefile kısmı ise eskiye uyumluluk için gerekiyor. Bu dosya içinde deployment'lar ve makine bilgileri tutuluyor.

  Network bloğuna eğer örneğin bir AWS konfigürasyonu yapacak olursak bu durumda makinelerimiz ve diğer AWS kaynakları AWS üzerinden Nixops tarafından oluşturulacaktır. Alttaki örnek Virtualbox üzerinde bir makine oluşturmak için kullanılabilir.

  ```nix
  # flake.nix
  {
  description = "Murat Cabuk NixOS Configuration";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-23.11";
    nixops.url = "github:NixOS/nixops";
  };

  outputs = { self, nixpkgs, ... }@inputs: {

  nixopsConfigurations.default = {

            inherit nixpkgs;

            network =  {
                description = "my virtual network";
                enableRollback = true;
                storage.legacy = {
                    databasefile = "~/.nixops/deployments.nixops";
                };

                deployment.targetEnv = "virtualbox";
                deployment.virtualbox.memorySize = 4096; # megabytes
                deployment.virtualbox.vcpu = 2; # number of cpus
                deployment.virtualbox.headless = true;

            };

            defaults = {
                imports = [ ];
            };

            machine = { config, pkgs, ...  }: {
                services = {
                  httpd.enable = true;
                  openssh.enable = true;
                };

                environment.systemPackages = with pkgs; [ vim git htop];
            };
    };

    nixosConfigurations.nixos = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      modules = [ /configuration.nix ];
      };
    };
  }

  ```

## Kubernetes Cluster'ı Oluşturmak

- [Kubenix](https://github.com/saschagrunert/kubernix): Bu araç Rust dili kullnılarak yazılmış amaç development ortamı için Kubernetes cluster'ı oluşturmak.

Nixops ile Kubernetes kurulumu için fikir olması adına altta 2 Github projesinin linkini paylaşıyorum.

- Sadene Kubernetes değil ELK, Kafka, Mesos ve Postgres cluster'larının da Nixops ile nasıl oluşturabilceğinize örnek olarak [şu linkten](https://github.com/thpham/magics/tree/master/k8s-cluster) inmceleyebilirsiniz.
- [Bu örnek](https://gist.github.com/offlinehacker/5f77f4ae9accc1859a4381429bd6fdbb) de sade bir şekilde 6 sunuculu bir Kubernetes cluster'ının nasıl yapılabileceğini gösteriyor.

Evet bu yazmızda bu kadar. Fırsat buldukça NixOS hakkındaki deneyimlerimi paylaşmaya devam edeceğim. Umarım faydalı olmuştur diğer yazılarımda görüşmek ümidiyle.

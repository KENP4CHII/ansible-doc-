# ANSIBLE ÇALIŞMA SORULARI


## Playbook Oluşturma ve Yürütme:

 - Bir Ansible playbook oluşturun ve bu playbook ile bir uzak sunucuda basit bir komut çalıştırma işlemi gerçekleştirin.
 
-  Playbook'unuzda birden fazla adımdan oluşan bir yapı oluşturarak sunucu yapılandırması yapın (örneğin, Nginx kurulumu ve yapılandırması). 
  
-  Ansible Vault kullanarak hassas verileri playbook'unuza nasıl entegre edeceğinizi gösterin. 
  
  ## Vault Dosyası Oluşturma:
İlk adım olarak, Ansible Vault kullanarak hassas verileri saklayacağınız bir Vault dosyası oluşturun. Bu dosyayı oluşturmak için aşağıdaki komutu kullanabiliriz

```yml
ansible-vault create secrets.yml
```

## Hassas Verileri Vault Dosyasına Eklemek:
Oluşturulan Vault dosyasına hassas verileri eklemek için, dosyayı düzenleyin ve YAML formatında verileri ekleyin. Örneğin:

```yaml
db_password: my_secret_password
api_key: my_api_key
```
Bu örnekte, db_password ve api_key adında iki hassas veri saklanmaktadır.

## Playbook'a Vault Dosyasını Entegre Etmek:

Playbook'unuzda Vault dosyasındaki hassas verilere erişmek için, Vault dosyasını playbook'a eklemeniz gerekmektedir. Bunu yapmak için playbook'unuzdaki değişkenler bölümünde, Vault dosyasının yolunu belirtin. Örneğin:

```yaml

- hosts: all
  vars_files:
    - secrets.yml
  tasks:
    - name: Task using sensitive data
      debug:
        msg: "Database password is {{ db_password }}, API key is {{ api_key }}"
```

## Playbook'u Çalıştırırken Vault Şifresini Belirtme:
Playbook'u çalıştırırken Vault dosyasına erişmek için, 

```yaml
--ask-vault-pass
```
 bayrağını kullanarak Vault şifresini belirtmeniz gerekmektedir. Örneğin:

```yaml
ansible-playbook --ask-vault-pass playbook.yml
```
  - Jinja2 şablonları kullanarak dinamik yapılandırma dosyaları oluşturun. e. Playbook'u farklı birçok sunucuda aynı anda çalıştırarak hızlı ve etkili bir yapılandırma yönetimi gerçekleştirin.
  
  ```yaml
- name: Konfigürasyon Yönetimi
  hosts: web_servers
  vars:
    nginx_port: 80
    nginx_server_name: "{{ inventory_hostname }}"
    nginx_document_root: "/var/www/{{ inventory_hostname }}"
  tasks:
    - name: NGINX yapılandırma dosyasını oluştur
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - restart nginx

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```
```yaml
# nginx.conf.j2

worker_processes auto;
events {
    worker_connections 1024;
}
http {
    include mime.types;
    default_type application/octet-stream;
    server {
        listen {{ nginx_port }};
        server_name {{ nginx_server_name }};
        root {{ nginx_document_root }};
        index index.html index.htm;
        location / {
            try_files $uri $uri/ =404;
        }
    }
}
```

```yaml
ansible-playbook -i inventory.ini playbook.yml
```

## Inventory Yönetimi ve Gruplama: 

- Bir Ansible inventory dosyası oluşturun ve bu dosyada farklı sunucu grupları tanımlayın (örneğin, web sunucuları, veritabanı sunucuları).


- Inventory dosyanızı dinamik olarak nasıl oluşturabileceğinizi araştırın ve uygulayın. 

Ansible, dinamik inventory oluşturmak için kullanabileceğiniz özelleştirilmiş betiklerin çalıştırılmasını destekler. Örneğin, bir Python, Bash veya diğer bir betik dili kullanarak, bir veritabanından, bir API'den veya başka bir kaynaktan sunucu bilgilerini alabilir ve bu bilgilere dayalı olarak inventory dosyanızı oluşturabilirsiniz. Bu betikleri çalıştırarak, dinamik bir şekilde sunucu envanterinizi güncel tutabilirsiniz.

- Bir sunucu grubuna ait IP adreslerini veya etiketlerini (tags) nasıl toplayacağınızı ve kullanacağınızı öğrenin. 

'ec2.py' betiğini indirin ve çalıştırılabilir hale getirin:

```yaml
wget https://raw.githubusercontent.com/ansible/ansible/stable-2.10/contrib/inventory/ec2.py
chmod +x ec2.py
```
ec2.ini adında bir yapılandırma dosyası oluşturun ve AWS kimlik bilgilerinizi ve diğer gerekli ayarları belirtin.

 'ec2.py' betiğini çalıştırarak sunucu bilgilerini alın:

```yaml
./ec2.py --list
```

Bu komut, tüm AWS sunucularını ve bunların IP adreslerini veya etiketlerini döndürecektir.

Daha sonra, bu IP adreslerini veya etiketleri, Ansible playbook'larında veya komutlarında hedef olarak belirleyebilirsiniz. Örneğin:

```yaml
ansible-playbook -i ./ec2.py playbook.yml --limit "tag_Name_web_servers"
```
Bu komut, 'tag_Name_web_servers' etiketine sahip tüm sunucuları hedef alarak playbook.yml dosyasını çalıştıracaktır.

-  Ansible grup değişkenleri kullanarak farklı sunucu gruplarına özgü yapılandırma ayarları nasıl uygulanır? 


### Grup Değişkenlerini Tanımlama:
İlk adım, farklı sunucu gruplarına ait grup değişkenlerini tanımlamaktır. Bu değişkenler, group_vars dizini altında grup adıyla oluşturulan YAML dosyalarında tanımlanır. Örneğin:

'group_vars/web_servers.yml'

```yaml
nginx_port: 80
nginx_document_root: "/var/www/html"
group_vars/database_servers.yml:
```
```yaml
db_port: 3306
db_name: my_database
```
### Playbook'u Oluşturma:
Daha sonra, playbook'u oluştururken grup değişkenlerini kullanabilirsiniz. Örneğin:

```yaml
---
- name: Konfigürasyon Yönetimi
  hosts: web_servers
  tasks:
    - name: Configure NGINX
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - restart nginx

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```
### Jinja2 Şablonlarını Kullanma:

Playbook'ta kullanılan Jinja2 şablonlarında, grup değişkenlerine erişerek farklı sunucu gruplarına özgü yapılandırma ayarlarını uygulayabilirsiniz. Örneğin, nginx.conf.j2 şablonunda:

```yaml
# nginx.conf.j2

worker_processes auto;
events {
    worker_connections 1024;
}
http {
    include mime.types;
    default_type application/octet-stream;
    server {
        listen {{ nginx_port }};
        server_name {{ inventory_hostname }};
        root {{ nginx_document_root }};
        index index.html index.htm;
        location / {
            try_files $uri $uri/ =404;
        }
    }
}
```

e. Ansible inventory dosyasındaki host parametrelerini ve varyasyonlarını nasıl kullanabileceğinizi gösterin.

```ini
# inventory.ini

[web_servers]
web1 ansible_host=192.168.1.101 ansible_port=22 ansible_user=ubuntu

[database_servers]
db1 ansible_host=10.0.0.5 ansible_port=2222 ansible_user=admin ansible_ssh_private_key_file=~/.ssh/db_key.pem
db2 ansible_host=10.0.0.6 ansible_port=2222 ansible_user=admin ansible_ssh_private_key_file=~/.ssh/db_key.pem

[load_balancers]
lb1.example.com

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

##	Modül Kullanımı ve Özelleştirme: 

- Ansible'in standart modüllerini kullanarak bir dosya oluşturma veya silme işlemi gerçekleştirin. 

### Dosya Oluşturma:

```yaml
---
- name: Dosya oluşturma işlemi
  hosts: localhost
  tasks:
    - name: Dosya oluştur
      file:
        path: /path/to/file.txt
        state: touch
```
### Dosya Silme:
```yaml
---
- name: Dosya silme işlemi
  hosts: localhost
  tasks:
    - name: Dosya sil
      file:
        path: /path/to/file.txt
        state: absent
```
- Bir kullanıcı hesabı oluşturma veya silme işlemleri için user modülünü kullanın. 

### Kullanıcı Hesabı Oluşturma:

```yaml
---
- name: Kullanıcı hesabı oluşturma işlemi
  hosts: localhost
  tasks:
    - name: Kullanıcı oluştur
      user:
        name: example_user
        state: present
        shell: /bin/bash
        password: "{{ 'password' | password_hash('sha512') }}"
```
### Kullanıcı Hesabı Silme:

```yaml
---
- name: Kullanıcı hesabı silme işlemi
  hosts: localhost
  tasks:
    - name: Kullanıcıyı sil
      user:
        name: example_user
        state: absent
```
-  Paket yönetimi işlemleri için apt veya yum modüllerini kullanarak bir paket yükleme veya kaldırma işlemi yapın.
  
### Kullanıcı Hesabı Oluşturma ve Paket Yükleme:

```yaml
Copy code
---
- name: Kullanıcı hesabı oluşturma ve paket yükleme işlemi
  hosts: localhost
  tasks:
    - name: Kullanıcı oluştur
      user:
        name: example_user
        state: present
        shell: /bin/bash

    - name: Paket yükle
      become: yes
      apt:
        name: vim
        state: present
```
### Kullanıcı Hesabı Silme ve Paket Kaldırma:

```yaml
---
- name: Kullanıcı hesabı silme ve paket kaldırma işlemi
  hosts: localhost
  tasks:
    - name: Kullanıcıyı sil
      user:
        name: example_user
        state: absent

    - name: Paket kaldır
      become: yes
      apt:
        name: vim
        state: absent
```
-   Servis yönetimi için service modülünü kullanarak bir servisi başlatın, durdurun veya yeniden başlatın.

Servisi Durdurma:

```yaml

---
- name: Servisi durdurma işlemi
  hosts: localhost
  tasks:
    - name: Servisi durdur
      service:
        name: nginx
        state: stopped
```
### Servisi Yeniden Başlatma:

```yaml

---
- name: Servisi yeniden başlatma işlemi
  hosts: localhost
  tasks:
    - name: Servisi yeniden başlat
      service:
        name: nginx
        state: restarted   
```
-   Kendi özel Ansible modüllerinizi nasıl yazabileceğinizi araştırın ve basit bir özel modül oluşturun.

## Roller ve Dosya Yapısı: 

- Ansible projesi için tipik bir dosya yapısı oluşturun (örneğin, roles, inventories, playbooks). 
  
 ```css 
  my_ansible_project/
│
├── inventories/
│   ├── production/
│   │   ├── hosts.ini
│   │   └── group_vars/
│   │       └── all.yml
│   └── staging/
│       ├── hosts.ini
│       └── group_vars/
│           └── all.yml
│
├── roles/
│   ├── common/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── templates/
│   └── webserver/
│       ├── tasks/
│       │   └── main.yml
│       ├── templates/
│       └── vars/
│           └── main.yml
│
└── playbooks/
    ├── deploy_webapp.yml
    └── setup_servers.yml

```    

#### inventories/: 
Ansible'in kullanacağı envanter dosyalarını ve grup değişkenlerini içeren dizin.
#### production/: 
Prodüksiyon ortamı için envanter dosyası ve grup değişkenleri.
#### staging/: 
Staging ortamı için envanter dosyası ve grup değişkenleri.
#### roles/: 
Ansible rollerini içeren dizin. Her rol, belirli bir yapılandırma veya görev kümesini temsil eder.
#### common/: 
Genel kullanıma yönelik görevler içeren rol.
#### tasks/:
 Genel görevleri içeren görev dosyaları.
#### templates/:
 Genel şablon dosyalarını içeren şablon dizini.
#### webserver/:
 Web sunucusu yapılandırması için özel rol.
#### tasks/: 
Web sunucusu yapılandırmasıyla ilgili görevleri içeren görev dosyaları.
#### templates/:
 Web sunucusu yapılandırması için özel şablon dosyalarını içeren şablon dizini.
#### vars/:
 Web sunucusu yapılandırması için özel değişken dosyalarını içeren değişken dizini.
#### playbooks/:
 Ansible playbook'larını içeren dizin.
#### deploy_webapp.yml:
 Bir web uygulamasını dağıtmak için playbook.
#### setup_servers.yml:
 Sunucuların temel yapılandırması için playbook.

- Bir Ansible rolü oluşturun ve bu rolü bir playbook içinde nasıl kullanacağınızı gösterin. 

### Ansible Rolü Oluşturma:
Öncelikle, yeni bir Ansible rolü oluşturmak için aşağıdaki komutu kullanabiliriz:

```bash
ansible-galaxy init my_role
```
Bu komut, my_role adında bir klasör oluşturacak ve bu klasör içinde bir rol yapısını hazırlayacaktır.

### Rolün Görevlerini Düzenleme:

Oluşturulan rol klasörüne girerek, rolün görevlerini içeren tasks/main.yml dosyasını düzenleyelim. Örneğin, aşağıdaki gibi bir görev ekleyebiliriz:

```yaml
---
- name: Ensure nginx is installed
  apt:
    name: nginx
    state: present
```    
Bu görev, nginx paketinin yüklü olup olmadığını kontrol eder ve eksikse yükler.

### Rolün Playbook ile Kullanımı:

Şimdi, oluşturduğumuz rolü bir playbook içinde kullanalım. Örnek olarak, aşağıdaki gibi bir playbook oluşturalım (my_playbook.yml):

```yaml
---
- name: My playbook with custom role
  hosts: servers
  become: true
  roles:
    - my_role
```

Bu playbook, servers grubundaki tüm sunuculara my_role adlı rolü uygular. Rolün içindeki görevler, bu sunucular üzerinde çalıştırılacaktır.

### Playbook'u Çalıştırma:

```bash
ansible-playbook -i inventory.ini my_playbook.yml
```
-  Role dosyalarınızı nasıl parametreize edeceğinizi ve yeniden kullanılabilir hale getireceğinizi öğrenin. 
  
-   Bir rol içindeki vars/main.yml dosyasını kullanarak değişkenler tanımlayın ve bu değişkenleri playbook'larda nasıl kullanacağınızı gösterin. 
  
-   Ansible Galaxy'den (veya başka bir kaynaktan) bir hazır rol indirin ve projenizde kullanın.

## Ansible Tower veya AWX Entegrasyonu: 
- Ansible Tower veya AWX uygulamasını kurun ve yapılandırın. 

-  Ansible Tower veya AWX'yi bir Ansible projesiyle nasıl entegre edeceğinizi ve yöneteceğinizi öğrenin.

-  AWX veya Tower'da bir çalışma zamanı (job) oluşturun ve bu çalışma zamanını zamanlanmış olarak nasıl çalıştıracağınızı ayarlayın. 

-  Proje, envanter, çalışma zamanı ve kullanıcı yönetimi yapın. 

-  AWX veya Tower'ın API'sini kullanarak otomasyon senaryoları oluşturun veya mevcut senaryoları otomatikleştirin.

**Первые шаги с Ansible**

**1. Описание задания**

Подготовить стенд на Vagrant как минимум с одним сервером. На этом сервере используя Ansible необходимо развернуть nginx со следующими условиями:

    необходимо использовать модуль yum/apt;
    конфигурационные файлы должны быть взяты из шаблона jinja2 с перемененными;
    после установки nginx должен быть в режиме enabled в systemd;
    должен быть использован notify для старта nginx после установки;
    сайт должен слушать на нестандартном порту - 8080, для этого использовать переменные в Ansible.
    
    
    
 **2. Выполенение задания.**

1. Рабочий стенд для выполнения занадания физическая машина с Ubuntu 22.04, в ней виртаульная машина виртуал бокс, развернутая через Vagrant.
2. Поготовка рабочего стенда, проверил наличие и версию Python, установил и проверил версию Ansible.
```spa@stnd:~/Ass$ python3 --version
   Python 3.10.6
   spa@stnd:~/Ass$ ansible --version
   ansible [core 2.14.6]
   config file = /home/spa/Ass/ansible.cfg
   configured module search path = ['/home/spa/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
   ansible python module location = /usr/lib/python3/dist-packages/ansible
   ansible collection location = /home/spa/.ansible/collections:/usr/share/ansible/collections
   executable location = /usr/bin/ansible
   python version = 3.10.6 (main, Mar 10 2023, 10:55:28) [GCC 11.3.0] (/usr/bin/python3)
   jinja version = 3.0.3
   libyaml = True 
```
3. Создал директори проекта, создал вагрантфайл с конфигурацией виртаульной машины, файл добавлен в репозиторию.  
   Поднял виртуальную машину, проверил настройки ssh подключения машины.
```   vagrant ssh-config
      Host nginx
      HostName 127.0.0.1
      User vagrant
      Port 2222
      UserKnownHostsFile /dev/null
      StrictHostKeyChecking no
      PasswordAuthentication no
      IdentityFile /home/spa/Ass/.vagrant/machines/nginx/virtualbox/private_key
      IdentitiesOnly yes
      LogLevel FATAL
  ```    
   На основе этой информации и информации указанной в вагрнат файле создал inventory файл.   
5. В директории проекта создал inventory файл, в которой внес информацию о хосте, который буду конфигурировать при помощи Ansible
   ```[nginx]
      nginx ansible_host=192.168.56.10
      [nginx:vars]
      ansible_user=vagrant
      ansible_ssh_private_key_file=/home/spa/Ass/.vagrant/machines/nginx/virtualbox/private_key
    ```    
6. В директороий проекта создал ansible.cfg.
   ```[defaults]
      inventory = ./inventory
      remote_user = vagrant
      host_key_checking = False
      retry_files_enabled = False
   ```     
7. Проверил доступность виртуальной машины.
  ```spa@stnd:~/Ass$ ansible nginx -m ping
      nginx | SUCCESS => {
     "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
     },
     "changed": false,
     "ping": "pong"
   }
   ```
8. Выполнил Ad-Hoc команду для проверки работы Ansible.
```spa@stnd:~/Ass$ ansible nginx -m command -a "uname -r"
    nginx | CHANGED | rc=0 >>
    3.10.0-1127.el7.x86_64
```
9. Создал файл плейбука с именем nginx.yml, добавлен в репозиторий, основываясь на методичке. Так же создал в директории проекта поддиректорию с именем templates.
В эту поддиректорию положил файл шаблон с именем nginx.conf.j2.

10. После этого запусти выполенение плейбука.

```ansible-playbook nginx.yml
   PLAY [NGINX | Install and configure NGINX] **********************************************************

  TASK [Gathering Facts] ******************************************************************************
  ok: [nginx]

  TASK [NGINX | Install EPEL Repo package from standart repo] *****************************************
  ok: [nginx]

  TASK [NGINX | Install NGINX package from EPEL Repo] *************************************************
  ok: [nginx]

  TASK [NGINX | Create NGINX config file from template] ***********************************************
  changed: [nginx]

  RUNNING HANDLER [reload nginx] **********************************************************************
  changed: [nginx]

  PLAY RECAP ******************************************************************************************
  nginx                      : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

11. После успешного выполениня плейбука, открыл браузер и проверил доступность сайт по нужному адресу и порту:  http://192.168.5610:8080.  
  
12. Так же проверил работу сайте из терминала.  

```spa@stnd:~/Ass$ curl http://192.168.56.10:8080
  <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
  <html>
  <head>
     <title>Welcome to CentOS</title>
     <style rel="stylesheet" type="text/css"> 

	    html {
	    background-image:url(img/html-background.png);
	    background-color: white;
	    font-family: "DejaVu Sans", "Liberation Sans", sans-serif;
	    font-size: 0.85em;
	    line-height: 1.25em;
	    margin: 0 4% 0 4%;
	    }

	    body {
	    border: 10px solid #fff;
	    margin:0;
	    padding:0;
	    background: #fff;
	    }
```

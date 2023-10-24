# ansible
Плейбуки Ansible для создания двух веб-серверов Nginx и одного сервера Haproxy.
Настраивает и запускает zabbix-agent в docker-контейнере.
Подходит для самостоятельного изучения Ansible.

Плейбуки протестированы на Photon OS GA 4.0 и в качестве целевых виртуальных машин используется тестовая среда, развертываемая с помощью Terraform-манифеста https://github.com/jabuinn/ansible-lab.

Что делает набор плейбуков:
1. Устанавливает Nginx на серверы node-vm-01/node-vm-02.
2. Конфигурирует Nginx (копирует целевой файл index.html) и запускает Nginx.
3. Устанавливает Haproxy на сервер node-haproxy.
4. Конфигурирует Haproxy (использует шаблон Jinja с настроенным frontent/backend/stats) и запускает Haproxy.
5. Запускает zabbix-agent в docker-контейнере на серверах node-vm-01/node-vm-02/node-haproxy. 

Что нужно для запуска набора плейбуков:
1. Получите Облачный ЦОД в VMware Cloud Director с объемом ресурсов не менее, чем:
   CPU = 8,
   RAM = 6 ГБ,
   HDD = 100 Гб,
   NSX-T edge = 1,
   IP (routed) = 1,
2. Скачайте и разместите в рабочую папку OVA-шаблон Photon OS GA 4.0 (~234 МБ): https://packages.vmware.com/photon/4.0/GA/ova/photon-hw11-4.0-1526e30ba0.ova
3. Скачайте манифест в рабочую папку, например:
   $ git clone https://github.com/jabuinn/ansible-lab ~/ansible-lab
4. Внесите в terraform.tfvars адрес Cloud Director, логин/пароль УЗ с правами Org Admin, наименование Org, OrgVDC и Edge. При необходимости заменить тип storage policy и имя шаблона PhotonOS. Можно использовать логин по API Token, для этого его нужно внести в файл token.json и внести изменения в файл main.tf в части авторизации.
5. Выполните последовательно команды:
   $ terraform init
   $ terraform apply -auto-approve
6. После успешного развертывания виртуальных машин, выполните ssh-вход по адресу ssh-haproxy (логин-пароль будут в выводе после выполнения манифеста Terraform).
7. Скачайте набор плейбуков в рабочую папку, например:
   $ git clone https://github.com/jabuinn/ansible /tmp/ansible
8. Выполните последовательно плейбуки (пароль root будет выведен при выполнении п.5):
   $ ansible-playbook -i inventory.txt -k play_web.yml
   $ ansible-playbook -i inventory.txt -k play_haproxy.yml
   $ ansible-playbook -i inventory.txt -k play_zabbix_agent.yml
9. Проверьте работоспособность Nginx, Haproxy и Zabbix по ссылкам, полученным в п.5.
10. Добавьте в консоли Zabbix-сервера хосты для мониторинга.

Что нужно сделать после успешного выполнения
1. Запустите удаление созданной инфраструктуры. Перейдите в каталог с манифестом Terraform и выполните команду:
   $ terraform destroy -auto-approve
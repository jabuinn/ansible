# ansible
Плейбуки Ansible для создания двух веб-серверов Nginx и одного сервера Haproxy. 
Подходит для самостоятельного изучения Ansible.

Плейбуки протестированы на Photon OS GA 4.0 и в качестве целевых виртуальных машин используется тестовая среда, развертываемая с помощью Terrafor-манифеста https://github.com/jabuinn/ansible-lab.

Что делает набор плейбуков:
1. Устанавливает Nginx на серверы node-vm-01/node-vm-02.
2. Конфигурирует Nginx (копирует целевой файл index.html) и запускает Nginx.
3. Устанавливает Haproxy на сервер app.
4. Конфигурирует Haproxy (использует шаблон Jinja с настроенным frontent/backend/stats) и запускает Haproxy.

Что нужно для запуска набора плейбуков:
1. Получите Облачный ЦОД в VMware Cloud Director с объемом ресурсов не менее, чем:
   CPU = 2
   RAM = 1 ГБ
   HDD = 60 Гб
   NSX-T edge = 1
   IP (routed) = 1
2. Скачайте и разместите в рабочую папку OVA-шаблон Photon OS GA 4.0 (~234 МБ): https://packages.vmware.com/photon/4.0/GA/ova/photon-hw11-4.0-1526e30ba0.ova
3. Скачайте манифест в рабочую папку, например:
   $ git clone https://github.com/jabuinn/ansible-lab ~/ansible-lab
4. Внесите в terraform.tfvars адрес Cloud Director, логин/пароль УЗ с правами Org Admin, наименование Org, OrgVDC и Edge. При необходимости заменить тип storage policy и имя шаблона PhotonOS. Можно использовать логин по API Token, для этого его нужно внести в файл token.json и внести изменения в файл main.tf в части авторизации.
5. Выполните последовательно команды:
   $ terraform init
   $ terraform apply -auto-approve
6. После успешного развертывания виртуальных машин, выполните вход по ssh на виртуальную машину node-haproxy (app).
7. Скачайте набор плейбуков в рабочую папку, например:
   $ git clone https://github.com/jabuinn/ansible ~/ansible
8. Выполните последовательно плейбуки (пароль root будет выведен при выполнении п.5):
   $ ansible-playbook -i inventory.txt -k play_web.yml
   $ ansible-playbook -i inventory.txt -k play_haproxy.yml
9. Проверьте работоспособность Nginx и Haproxy по ссылкам, полученным в п.5

Что нужно сделать после успешного выполнения
1. Запустите удаление созданной инфраструктуры. Перейдите в каталог с манифестом Terraform и выполните команду:
   $ terraform destroy -auto-approve
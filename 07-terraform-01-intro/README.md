# Домашнее задание к занятию "7.1. Инфраструктура как код"

## Проценко Анастасия

## Задача 1. Выбор инструментов. 
 
### Легенда
 
*Через час совещание на котором менеджер расскажет о новом проекте. Начать работу над которым надо 
будет уже сегодня. 
На данный момент известно, что это будет сервис, который ваша компания будет предоставлять внешним заказчикам.
Первое время, скорее всего, будет один внешний клиент, со временем внешних клиентов станет больше.*

*Так же по разговорам в компании есть вероятность, что техническое задание еще не четкое, что приведет к большому
количеству небольших релизов, тестирований интеграций, откатов, доработок, то есть скучно не будет.*  
   
*Вам, как девопс инженеру, будет необходимо принять решение об инструментах для организации инфраструктуры.
На данный момент в вашей компании уже используются следующие инструменты:*
- *остатки Сloud Formation,*
- *некоторые образы сделаны при помощи Packer,*
- *год назад начали активно использовать Terraform,*
- *разработчики привыкли использовать Docker,*
- *уже есть большая база Kubernetes конфигураций,*
- *для автоматизации процессов используется Teamcity,*
- *также есть совсем немного Ansible скриптов,*
- *и ряд bash скриптов для упрощения рутинных задач.* 

*Для этого в рамках совещания надо будет выяснить подробности о проекте, что бы в итоге определиться с инструментами:*

1. *Какой тип инфраструктуры будем использовать для этого проекта: изменяемый или не изменяемый?*  
Только не изменяемая инфраструктура на серверах. В изменяемой велик риск, что некоторые изменения могут отрицательно повлиять на работу.  
2. *Будет ли центральный сервер для управления инфраструктурой?*  
Либо воспользуемся кластером, либо вообще будем без центрального сервера, чтобы минимизировать дополнительное обслуживание.
3. *Будут ли агенты на серверах?*  
Не будет в этом необходимости. Ansible и Terraform агентами не пользуются. С установкой агентов появляется лишняя точка отказа, + на агенты нужно тратить время - устанавливать, обновлять, следить за их функционированием.
4. *Будут ли использованы средства для управления конфигурацией или инициализации ресурсов?*  
У Ansible и Terraform нет центрального сервера и не требуют установки агентов.
 
*В связи с тем, что проект стартует уже сегодня, в рамках совещания надо будет определиться со всеми этими вопросами.*  

### В результате задачи необходимо

1. *Ответить на четыре вопроса представленных в разделе "Легенда".* 
2. *Какие инструменты из уже используемых вы хотели бы использовать для нового проекта?*  
Kubernetes, Terraform, Ansible.
3. *Хотите ли рассмотреть возможность внедрения новых инструментов для этого проекта?*  
Безусловно возникнет потребность в новых инструментах, которые исходно не были предусмотрены.

## Задача 2. Установка терраформ. 

*Официальный сайт: https://www.terraform.io/*

*Установите терраформ при помощи менеджера пакетов используемого в вашей операционной системе.
В виде результата этой задачи приложите вывод команды terraform --version.*
 
![Версия Terraform](terraform.png)

## Задача 3. Поддержка легаси кода. 

*В какой-то момент вы обновили терраформ до новой версии, например с 0.12 до 0.13.* 
*А код одного из проектов настолько устарел, что не может работать с версией 0.13.* 
*В связи с этим необходимо сделать так, чтобы вы могли одновременно использовать последнюю версию терраформа установленную при помощи
штатного менеджера пакетов и устаревшую версию 0.12.* 

*В виде результата этой задачи приложите вывод --version двух версий терраформа доступных на вашем компьютере 
или виртуальной машине.*

На сайте Hashicorp размещена только текущая версия terraform, приложить вывод двух различных версий команды ```terraform -version``` не смогу, но предполагаю, что для поддержки legacy code достаточно устаревшую версию бинарника не удалять, а переименовывать; сделать линк на старую версию бинарника, переименовав его (добавить номер версии).

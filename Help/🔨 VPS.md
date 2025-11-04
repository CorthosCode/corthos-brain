# 🔨 VPS

![](assets/vW30fmWZ6TvxAwlzW8ZT6MLfY0_H5q-bvEP7jdNy1D4=.png)



###### Установка SSH-сервера

```shellscript
sudo apt install openssh-server
sudo systemctl start ssh
sudo systemctl status ssh
sudo systemctl enable ssh
```



###### Как узнать IP адрес сервера

```shellscript
ifconfig | grep inet | awk '{ print $2 }'
```



###### Как узнать какие есть пользователи на сервере

```shellscript
cut -d: -f1 /etc/passwd
```



###### В случае ошибки



Если ранее на ноутбуке подключался к серверу но появилась следующая ошибка:

```shellscript
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@ WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!  @ 
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:4w2I6kkV4TOU4s9vtAfkoslgK1xHEIli1+lQiiHUd2s.
Please contact your system administrator.
Add correct host key in /Users/nakotov4/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /Users/nakotov4/.ssh/known_hosts:8
Host key for 192.168.1.172 has changed and you have requested strict checking.
Host key verification failed.
```

Необходимо выполнить команду:

```
ssh-keygen -R <IP/host>
```



###### Полезные ссылки

[https://my.aeza.net/](https://my.aeza.net/services/998608)

https://ruvds.com/ru/vps\_start/ 

https://timeweb.com/ru/services/vds/ 


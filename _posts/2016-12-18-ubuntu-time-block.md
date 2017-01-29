---
layout: post
title: "Bloqueando máquinas Ubuntu pelo horário"
---

**Objetivo**: bloquear máquina de funcionário ás 18:00. Para evitar supresas desagradáveis, alertar o usuário minutos antes do bloqueio da máquina.

Ferramentas utilizadas:

* notify-send para alertar o usuário;
* pam_time para bloqueio do usuário;
* crond para executar os agendamentos necessários;

O pam_time é um módulo PAM que já vem presente por padrão na grande maioria das distros. 

Configurado seguintes agendamentos em /etc/crontab:

```
45 17 * * *	user	DISPLAY=:0.0 notify-send -u low "Seu computador será bloqueado daqui 15 minutos!"
50 17 * * *	user	DISPLAY=:0.0 notify-send -u normal "Seu computador será bloqueado daqui 10 minutos!"
59 17 * * *	user	DISPLAY=:0.0 notify-send -u critical "Encerre seus trabalhos, o sistema será bloqueado!"
00 18 * * *	root	systemctl restart lightdm.service
```

Configure regra de bloqueio em /etc/security/time.conf:

```
lightdm*;*;user;!Al0759-1800
```

No arquivo /etc/pam.d/lightdm, insira a seguinte linha:

```
account   required  pam_time.so
```

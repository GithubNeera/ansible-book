# 用語
## インベントリ
* Ansibleが知っているホスト群をインベントリと呼ぶ
* インベントリファイル
  - ホスト群を記述するデフォルトの方法
  - localhostはインベントリに自動的に追加される
    - localhostにはSSH接続はせずに直接接続する
  - 変数に指定可能なものは論理型と文字列だけ(他ではいろいろ使える)

## ファクト
* CPUのアーキテクチャ、OS,IPアドレス、メモリ、ディスクなどのホストの詳細情報のことをファクトという
  - 変数に保存される

## モジュール
Facts返すモジュール
* ec2_facts
* docker

## 変数の優先順位
1. ansible-playbook -e var=value
1. この優先順位欄にないすべての方法？
1. ホスト or グループに対して利用している定義(インベントリファイル、YAML)
1. ファクト
1. ロールのdefaults/main.yml定義

**1(一番上)が最優先**

## add_host
## group_by
* あまり使わないらしい

# ansible コマンド
* ansible all -m setup
  - ファクトをすべて出力する
* ansible vagrant1 -m setup -a 'filter=ansible_eth*'
  - filter利用

```log
$ ansible testserver -m ping
testserver | SUCCESS => {
    "changed": false,
        "ping": "pong"
        }
```


```log
$ ansible testserver -m command -a uptime
testserver | SUCCESS | rc=0 >>
 07:14:09 up 35 min,  1 user,  load average: 0.00, 0.01, 0.05
```


```log
$ ansible testserver -a "tail /var/log/dmesg"
testserver | SUCCESS | rc=0 >>
[   16.671543] type=1400 audit(1505803131.575:9): apparmor="STATUS" operation="profile_replace" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-client.action" pid=1026 comm="apparmor_parser"
[   16.671546] type=1400 audit(1505803131.575:10): apparmor="STATUS" operation="profile_replace" profile="unconfined" name="/usr/lib/connman/scripts/dhclient-script" pid=1026 comm="apparmor_parser"
[   16.948275] type=1400 audit(1505803131.851:11): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/sbin/tcpdump" pid=1028 comm="apparmor_parser"
[   17.021484] vboxvideo: Unknown symbol drm_open (err 0)
[   17.021501] vboxvideo: Unknown symbol drm_poll (err 0)
[   17.021505] vboxvideo: Unknown symbol drm_pci_init (err 0)
[   17.021510] vboxvideo: Unknown symbol drm_ioctl (err 0)
[   17.021515] vboxvideo: Unknown symbol drm_mmap (err 0)
[   17.021519] vboxvideo: Unknown symbol drm_pci_exit (err 0)
[   17.021523] vboxvideo: Unknown symbol drm_release (err 0)
```


```log
$ ansible testserver -s -a "tail /var/log/syslog"
testserver | SUCCESS | rc=0 >>
Sep 19 07:14:09 vagrant-ubuntu-trusty-64 ansible-command: Invoked with warn=True executable=None _uses_shell=False _raw_params=uptime removes=None creates=None chdir=None
Sep 19 07:15:17 vagrant-ubuntu-trusty-64 ansible-ping: Invoked with data=None
Sep 19 07:15:45 vagrant-ubuntu-trusty-64 puppet-agent[1298]: Could not request certificate: getaddrinfo: Name or service not known
Sep 19 07:16:47 vagrant-ubuntu-trusty-64 ansible-command: Invoked with warn=True executable=None _uses_shell=False _raw_params=tail /var/log/dmesg removes=None creates=None chdir=None
Sep 19 07:17:01 vagrant-ubuntu-trusty-64 CRON[2700]: (root) CMD (   cd / && run-parts --report /etc/cron.hourly)
Sep 19 07:17:48 vagrant-ubuntu-trusty-64 puppet-agent[1298]: Could not request certificate: getaddrinfo: Name or service not known
Sep 19 07:28:01 vagrant-ubuntu-trusty-64 puppet-agent[1298]: message repeated 5 times: [ Could not request certificate: getaddrinfo: Name or service not known]
Sep 19 07:28:34 vagrant-ubuntu-trusty-64 ansible-command: Invoked with warn=True executable=None _uses_shell=False _raw_params=tail /var/log/dmsg removes=None creates=None chdir=None
Sep 19 07:28:57 vagrant-ubuntu-trusty-64 ansible-command: Invoked with warn=True executable=None _uses_shell=False _raw_params=tail /var/log/dmesg removes=None creates=None chdir=None
Sep 19 07:29:25 vagrant-ubuntu-trusty-64 ansible-command: Invoked with warn=True executable=None _uses_shell=False _raw_params=tail /var/log/syslog removes=None creates=None chdir=None

$ ansible testserver -a "tail /var/log/syslog"
testserver | FAILED | rc=1 >>
tail: cannot open ‘/var/log/syslog’ for reading: Permission denied
```

```log
$ ansible testserver -s -m apt -a "name=nginx update_cache=yes"
testserver | SUCCESS => {
    "cache_update_time": 1505806430, 
    "cache_updated": true, 
    "changed": true, 
    "stderr": "", 
    "stderr_lines": [], 
    "stdout": "Reading package lists...\nBuilding dependency tree...\nReading state information...\nThe following extra packages will be installed:\n  libxslt1.1 nginx-common nginx-core\nSuggested packages:\n  fcgiwrap nginx-doc\nThe following NEW packages will be installed:\n  libxslt1.1 nginx nginx-common nginx-core\n0 upgraded, 4 newly installed, 0 to remove and 1 not upgraded.\nNeed to get 495 kB of archives.\nAfter this operation, 1802 kB of additional disk space will be used.\nGet:1 http://archive.ubuntu.com/ubuntu/ trusty-updates/main libxslt1.1 amd64 1.1.28-2ubuntu0.1 [145 kB]\nGet:2 http://archive.ubuntu.com/ubuntu/ trusty-updates/main nginx-common all 1.4.6-1ubuntu3.8 [19.1 kB]\nGet:3 http://archive.ubuntu.com/ubuntu/ trusty-updates/main nginx-core amd64 1.4.6-1ubuntu3.8 [325 kB]\nGet:4 http://archive.ubuntu.com/ubuntu/ trusty-updates/main nginx all 1.4.6-1ubuntu3.8 [5394 B]\nPreconfiguring packages ...\nFetched 495 kB in 7s (70.0 kB/s)\nSelecting previously unselected package libxslt1.1:amd64.\n(Reading database ... 63129 files and directories currently installed.)\nPreparing to unpack .../libxslt1.1_1.1.28-2ubuntu0.1_amd64.deb ...\nUnpacking libxslt1.1:amd64 (1.1.28-2ubuntu0.1) ...\nSelecting previously unselected package nginx-common.\nPreparing to unpack .../nginx-common_1.4.6-1ubuntu3.8_all.deb ...\nUnpacking nginx-common (1.4.6-1ubuntu3.8) ...\nSelecting previously unselected package nginx-core.\nPreparing to unpack .../nginx-core_1.4.6-1ubuntu3.8_amd64.deb ...\nUnpacking nginx-core (1.4.6-1ubuntu3.8) ...\nSelecting previously unselected package nginx.\nPreparing to unpack .../nginx_1.4.6-1ubuntu3.8_all.deb ...\nUnpacking nginx (1.4.6-1ubuntu3.8) ...\nProcessing triggers for ufw (0.34~rc-0ubuntu2) ...\nProcessing triggers for ureadahead (0.100.0-16) ...\nProcessing triggers for man-db (2.6.7.1-1ubuntu1) ...\nSetting up libxslt1.1:amd64 (1.1.28-2ubuntu0.1) ...\nSetting up nginx-common (1.4.6-1ubuntu3.8) ...\nProcessing triggers for ufw (0.34~rc-0ubuntu2) ...\nProcessing triggers for ureadahead (0.100.0-16) ...\nSetting up nginx-core (1.4.6-1ubuntu3.8) ...\nSetting up nginx (1.4.6-1ubuntu3.8) ...\nProcessing triggers for libc-bin (2.19-0ubuntu6.13) ...\n", 
    "stdout_lines": [
        "Reading package lists...", 
        "Building dependency tree...", 
        "Reading state information...", 
        "The following extra packages will be installed:", 
        "  libxslt1.1 nginx-common nginx-core", 
        "Suggested packages:", 
        "  fcgiwrap nginx-doc", 
        "The following NEW packages will be installed:", 
        "  libxslt1.1 nginx nginx-common nginx-core", 
        "0 upgraded, 4 newly installed, 0 to remove and 1 not upgraded.", 
        "Need to get 495 kB of archives.", 
        "After this operation, 1802 kB of additional disk space will be used.", 
        "Get:1 http://archive.ubuntu.com/ubuntu/ trusty-updates/main libxslt1.1 amd64 1.1.28-2ubuntu0.1 [145 kB]", 
        "Get:2 http://archive.ubuntu.com/ubuntu/ trusty-updates/main nginx-common all 1.4.6-1ubuntu3.8 [19.1 kB]", 
        "Get:3 http://archive.ubuntu.com/ubuntu/ trusty-updates/main nginx-core amd64 1.4.6-1ubuntu3.8 [325 kB]", 
        "Get:4 http://archive.ubuntu.com/ubuntu/ trusty-updates/main nginx all 1.4.6-1ubuntu3.8 [5394 B]", 
        "Preconfiguring packages ...", 
        "Fetched 495 kB in 7s (70.0 kB/s)", 
        "Selecting previously unselected package libxslt1.1:amd64.", 
        "(Reading database ... 63129 files and directories currently installed.)", 
        "Preparing to unpack .../libxslt1.1_1.1.28-2ubuntu0.1_amd64.deb ...", 
        "Unpacking libxslt1.1:amd64 (1.1.28-2ubuntu0.1) ...", 
        "Selecting previously unselected package nginx-common.", 
        "Preparing to unpack .../nginx-common_1.4.6-1ubuntu3.8_all.deb ...", 
        "Unpacking nginx-common (1.4.6-1ubuntu3.8) ...", 
        "Selecting previously unselected package nginx-core.", 
        "Preparing to unpack .../nginx-core_1.4.6-1ubuntu3.8_amd64.deb ...", 
        "Unpacking nginx-core (1.4.6-1ubuntu3.8) ...", 
        "Selecting previously unselected package nginx.", 
        "Preparing to unpack .../nginx_1.4.6-1ubuntu3.8_all.deb ...", 
        "Unpacking nginx (1.4.6-1ubuntu3.8) ...", 
        "Processing triggers for ufw (0.34~rc-0ubuntu2) ...", 
        "Processing triggers for ureadahead (0.100.0-16) ...", 
        "Processing triggers for man-db (2.6.7.1-1ubuntu1) ...", 
        "Setting up libxslt1.1:amd64 (1.1.28-2ubuntu0.1) ...", 
        "Setting up nginx-common (1.4.6-1ubuntu3.8) ...", 
        "Processing triggers for ufw (0.34~rc-0ubuntu2) ...", 
        "Processing triggers for ureadahead (0.100.0-16) ...", 
        "Setting up nginx-core (1.4.6-1ubuntu3.8) ...", 
        "Setting up nginx (1.4.6-1ubuntu3.8) ...", 
        "Processing triggers for libc-bin (2.19-0ubuntu6.13) ..."
    ]
}
```

```log
$ ansible testserver -s -m service -a "name=nginx state=restarted"
testserver | SUCCESS => {
    "changed": true,
    "name": "nginx",
    "state": "started"
```


```log
ansible webservers -m ping                                                                                                 master ✱ ◼
testserver | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

```log
ansible vagrant2 -a "ip addr show dev eth0"                                                              master ✱
vagrant2 | SUCCESS | rc=0 >>
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:cf:65:24 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fecf:6524/64 scope link 
       valid_lft forever preferred_lft forever
```


```log
ansible all -a "date"
ansible '*' -a "date"

vagrant1 | SUCCESS | rc=0 >>
Wed Sep 20 09:38:24 UTC 2017

vagrant2 | SUCCESS | rc=0 >>
Wed Sep 20 09:38:24 UTC 2017

vagrant3 | SUCCESS | rc=0 >>
Wed Sep 20 09:38:24 UTC 2017
```

# ansible-playbook
* --list-tasks

## log

```log
ansible-playbook web-notls.yml                                                                                             master ✱ ◼

PLAY [Configure webserver with nginx] ********************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************
ok: [testserver]

TASK [install nginx] *************************************************************************************************************************************
ok: [testserver]

TASK [copy nginx config file] ****************************************************************************************************************************
changed: [testserver]

TASK [enable configuration] ******************************************************************************************************************************
ok: [testserver]

TASK [copy index.html] ***********************************************************************************************************************************
changed: [testserver]

TASK [restart nginx] *************************************************************************************************************************************
changed: [testserver]

PLAY RECAP ***********************************************************************************************************************************************
testserver                 : ok=6    changed=3    unreachable=0    failed=0   
```

```log
ansible-playbook greet.yml -e greeing=hiya

PLAY [pass a message on the command line] **************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [output a message] ********************************************************
ok: [localhost] => {
    "msg": "you didn't specify a message"
}

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0   

$ ansible-playbook greet.yml -e @greetvars.yml

PLAY [pass a message on the command line] ******************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************
ok: [localhost]

TASK [output a message] ************************************************************************************************************
ok: [localhost] => {
    "msg": "hiya"
}

PLAY RECAP *************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0   
```

```log
ansible-playbook --list-tasks web-tls.yml                                                            master ✱ ◼

playbook: web-tls.yml

  play #1 (webservers): Configure webserver with nginx	TAGS: []
    tasks:
      install nginx	TAGS: []
      create directories for TLS certificates	TAGS: []
      copy TLS key	TAGS: []
      copy TLS certificate	TAGS: []
      copy nginx config file	TAGS: []
      enable configuration	TAGS: []
      copy index.html	TAGS: []
      restart nginx	TAGS: []
```


# ansible-doc
```log
ansible-doc service
```

# vagrant
* up
* reload
* destroy --force

## log

```log
vagrant reload                                                                                                               master ◼
==> default: Attempting graceful shutdown of VM...
==> default: Checking if box 'ubuntu/trusty64' is up to date...
==> default: Clearing any previously set forwarded ports...
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 80 (guest) => 8080 (host) (adapter 1)
    default: 443 (guest) => 8443 (host) (adapter 1)
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default: 
    default: Guest Additions Version: 4.3.36
    default: VirtualBox Version: 5.0
==> default: Mounting shared folders...
    default: /vagrant => /Users/d-inoue/workspace/ansible/playbooks
==> default: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> default: flag to force provisioning. Provisioners marked to run always will still run.
```

```log
vagrant destroy --force
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```

```log
$ vagrant status                                                                                           master ✱
Current machine states:

vagrant1                  running (virtualbox)
vagrant2                  running (virtualbox)
vagrant3                  running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

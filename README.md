# Ansible on Docker
English follows Japanese

Dockerを使ってAnsibleをCentOS,Ubuntuで試すチュートリアルです。

## チュートリアルで試すこと
- Ansibleを複数OS（CentOSとUbuntu）で動かす
- ターゲットノードにPythonがないと動かないことを試す
- パスワードの暗号化（ansible vault)を試す

**AnsibleはPythonがないと動きません。Ansibleは、エージェントレス（ターゲットにAnsibleをインストールする必要がない）が他のツールに比べて魅力ですが、ターゲットノードには、Pythonが使えることは必須となっています。Pythonを入れられない機器（ネットワーク機器など）を管理する際は注意が必要です。**


## 使用OS
- CentOS
- Ubuntu(２台)
  - うち１台はPythonを削除します。
  - UbuntuはパッケージマネージャにAptを使っており、AptはPythonに依存しないので、Pythonを消すことができます。

## 検証環境(ローカル)
- MacOS Monterey 12.0.1
- Docker Desktop 4.3.2 (72729)


## ディレクトリ構成
```
.
├── README.md
├── docker
│   ├── ansible
│   │   └── Dockerfile      Ansible master node DockerFile
│   ├── target-centos
│   │   └── Dockerfile      Target node DockerFile(OS: CentOS)
│   ├── target-no-python-ubuntu
│   │   └── Dockerfile      Target node DockerFile(OS: ubuntu without python)
│   └── target-ubuntu
│       └── Dockerfile      Target node DockerFile(OS: ubuntu with python)
├── docker-compose.yml      docker-compose file
├── inventry.ini            for Ansible inventry file
├── playbook.yml            for Ansible playbook file
└── vaulted_vars.yaml       for Ansible password encrypted file

```

## 使い方
1. Dockerコンテナの起動
```
docker-compose up -d
```
2. Ansibleコンテナに接続
```
docker exec -it ansible /bin/bash
```
3. targetに対し、sshの接続確認
    1. centos, ubuntu への接続
        ```
        ssh target-centos  # yesで接続
        exit
        ssh target-ubuntu    # yesで接続
        root    # loginパスワード:rootを入力（設定方法は後述）
        python3 -V  # pythonのバージョンが表示されることを確認
        exit
        ```
    2. ubuntu without python への接続
        ```
        ssh target-no-python-ubuntu   # yesで接続
        root    # loginパスワード:rootを入力（設定方法は後述）
        python3 -V  # pythonが入っていないことを確認
        exit
        ```


4. target nodeに対し、ansibleコマンドを実行
```
ansible-playbook -i inventry.ini playbook.yml --ask-vault-pass -e @vaulted_vars.yaml


# vaultファイルの復号パスワードを聞くように設定しています。
# -e @<filename> とすることで、変数ファイルを渡すことができます。

pass # Vault password: pass を入力します。
```

5. 実行ログの確認（pythonがないUbuntuのみ失敗する）
```
root@e03b1b5dae16:/var/data# ansible-playbook -i inventry.ini playbook.yml --ask-vault-pass -e @vaulted_vars.yaml 
Vault password: 

PLAY [targets] ****************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************
fatal: [target-no-python-ubuntu]: FAILED! => {"ansible_facts": {}, "changed": false, "failed_modules": {"setup": {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "failed": true, "module_stderr": "Shared connection to target-no-python-ubuntu closed.\r\n", "module_stdout": "/bin/sh: 1: /usr/bin/python: not found\r\n", "msg": "The module failed to execute correctly, you probably need to set the interpreter.\nSee stdout/stderr for the exact error", "rc": 127, "warnings": ["No python interpreters found for host target-no-python-ubuntu (tried ['/usr/bin/python', 'python3.7', 'python3.6', 'python3.5', 'python2.7', 'python2.6', '/usr/libexec/platform-python', '/usr/bin/python3', 'python'])"]}}, "msg": "The following modules failed to execute: setup\n"}
ok: [target-ubuntu]
ok: [target-centos]

TASK [targets test] ***********************************************************************************************************************************************
[WARNING]: Consider using the file module with state=touch rather than running 'touch'.  If you need to use command because file is insufficient you can add
'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.
changed: [target-ubuntu]
changed: [target-centos]

PLAY [centos] *****************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************
ok: [target-centos]

TASK [centos test] ************************************************************************************************************************************************
changed: [target-centos]

PLAY [ubuntu] *****************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************
ok: [target-ubuntu]

TASK [ubuntu test] ************************************************************************************************************************************************
changed: [target-ubuntu]

PLAY RECAP ********************************************************************************************************************************************************
target-centos              : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target-no-python-ubuntu    : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
target-ubuntu              : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

6. target-centos, target-ubuntuに再接続し、targets, centos, ubuntu ファイルが追加されていることを確認
```
ssh target-centos
ls # targets, centosファイルができる
exit

ssh target-ubuntu
root # enter password
ls # targets, ubuntuファイルができる
exit
```

## 暗号化ファイルの作成方法
1. 暗号化したいファイルを作成する
```
ansible-vault create vaulted_vars.yaml 
```
2. ファイルに暗号化したい内容を記述し保存する
```
ubuntu_pass: root
np_ubuntu_pass: root
```
3. 引数に与えて実行する
```
ansible-playbook -i inventry.ini playbook.yml --ask-vault-pass -e @vaulted_vars.yaml 
```

****

# Ansible on Docker
This is a tutorial for testing Ansible at CentOS and Ubuntu by using Docker.

## What we will try 3 things heew
- Execute ansible on multiple OS: CentOS and Ubuntu
- Confirm target node can't run without python
- Try to use ansible vault which helps encrpt confidential information

**Ansible can't run without python. Ansible is agent-less tool(you don't need to install ansible on target node), but target node should be able to run python. Be ware to use a machine which can't install python like network router **

## OS used at this tutorial
- CentOS
- Ubuntu(x2)
  - One has python, the other is deleted python
  - ubuntu can uninstall python because ubuntu uses Apt for package manager which is not depend on python.(yum is depend on python)

## Testing environment(local)
- MacOS Monterey 12.0.1
- Docker Desktop 4.3.2 (72729)


## Directory structure
```
.
├── README.md
├── docker
│   ├── ansible
│   │   └── Dockerfile      Ansible master node DockerFile
│   ├── target-centos
│   │   └── Dockerfile      Target node DockerFile(OS: CentOS)
│   ├── target-no-python-ubuntu
│   │   └── Dockerfile      Target node DockerFile(OS: ubuntu without python)
│   └── target-ubuntu
│       └── Dockerfile      Target node DockerFile(OS: ubuntu with python)
├── docker-compose.yml      docker-compose file
├── inventry.ini            for Ansible inventry file
├── playbook.yml            for Ansible playbook file
└── vaulted_vars.yaml       for Ansible password encrypted file

```

## How to use
1. Start Docker containter
```
docker-compose up -d
```
2. Connect to Ansible container via SSH
```
docker exec -it ansible /bin/bash
```
3. Confirm SSH connection to targets.
    1. Connect to centos, ubuntu
        ```
        ssh target-centos  # type yes to connect
        exit
        ssh target-ubuntu    # type yes to connect
        root    # Enter root to login
        python3 -V  # pythonのバージョンが表示されることを確認
        exit
        ```
    2. Connect to ubuntu without python
        ```
        ssh target-no-python-ubuntu   # type yes to connect
        root    # Enter root to login
        python3 -V  # confirm python is not installed
        exit
        ```


4. Execute ansible command to target node.
```
ansible-playbook -i inventry.ini playbook.yml --ask-vault-pass -e @vaulted_vars.yaml

# --ask-vaul-pass option means asking password to decrypt vault file 
# -e @<filename> option can pass variable file.

pass # Vault password: pass
```

5. Check log（fail if no python environment）
```
root@e03b1b5dae16:/var/data# ansible-playbook -i inventry.ini playbook.yml --ask-vault-pass -e @vaulted_vars.yaml 
Vault password: 

PLAY [targets] ****************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************
fatal: [target-no-python-ubuntu]: FAILED! => {"ansible_facts": {}, "changed": false, "failed_modules": {"setup": {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "failed": true, "module_stderr": "Shared connection to target-no-python-ubuntu closed.\r\n", "module_stdout": "/bin/sh: 1: /usr/bin/python: not found\r\n", "msg": "The module failed to execute correctly, you probably need to set the interpreter.\nSee stdout/stderr for the exact error", "rc": 127, "warnings": ["No python interpreters found for host target-no-python-ubuntu (tried ['/usr/bin/python', 'python3.7', 'python3.6', 'python3.5', 'python2.7', 'python2.6', '/usr/libexec/platform-python', '/usr/bin/python3', 'python'])"]}}, "msg": "The following modules failed to execute: setup\n"}
ok: [target-ubuntu]
ok: [target-centos]

TASK [targets test] ***********************************************************************************************************************************************
[WARNING]: Consider using the file module with state=touch rather than running 'touch'.  If you need to use command because file is insufficient you can add
'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.
changed: [target-ubuntu]
changed: [target-centos]

PLAY [centos] *****************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************
ok: [target-centos]

TASK [centos test] ************************************************************************************************************************************************
changed: [target-centos]

PLAY [ubuntu] *****************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************
ok: [target-ubuntu]

TASK [ubuntu test] ************************************************************************************************************************************************
changed: [target-ubuntu]

PLAY RECAP ********************************************************************************************************************************************************
target-centos              : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target-no-python-ubuntu    : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
target-ubuntu              : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

6. Connect to target-centos, target-ubuntu again to check wheather  targets, centos, ubuntu file is added or not
```
ssh target-centos
ls # targets, centos was created
exit

ssh target-ubuntu
root # enter password
ls # targets, ubuntu was created
exit
```

## How to make encripted file
1. Create file to encript
```
ansible-vault create vaulted_vars.yaml 
```
2. Write contents to encript and save
```
ubuntu_pass: root
np_ubuntu_pass: root
```
3. Execute
```
ansible-playbook -i inventry.ini playbook.yml --ask-vault-pass -e @vaulted_vars.yaml 
```


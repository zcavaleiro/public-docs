# Notas: configurações e resolução de problemas de comunicações SSH entre cliente e servidor.

## Tabela de conteúdos
1. [Porquê SSH e este documento?](#porquê-ssh-e-este-documento)
2. [Os diferentes ficheiros de configurações SSH.](#os-diferentes-ficheiros-de-configurações-ssh)
3. [Permissões do directório .ssh e seus ficheiros.](#permissões-do-directório-ssh-e-seus-ficheiros)
4. [Resolução de erros comuns de autenticação ou comunicação.](#resolução-de-erros-comuns-de-autenticação-ou-comunicação)
5. [Ordem de Operações.](#ordem-de-operações)
6. [Configurações do cliente SSH.](#configurações-do-cliente-ssh)
    1. [Exemplo de um ficheiro de configuração shh de um user.](#exemplo-de-um-ficheiro-de-configuração-shh-de-um-user)
    2. [Exemplo do ficheiro de configuração do sistema do SSH](#exemplo-do-ficheiro-de-configuração-do-sistema-do-ssh)
7. [Configurações do servidor SSH.](#configurações-do-servidor-ssh)
    1. [Serviço do servidor SSH.](#serviço-do-servidor-ssh)
    2. [Exemplo de um ficheiro de configuração shhd de um servidor de SSH.](#exemplo-de-um-ficheiro-de-configuração-shhd-de-um-servidor-de-ssh)
    3. [Estrutura do directório /etc/ssh.](#estrutura-do-directório-etcssh)
8. [Outros.](#outros)
9. [Bibliografia.](#bibliografia)
    1. [Docs.](#docs)
    2. [Vids.](#vids)

<br>

## Porquê SSH e este documento?
1. SSH (Secure Shell Protocol), é uma forma de comunicação com segurança em mente que permite, por exemplo, acessos remotos entre computadores, servidores, etc. Este protocolo de comunicação fornece também uma robusta autenticação e protecção durante a comunicação de dados com uma forte encriptação.
2. OpenSSH (OpenBSD Secure shell), é um conjunto de programas com a função de gerir e manter segurança da rede com base no SSH. É de tal forma popular que por exemplo, é um standard entre formas consideradas seguras (propriamente configurada e mantida) a nível de desenvolvimento profissional de Tecnologias de Informação. De momento, 2024, a maioria dos sistemas operativos usam o OpenSSH.
3. Benefícios e usos:
    - Acesso remoto e seguro a recursos.
    - Encriptação de comunicações para diferentes usos.
    - Execução de comandos de forma remota (updates, patches, manutenção)
    - É uma das bases comunicações em IT, por exemplo em automação utilizada em DevOps.
    - Etc.

4. Este documento não é um guia ou tutorial de SSH. São notas pessoais de acordo com o título para máquinas Linux.  
Se tivermos que lidar de vez em quando com configurações e comunicações básicas em SSH, é fácil esquecer certos conceitos, configurações, etc. Como a memória nem sempre pode estar presente, resolvi escrever em formato de notas para ser mais fácil a consulta ou "reconsulta".   
Optei pela língua portuguesa por diversas razões. Uma delas é porque em  inglês já existe muita documentação acerca do tema. Assim sendo, fica também como tentativa de que possa ser útil à comunidade de língua Portuguesa

<br>

## Os diferentes ficheiros de configurações SSH

1. O ficheiro de configuração do ***ssh daemon server***, no caso de ser um servidor SSH. Este ficheiro, específica como um cliente ssh se vai conectar/ligar a este servidor:
    - `etc/ssh/sshd_config`

2. ***Opções nos comandos SSH*** do lado do cliente. É possivel incluir opções em formato de strings quando fazemos a ligação. Existem muitas possibilidades:
    - `ssh -i ~/.ssh/mykey user@host` A opção `-i` serve para definir qual a chave a usar. Com `-o` é também possivel definir `IdentityFile` tal como `-i`. Outras opções podem ser consultadas na documentação ou *man page* do ssh_config. Mais exemplos para as opções, `-o`:
      - `PreferredAuthentications`
      - `PasswordAuthentication`
      - `PubkeyAuthentication`
      - `IdentitiesOnly`
    - `ssh username@remote_host command_to_run`

3. ***Personalizações do utilizador*** (cliente) nas configurações SSH:
    - `~/.ssh/config`
      - Caso mais comum de uma só utilizador/entidade.
    - `~/.sshconfig.d`
      - É um directório que contem multiplos ficheiros de configurações SSH.
      - Para o caso em que é necessário gerir diferentes ficheiros de configuração para diferentes sistemas ou tipos de sistemas, por exemplo, por grupos .

4. O ficheiro de ***configuração do sistema*** da parte do cliente SSH. Configura cada ligação e o fluxo de dados que saí do sistema do cliente SSH.
    - Da mesma forma que um user pode ter as suas necessidades de configuração ou personalização de opções com o ssh, o próprio sistema operativo também pode ter as suas próprias opções. A sua localização, por norma é:
      - `/etc/ssh/ssh_config`
      - `/etc/ssh/ssh_config.d`

<br>

## Permissões do directório .ssh e seus ficheiros

A meu ver, as permissoes do directório `.ssh` e seus ficheiros devem ser as seguintes:


|  ficheiro/diretório  | descrição | permissões |
|         :---         |    :---:  |    ---:    |
|  `.ssh` | o diretório  | `700`, `drwx------`|
|    chaves privadas   | as chaves privadas | `600`, `-rw-------` |
|    chaves públicas   | as chaves públicas | `644`, `-rw-r--r--` |
| `config`  |  ficheiro personalização |  `600`, `-rw-------` |
| `known_hosts`|  guarda chaves pub de servidores a aceder |  `644`, `-rw-r--r--`  |
| `authorized_keys` | define que chaves podem aceder ao sistema | `600`, `-rw-------` |
| `/home` | as permissões não devem ser maiores que | `755`, `drwxr-xr-x` |

<br>

## Resolução de erros comuns de autenticação ou comunicação
  - Verificar que entre ponto A e B existe conexão. Com `ping` por exemplo.
  - Verificar que temos o cliente de SSH instalado do nosso lado. Normalmente vem incluido no OS.
  - Verificar qual a porta de conexão do lado do servidor, pode ser diferente da 22. Verificar também o firewall caso a porta seja random e esteja bloqueada, é muito frequente quando erro devolvido é de timeout.
  - Rever as permissões do directório `.ssh` e seus ficheiros.
  - Há um limite no número de tentativas de autenticação até que o servidor SSH devolva falha de autenticação. `Received disconnect from 10.10.10.10 port 22:2: Too many authentication failures` É exemplo disso. Significa que o cliente tentou autenticar para cada método e cada chave e acabou por ser desligado da conexão. No `sshd_config` podemos configurar o parametro `MaxAuthTries`. O *default* é 6. Se eu tiver uma chave e password como método de autenticação e tiver mais de 5 chaves a serem verificadas, cada chave é verificada até ser desligado por atingir o limite de tentativas. Se eu souber que necessito autenticação por password, posso usar `PubkeyAuthentication=no` ou `PrefferedAuthentication=password ` para garantir que o meu terminal chegue à opção de introduzir a password. Se tiver uma chave específica para usar, posso definir essa chave como `IdentiesOnly=yes` para assim só essa key ser tentada.
      ```bash
      ssh-copy-id -i ~/.ssh/id_somehubs -o PreferredAuthentications=password user@host
      ```
  - Usar o modo verboso ao fazer a ligação.
  - Verificar os logs da comunicação entre cliente e servidor (em janelas ou separadores diferentes).
    - no servidor SSH, em Ubuntu `/var/log/auth.log`, em Fedora `var/log/secure`
    - o cliente ao tentar-se ligar ao servidor vai permitir ver nos logs do servidor o que acontece.
    - A opção mais actual (os com sistemd) será ver os logs no journalctl `journalctl -u ssh` ou `sshd`, conforme OS. Também pode ser útil seguir os logs em tempo real com a opção `-f` de *follow*, `journalctl -fu sshd`  
  - Mais, por exemplo, em [Wow to Troubleshoot SSH Connectivity Issues , da DigitalOcean.](https://docs.digitalocean.com/support/how-to-troubleshoot-ssh-connectivity-issues/)

<br>

## Ordem de Operações
1. As opções definidas na linha de comandos são processadas primeiro.
   - Se forem especificadas na linha de comandos, essas opções nos ficheiros de configuração vão ser ignoradas.
   - Numa perspectiva de segurança, o servidor de SSH é que deve estabelecer os parametros requiridos para a forma de conexão por SSH. Devendo dizer aos seus utilizadores como aceder ao seu sistema.
2. Se não forem definidas opções na linha de comandos, as opções no ficheiro de configuração do **utilizador** são processadas.
3. Se não houver opções especificadas nos primeiros dois pontos, a configuração do ficheiro de sistema SSH vai ser processado.

<br>

## Configurações do cliente SSH
### Exemplo de um ficheiro de configuração shh de um user.

  - Para não termos que decorar e escrever longos comandos, é preferível manter um ficheiro local de configuração que identifica as chaves e outras opções para cada destino. Por norma, o utilizador cria ou usa o ficheiro `~/.ssh/config`. Uma boa forma de começar esse ficheiro pode ser copiando um exemplo do `ssh_config` que está em `/etc/ssh`

    ```shell
    cp /etc/ssh/ssh_config ~/.ssh/config
    ```    
  - Exemplo do ficheiro `.ssh/config`:

    ```shell
    # Forma de definir regras/opções genéricas SSH
    Host *
      Ciphers aes256-gcm@openssh.com,chacha20-poly1305@openssh.com
      KexAlgorithms curve25519-sha256@libssh.org
      PasswordAuthentication no
      PreferredAuthentications publickey
      #StrictHostKeyChecking yes

    # Exemplos por host
    Host ubuntu
      Hostname ec2-22-22-33-44-compute-1.amazonaws.com
      User ec2-user
      IdentityFile ~/.ssh/key1.pem
      IdentiesOnly yes

    Host *fedoraproject.org *fedorapeople.org *pagure.io
      User lookitup
      IdentityFile ~/.ssh/id_IKnowWhichKey
      IdentitiesOnly yes

    Host *.testlab.example.com 192.168.1.101 
      Hostname 192.168.1.101
      User dev
      IdentityFile ~/.ssh/id_mylab
      IdentityFile ~/.ssh/id_my_other_key
      #PreferredAuthentications publickey, password
      PreferredAuthentications password
      StrictHostKeyChecking no
      #Port 10001

    #Include ~/.ssh/config.d/config-jogos
    ```

    - A parte `Host *`, aplica-se a configuração geral.
    - As configurações específicas por host no arquivo .ssh/config sobrepõem-se às configurações gerais. Ou seja têm prioridade às configurações definidas em `Host *`. Exemplo, `PreferredAuthentications password`.
    - O `Include` do directório `~/.ssh/config.d/config-jogos` é um exemplo. O `path` do diretório pode ser outro qualquer. Por questões de organização é comum ser semelhante a este exemplo.

<br>

### Exemplo do ficheiro de configuração do sistema do SSH
- No sistema operativo, existe também um ficheiro de configuração do cliente SSH, o `/etc/ssh/ssh_config`:


  ```shell
  #       $OpenBSD: ssh_config,v 1.36 2023/08/02 23:04:38 djm Exp $

  # This is the ssh client system-wide configuration file.  See
  # ssh_config(5) for more information.  This file provides defaults for
  # users, and the values can be changed in per-user configuration files
  # or on the command line.

  # Configuration data is parsed as follows:
  #  1. command line options
  #  2. user-specific file
  #  3. system-wide file
  # Any configuration value is only changed the first time it is set.
  # Thus, host-specific definitions should be at the beginning of the
  # configuration file, and defaults at the end.

  # Site-wide defaults for some commonly used options.  For a comprehensive
  # list of available options, their meanings and defaults, please see the
  # ssh_config(5) man page.

  # Host *
  #   ForwardAgent no
  #   ForwardX11 no
  #   PasswordAuthentication yes
  #   HostbasedAuthentication no
  #   GSSAPIAuthentication no
  #   GSSAPIDelegateCredentials no
  #   GSSAPIKeyExchange no
  #   GSSAPITrustDNS no
  #   BatchMode no
  #   CheckHostIP no
  #   AddressFamily any
  #   ConnectTimeout 0
  #   StrictHostKeyChecking ask
  #   IdentityFile ~/.ssh/id_rsa
  #   IdentityFile ~/.ssh/id_dsa
  #   IdentityFile ~/.ssh/id_ecdsa
  #   IdentityFile ~/.ssh/id_ed25519
  #   Port 22
  #   Ciphers aes128-ctr,aes192-ctr,aes256-ctr,aes128-cbc,3des-cbc
  #   MACs hmac-md5,hmac-sha1,umac-64@openssh.com
  #   EscapeChar ~
  #   Tunnel no
  #   TunnelDevice any:any
  #   PermitLocalCommand no
  #   VisualHostKey no
  #   ProxyCommand ssh -q -W %h:%p gateway.example.com
  #   RekeyLimit 1G 1h
  #   UserKnownHostsFile ~/.ssh/known_hosts.d/%k
  #
  # This system is following system-wide crypto policy.
  # To modify the crypto properties (Ciphers, MACs, ...), create a  *.conf
  #  file under  /etc/ssh/ssh_config.d/  which will be automatically
  # included below. For more information, see manual page for
  #  update-crypto-policies(8)  and  ssh_config(5).
  Include /etc/ssh/ssh_config.d/*.conf
  ```
  - Todos os parâmetros que não estejam definidos no ficheiro de config do host `.ssh/config`, vão usar as configurações definidas neste ficheiro de configuração do sistema SSH no cliente `/etc/ssh/ssh_config`.
  - As configurações deste ficheiro `/etc/ssh/ssh_config` não se sobrepõem às configurações do ficheiro `.ssh/config`. 
  - Ou seja, as configurações definidas em `.ssh/config` têm prioridade sobre as configurações globais definidas em `/etc/ssh/ssh_config`.
  

<br>

## Configurações do servidor SSH

### Serviço do servidor SSH
- A maioria das distrbuições da família RHEL usam o binário `sshd` para este efeito e da família Debian o `ssh`.
  ```shell
  which sshd
  # /usr/sbin/sshd
  ```
- O sshd, *ssh deamon*, é um serviço que corre em *background* e aceita conexões.
- Para verificarmos se está activo e é iniciado no startup:
  
  ```shell
  sudo systemctl status sshd

  # devemos ter um output semelhante a 
  [root@192 ~]# systemctl status sshd.service
  ● sshd.service - OpenSSH server daemon
    Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
    Active: active (running) since Fri 2021-06-29 11:43:45 PDT; 5min 33s ago
      Docs: man:sshd(8)
            man:sshd_config(5)
  Main PID: 1348 (sshd)
      Tasks: 1 (limit: 100917)
    Memory: 2.2M
    CGroup: /system.slice/sshd.service
            └─1288 /usr/sbin/sshd -D -oCiphers=aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes256-ctr,aes256-cbc,aes128-gcm@openssh.com,aes128-c>

  May 03 05:34:56 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
  May 03 05:34:56 localhost.localdomain sshd[1348]: Server listening on 0.0.0.0 port 22.
  May 03 05:34:56 localhost.localdomain sshd[1348]: Server listening on :: port 22.
  May 03 05:34:56 localhost.localdomain systemd[1]: Started OpenSSH server daemon.
  [root@192 ~]#

  ```
    - Podemos verificar que está `active`, `enabled` e a usar a porta 22.
    - Para ativarmos o serviço: `sudo systemctl start sshd`
    - Para reiniciarmos o serviço: `sudo systemctl restart sshd`
    - Para paramos o serviço: `sudo systemctl stop sshd`
    - Para termos o serviço iniciado no arranque: `sudo systemctl enable ssh`
    - Relembrar que em Debian, Ubuntu o binário é chamado `ssh`, em RHEL, Fedora é `sshd`

<br>

### Exemplo de um ficheiro de configuração shhd de um servidor de SSH


- Exemplo, `sudo cat /etc/ssh/sshd_config`:
  ```shell
  #       $OpenBSD: sshd_config,v 1.104 2021/07/02 05:11:21 dtucker Exp $

  # This is the sshd server system-wide configuration file.  See
  # sshd_config(5) for more information.

  # This sshd was compiled with PATH=/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin

  # The strategy used for options in the default sshd_config shipped with
  # OpenSSH is to specify options with their default value where
  # possible, but leave them commented.  Uncommented options override the
  # default value.

  # To modify the system-wide sshd configuration, create a  *.conf  file under
  #  /etc/ssh/sshd_config.d/  which will be automatically included below
  Include /etc/ssh/sshd_config.d/*.conf

  # If you want to change the port on a SELinux system, you have to tell
  # SELinux about this change.
  # semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
  #
  #Port 22
  #AddressFamily any
  #ListenAddress 0.0.0.0
  #ListenAddress ::

  #HostKey /etc/ssh/ssh_host_rsa_key
  #HostKey /etc/ssh/ssh_host_ecdsa_key
  #HostKey /etc/ssh/ssh_host_ed25519_key

  # Ciphers and keying
  #RekeyLimit default none

  # Logging
  #SyslogFacility AUTH
  #LogLevel INFO

  # Authentication:

  #LoginGraceTime 2m
  #PermitRootLogin prohibit-password
  #StrictModes yes
  #MaxAuthTries 6
  #MaxSessions 10

  #PubkeyAuthentication yes

  # The default is to check both .ssh/authorized_keys and .ssh/authorized_keys2
  # but this is overridden so installations will only check .ssh/authorized_keys
  AuthorizedKeysFile      .ssh/authorized_keys

  #AuthorizedPrincipalsFile none

  #AuthorizedKeysCommand none
  #AuthorizedKeysCommandUser nobody

  # For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
  #HostbasedAuthentication no
  # Change to yes if you don't trust ~/.ssh/known_hosts for
  # HostbasedAuthentication
  #IgnoreUserKnownHosts no
  # Don't read the user's ~/.rhosts and ~/.shosts files
  #IgnoreRhosts yes

  # To disable tunneled clear text passwords, change to no here!
  #PasswordAuthentication yes
  #PermitEmptyPasswords no

  # Change to no to disable s/key passwords
  #KbdInteractiveAuthentication yes

  # Kerberos options
  #KerberosAuthentication no
  #KerberosOrLocalPasswd yes
  #KerberosTicketCleanup yes
  #KerberosGetAFSToken no
  #KerberosUseKuserok yes

  # GSSAPI options
  #GSSAPIAuthentication no
  #GSSAPICleanupCredentials yes
  #GSSAPIStrictAcceptorCheck yes
  #GSSAPIKeyExchange no
  #GSSAPIEnablek5users no

  # Set this to 'yes' to enable PAM authentication, account processing,
  # and session processing. If this is enabled, PAM authentication will
  # be allowed through the KbdInteractiveAuthentication and
  # PasswordAuthentication.  Depending on your PAM configuration,
  # PAM authentication via KbdInteractiveAuthentication may bypass
  # the setting of "PermitRootLogin prohibit-password".
  # If you just want the PAM account and session checks to run without
  # PAM authentication, then enable this but set PasswordAuthentication
  # and KbdInteractiveAuthentication to 'no'.
  # WARNING: 'UsePAM no' is not supported in Fedora and may cause several
  # problems.
  #UsePAM no

  #AllowAgentForwarding yes
  #AllowTcpForwarding yes
  #GatewayPorts no
  #X11Forwarding no
  #X11DisplayOffset 10
  #X11UseLocalhost yes
  #PermitTTY yes
  #PrintMotd yes
  #PrintLastLog yes
  #TCPKeepAlive yes
  #PermitUserEnvironment no
  #Compression delayed
  #ClientAliveInterval 0
  #ClientAliveCountMax 3
  #UseDNS no
  #PidFile /var/run/sshd.pid
  #MaxStartups 10:30:100
  #PermitTunnel no
  #ChrootDirectory none
  #VersionAddendum none

  # no default banner path
  #Banner none

  # override default of no subsystems
  Subsystem       sftp    /usr/libexec/openssh/sftp-server

  # Example of overriding settings on a per-user basis
  #Match User anoncvs
  #       X11Forwarding no
  #       AllowTcpForwarding no
  #       PermitTTY no
  #       ForceCommand cvs server
    
  ```
  - Estas são as regras que o nosso servidor vai operar como forma de resposta ao cliente que se está a conectar.
  - Algumas das principais regras e boas práticas a implementar em primeiro lugar são:
     - desligar a permissão fazer login como root (assegurar outra conta admin no sistema.)
     - não permitir autenticação por password, é fundamental.
  
<br>

### Estrutura do directório /etc/ssh

- No lado do servidor SSH, os ficheiros em `/etc/ssh`:

  ```shell
  user@fedora:~$ ls -al /etc/ssh
  total 616
  drwxr-xr-x. 1 root root    104 jul  2 01:00 .
  drwxr-xr-x. 1 root root   4660 ago 18 11:47 ..
  -rw-r--r--. 1 root root 620105 jul  2 01:00 moduli
  -rw-r--r--. 1 root root   1916 jul  2 01:00 ssh_config
  -rw-------. 1 root root    670 jul  2 03:00 ssh_host_ecdsa_key
  -rw-r--r--. 1 root root    170 jul  2 03:00 ssh_host_ecdsa_key.pub
  -rw-------. 1 root root    430 jul  2 03:00 ssh_host_ed25519_key
  -rw-r--r--. 1 root root     80 jul  2 03:00 ssh_host_ed25519_key.pub
  -rw-------. 1 root root   2570 jul  2 03:00 ssh_host_rsa_key
  -rw-r--r--. 1 root root    670 jul  2 03:00 ssh_host_rsa_key.pub
  drwxr-xr-x. 1 root root     28 jul  2 01:00 ssh_config.d
  -rw-------. 1 root root   3670 jul  2 01:00 sshd_config
  drwx------. 1 root root     88 jul  2 01:00 sshd_config.d
  ```
    - Notar que as chaves ssh presentes são usadas para o *fingerprint* quando os clientes se ligam a este servidor. São as chaves identificativas do servidor de SSH. Não queremos modificar estas chaves, caso contrário podemos ficar desligados do server para sempre.
    - Notar que ao clonar, ou usar uma imagem por exemplo como o OS com as chaves iguais ao exemplo, estas chaves vao ser copiadas também e vai criar problemas de rede. É necessário criar chaves novas por servidor em clones ou réplicas.
  
<br>

## Outros

1. ***ssh-agent*** e ***passphrase*** 
    -  No caso de as chaves SSH terem *passphrase*, está é necessária ser introduzida manualmente cada vez que se realiza uma comunicação. Isto pode ser entediante. Uma das formas de "automatizar", pelo menos enquanto está o terminal ou a sessao SSH aberta, é usando o ssh-agent.
      - o ssh-agent é um mecanismo que guarda a chave e opções em memória para nos facilitar a conexão.
      - as versões de linux destop's vêm com o ssh-agent por default. As distribuições "server editions", sem GUI, podem não ter o ssh-agent instalado. 
      - Para o caso dos servidores sem ssh-agent:


        ```shell
        # verificamos que o ssh-agent não está ativo
        ps aux | grep ssh-agent

        # deve devolver apenas uma linha do próprio comando grep
        ```

        ```shell
        eval "$(ssh-agent)" 

        # vai iniciar o ssh-agent e devolver o pid, algo como:
        # Agent pid 24356
        ```
        Agora ao procurarmos pelo *agent* novamente:  
        ```shell
        # devemos ter duas linhas de entrada, com um novo pid do ssh-agent
        ps aux | grep ssh-agent
        
        ```

        Uma vez ativo, podemos adiconar uma chave ao ssh-agent
        ```shell
        ssh-add ~/.ssh/id_mykey
        ```
          - é a ***private key*** que adicionamos, preenchemos a ***passphrase*** e está feito.
          - enquanto o ssh-agent estiver em execução, não é mais necessário introduzir a passphrase.
          - desligando o terminal ou encerrando a sessão ssh vai fechar o ssh-agent. Será necessário voltar a adicioná-la.

      - Vantagens:
        - Segurança. Uma passphrase e as suas boas práticas adicionam uma camada extra de segurança à chave SSH.
        - Conveniência. O ssh-agent permite usar  uma chave ssh sem ter que inserir a "password" repetidamente. 

      - Desvantagens:
        - Algo complexo para iniciantes.
        - Persistência. Devido à necessidade de ter que re-configurar o ssh-agent após encerrar uma sessão SSH ou o terminal em uso.

      - Possiveis alternativas:
        - Chaves SSH sem passphrase, menos seguras no geral, mas usando chaves usb físicas (FIDO/FIDO2) como as chaves da Yubikeys ou Google Titan, por exemplo.
        - Gestor de passwords. Existem ferramentas como o ***pass***, ***gnome-keyring*** ou ***KDE Wallet***, entre outras, que permitem gerir e usar as credenciais e passphrases.


<br>

## Bibliografia:
### Docs
  - https://en.wikipedia.org/wiki/Secure_Shell
  - https://www.openssh.com/
  - https://docs.digitalocean.com/support/how-to-troubleshoot-ssh-connectivity-issues/
  - https://phoenixnap.com/kb/ssh-config


### Vids
  - [Complete SSH Tutorial: All-in-One Guide for Secure Connections, 
Learn Linux TV](https://www.youtube.com/watch?v=YS5Zh7KExvE)
  - [Simplifying SSH Key Management: Leveraging ssh config for Security and Efficiency, SANS Cyber Defense](https://www.youtube.com/watch?v=ysd7GqBf5U0)

<div dir=RtL> 
:Autor da tradução</br>
<i>Zito Cavaleiro  </i>-   email: zcavaleiro AT protonmail DOT com</br>
Professor de Matemática e Ciências da Natureza</br>
Engenheiro de Automação de Tecnologias de Informação</div>

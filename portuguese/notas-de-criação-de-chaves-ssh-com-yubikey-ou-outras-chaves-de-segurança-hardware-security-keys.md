# Notas de criação de chaves SSH com Yubikey ou outras chaves de segurança, *hardware security keys*.


## Tabela de conteúdos

- [Breve introdução sobre chaves de segurança de Hardware.](#breve-introdução-sobre-chaves-de-segurança-de-hardware)  
    - [Algumas das vantagens e desvantagens](#algumas-das-vantagens-e-desvantagens)
- [Configurações iniciais.](#configurações-iniciais)
- [Porquê usar chaves SSH com hardware security keys](#porquê-usar-chaves-ssh-com-hardware-security-keys)
- [Criar chaves SSH "Non Resident" com uma yubikey.](#criar-chaves-ssh-non-resident-com-uma-yubikey)
    - [Mais simples, com verificação de tocar na chave.](#mais-simples-com-verificação-de-tocar-na-chave)
    - [Com verificação de pin necessária.](#com-verificação-de-pin-necessária)
    - [Sem tocar na chave para autenticação.](#sem-tocar-na-chave-para-autenticação)
- [Cópia da chave pública SSH para um dispositivo remoto](#cópia-da-chave-pública-ssh-para-um-dispositivo-remoto)
- [Criar chaves SSH "Resident" com uma yubikey.](#criar-chaves-ssh-resident-com-uma-yubikey)
    - [Listar as chaves da Yubikey](#listar-as-chaves-da-yubikey)
    - [Extrair uma resident key da Yubikey para um novo sistema.](#extrair-uma-resident-key-da-yubikey-para-um-novo-sistema)
    - [Apagar uma Resident key de uma Yubikey](#apagar-uma-resident-key-de-uma-yubikey)
- [Outros.](#outros)
- [Bibliografia.](#bibliografia)
    - [Docs.](#docs)
    - [Vids.](#vids)

<br>


## Breve introdução sobre chaves de segurança de Hardware
As chaves de segurança de Hardware, *hardware security keys* ou muitas vezes chamadas de *security keys*, são dispositivos físicos de autenticação que, na maioria dos casos, permitem o armazenamento e gestão de credenciais de vários protocolos de encriptação.  

Estes dispositivos fornecem um dos acessos mais seguros a sistemas, aplicações, serviços web, etc, com base na autenticação de dois fatores (2FA) ou autenticação multifactor (MFA) e o dispositivo físico.

São dispositivos que permitem uma autenticação física e que fortalecem grandemente a proteção de contas online e comunicações. Estas chaves de segurança utilizam uma combinação de criptografia e autenticação física tornando o acesso não autorizado extremamente difícil.  

Adicionam uma camada extra de segurança às *passwords*, gestores de *passwords* tradicionais e autenticação de dois factores. Podemos comparar estas chaves de segurança como mini ou pessoais [Hardware Security Modules (HSM)](https://en.wikipedia.org/wiki/Hardware_security_module), usados a nível empresarial devido ao seu poder para criptografia, gestão de credenciais e autenticação. 



### Algumas das vantagens e desvantagens

***Vantagens:***
  - Facilidade de uso e forte segurança. Compatíveis com diversos dispositivos e sistemas operativos, em muitos casos não é necessário escrever o nome do utilizador nem a password para fazer login. Basta conectar a chave e tocar para autenticar.

  - Com estas chaves é possível autenticar a pessoa que está a aceder pois é exigida uma prova física de posse.

  - Alternativa superior a credenciais guardadas em telemóveis, smartphones ou outros dispositivos que podem ter um nível de risco a falhas superior a esta chaves de hardware. Acrescenta a portabilidade, isolamento e segurança física das credenciais.

  - Não necessita de ligação à internet ou dados móveis. Eliminam também a necessidade de depender de métodos menos seguros como as SMS.

  - Este tipo de chaves são conhecidas pela sua durabilidade e consistência. São resistentes a danos físicos e muito raramente têm falhas técnicas.

  - São também resistentes ao *phishing*. A autenticação de dois fatores necessita do factor físico de autenticação, não apenas de software. Mesmo que um utilizador seja redirecionado para um site malicioso, as credenciais de acesso não vão funcionar pois a Yubikey não está configurada para o site ilegítimo.


***Desvantagens:***
  - Ao contrário de soluções de software, o dispositivo físico tem um custo. De momento existem  opções válidas com preços a rondar entre 25€ a 55€. São necessárias sempre **duas** chaves ao considerar usar esta tecnologia.  

  - Apesar de existirem diversas soluções de conectividade, usb, usb-c, NFC, lightning, alguns dispositivos podem não funcionar corretamente.

  - Utilizadores com menos conhecimentos informáticos podem ter receio de configurar as suas chaves e contas e podem necessitar de ajuda.

  - A implementação, lista de exemplos [2fa.directory](https://2fa.directory/pt/), ainda não é geral na web, aplicações ou dispositivos. Esta implementação também não é consistente entre Apple, Microsoft ou Google por exemplo. Há ligeiras diferenças na implementação.

  - Em caso de perda ou roubo, os acessos e/ou credenciais que estão na chave, podem ficar perdidos para sempre se uma outra chave de backup com semelhantes acessos e credenciais não tiver sido criada. Neste caso, é aconselhável remover a respectiva chave dos acessos garantidos e acrescentar novas chaves/credenciais. Dependendo do nível da ameaça (*threat model*) devemos considerar que uma chave física de hardware se for roubada, mesmo protegida por pin ou password, pode  possivelmente ser clonada ou extraída informação protegida por algumas entidades no presente ou no futuro. 


<br>


## Configurações iniciais de uma Yubikey

- As chaves Yubikeys necessitam ser configuradas inicialmente. Estas vêm com "passwords", "pins", "puks" por padrão que precisam ser alterados pelo seu utilizador e muito provavelmente serão guardados como backup num lugar seguro. Este documento não tem esse objetivo. Recomendo o seguinte vídeo de , [***Ricci Gian Maria***](https://www.codewrecks.com/), para efetuar as configurações iniciais:

  - [I bought a Yubikey now what? Setting pin and configure the key, CodeWrecks](https://www.youtube.com/watch?v=rNtzmgt0Al4)

  - Não é necessário para este documento, mas recomendo também a sua [Yubikey Playlist](https://www.youtube.com/playlist?list=PLn9t_BnhwY0KXIqloOys7cCDFSHJycrDl) para outras funções da Yubikey.

- A aplicação de *frontend* Yubikey Manager, parece estar a ser deprecada pela Yubico e de momento, o Yubico Authenticator for Desktop, para Windows, Mac ou Linux, veio incluir o que o Yubikey Manager fazia e unificar as duas aplicações. Desde a versão 7.0 do Yubico Authenticator que este é o programa aconselhado para configurar e usar as Yubikeys.  

  - https://www.yubico.com/products/yubico-authenticator/#h-download-yubico-authenticator

- Depois destas configurações e backups feitas, podemos usar um dos benefícios destas *security keys* que é acrescentar uma camada extra de segurança nas chaves e sessões de SSH.

<br>


## Porquê usar chaves SSH com hardware security keys
  - O método tradicional de proteger uma chave SSH privada com uma *password* ou *passphrase*, mesmo que seja robusto, acarreta o risco de esta *password* ou *passphrase* ser usada por atores maliciosos. Isto pode acontecer através de fugas de informação, vaults, gestores de senhas ou com um acesso total ou parcial a ao sistema comprometido. As chaves de segurança de hardware são um incremento neste respeito, pois a chave privada é armazenada externamente no próprio dispositivo de hardware.

  - São resistentes ao *phishing* pois mesmo que um invasor obtenha forma de entrar no sistema, não se consegue autenticar, ou mesmo obtendo acesso ao sistema é mais difícil de enganar o utilizador da chave para que este permitia aceder a áreas críticas pois necessita da autorização com a chave de hardware física. 

  - Estão em conformidade com as últimas políticas de segurança, onde por exemplo é exigido o uso de autenticação multifator (MFA) ou de dois factores (2FA). O seu uso pode ajudar e melhorar essas políticas.

  - Apesar de apresentar diferentes conceitos, a segurança elevada é simples e rápida de usar.

  - Os comandos seguintes, foram testados com Yubikeys com firmware superior a 5.2.3, num sistema operativo Linux da "família" Red Hat em 2024, onde foi instalado o pacote yubikey manager na linha de comandos e as suas dependências. No entanto, a maioria dos comandos abaixo descritos também se aplicam a Windows, Mac ou outras security keys para além da Yubikey.

    ```bash
    user@machine:~/.ssh$ ykman
    bash: ykman: comando não encontrado...
    Instalar pacote "yubikey-manager" para disponibilizar comando "ykman"? [N/y] y
    ```
  
<br>

## Breves diferenças entre chaves SSH discoverable (resident) e chaves SSH non-discoverable (non resident)

-  As chaves ***resident*** ou chaves residentes, são chaves guardadas na Yubikey. É a forma mais segura pois a chave privada nunca é exposta fora do hardware da Yubikey. Útil para cenários que exigem máxima proteção de chaves. 
- Estas são criadas e armazenadas na Yubikey. É considerada a forma mais segura de SSH para uso diferenciado. É mais difícil de gerir entre múltiplos dispositivos e sistemas.

    - Podem ser usadas em ambientes de alta segurança onde a chave privada nunca é exposta aos sistemas, nem como "referência" (handle).

    - Ideal para serviços que contêm dados altamente sensíveis.

    - Equipas de desenvolvimento que lidam com código crítico.

    - Administração de servidores críticos.

    - Desenvolvimento em ambientes seguros.   
  

<br>

- Chaves ***non resident*** ou chaves não residentes, são chaves criadas pela Yubikey, imediatamente disponíveis na máquina/host. Mas onde a chave privada local é uma referência que, entre outros dados, identifica a *master key* da Yubikey para validação/autenticação. 

- A criação da chave privada na máquina local é como um identificador, uma referência à chave privada que vai ser armazenada na Yubikey. Mesmo que "alguém" fique na posse dessa chave privada, precisa da Yubikey física para se autenticar.

  - Apresentam maior flexibilidade para contas que necessitam de autenticação em diferentes dispositivos e serviços.

  - Ideal para o normal "trabalho remoto" e mobilidade entre dispositivos e sistemas.

  -  Útil para cenários onde múltiplos sistemas e serviços necessitam de uma validação de autenticação.

  - Acesso a múltiplos servidores e locais.



<br>


Tabela de comparação de chaves residentes e chaves não residentes com foco na chave privada SSH:

|  Característica  | Chaves Resident | Chaves Non Resident |
|         :---         |    :---  |    :---    |
|  Criação | Geradas e armazenadas na YubiKey local | Geradas na YubiKey, referência mais detalhada no sistema local|
|    Armazenamento  | Exclusivamente na YubiKey | Chave privada na YubiKey, referência no sistema local
Segurança | Alta, chave privada nunca exposta | Boa, mas referência local contém mais dados
Autenticação | YubiKey realiza todas as operações criptográficas | YubiKey necessária para operações criptográficas
Flexibilidade | Menos flexível, maior segurança | Mais flexível,  ligeiro compromisso de segurança



<br>

## Criar chaves SSH "Non Resident" com uma Yubikey


- Ao iniciar a criação da chave com os comandos `ssh-keygen` uma *passphrase* vai nos ser pedida como é comum. Neste caso não a vamos preencher, ou seja, vai ficar vazia. Pressionamos "enter, enter". Isto porque ao contrário das chaves ssh tradicionais, a chave privada local não é a real ou original, pois essa está na Yubikey, "não existem", por norma, vantagens de usar uma *passphrase* (proteção local vs proteção física).


### Mais simples, com verificação de tocar na chave

- É a forma mais comum, esta chave vai solicitar ser tocada para autenticar.

  ```shell
  # windows
  ssh-keygen -t ed25519-sk -f .\.ssh\id_key_sk

  # Linux
  ssh-keygen -t ed25519-sk -f ~/.ssh/id_key_sk 

  # com opção Comment
  ssh-keygen -t ed25519-sk -f ~/.ssh/id_key_sk -C "5Nano_NAME"
  ```
  
  


### Com verificação de pin necessária

- Mais recomendado devido a estar protegida por pin.
  ```shell
  # windows
  ssh-keygen -t ed25519-sk -O verify-required -f .\.ssh\id_key_sk

  # Linux
  ssh-keygen -t ed25519-sk -O verify-required -f ~/.ssh/id_key_sk 

  ```
  - É **Boa Prática**, de momento, ter duas chaves SSH no servidor remoto de yubikeys diferentes. Uma, a primeira, até pode ser protegida por pin (será mais Yubikey de backup) e a outra de uso normal ou de automação. Assim, perdendo uma Yubikey, teremos uma outra de acesso SSH.
  - Para proteção contra uso indevido, a Yubikey fica definitivamente bloqueada ao fim de 8 tentativas com o pin errado.

### Sem tocar na chave para autenticação:

- Menos recomendado, mais para ambientes de testes ou automação local talvez.

  ```shell
  # windows
  ssh-keygen -t ed25519-sk -O no-touch-required -f .\.ssh\id_key_sk

  # Linux
  ssh-keygen -t ed25519-sk -O no-touch-required -f ~/.ssh/id_key_sk 
  ```

<br>


## Cópia da chave pública SSH para um dispositivo remoto

  - Em linux, com a aplicação `ssh-copy-id`

    ```shell
    ssh-copy-id -i ~/.ssh/id_key_sk.pub username@remote_host
    ```

  - Caso seja necessário acrescentar (colar) a chave pública no sistema, depois de entrar por SSH,
    - se `authorized_keys` não existir, criar:
      ```shell
      echo "PUBLICKEYCONTENT" > authorized_keys
      ```
    - se `authorized_keys` existir, acrescentamos (*append*) a nossa chave pública:
      ```shell
      echo "PUBLICKEYCONTENT" >> authorized_keys  
      ```

  - Depois deste passo, devemos poder sair do servidor remoto e voltar a fazer login por SSH, indicando a chave de acesso SSH, sem pedir password.
    
    ```shell
    # exemplo Linux
    ssh USER@SERVERIP -i ~/.ssh/id_key_sk

    # Especificando uma chave presente configurada em `~/.ssh/config`.
    ssh USER@SERVERIP -i ~/.ssh/id_key_sk 
    ```
  
  - Para não ser necessário estar sempre a identificar qual a chave a usar e melhor gerir as ligações SSH, podemos criar um ficheiro `config` na pasta `~/.ssh` do tipo:
  
    ```shell
    HOST 10.0.0.123 # ou omeuservidor.com
      Hostname omeuserver # opcional, para tornar o comando ssh omeuserver possível
      User utilizador
      IdentityFile ~/.ssh/id_key_sk

     HOST 10.0.0.124 
      User utilizador
      Port 2022
      IdentityFile ~/.ssh/id_key_sk
      IdentitiesOnly yes  
    ```  
  - Posteriormente, podemos efetuar a ligação por SSH sem ter que escrever toda a informação.

    ```shell
    ssh omeuserver

    # ou

    ssh 10.0.0.124
    ```

<br>

## Criar chaves ssh "Resident" com uma Yubikey

- Para criar as chaves residentes:

  ```shell
  # windows modo admin
  ssh-keygen -t ed25519-sk -O resident -O verify-required -O application=ssh:5nano -f .\.ssh\id_5nano_sk -C "5nano Resident"

  # Linux
  ssh-keygen -t ed25519-sk -O resident -O verify-required -O application=ssh:5nano -f ~/.ssh/id_5nano_sk -C "5nano Resident"

  # seguintes comandos por testar:
  # outro exemplo - ssh sem verificação por testar
  ssh-keygen -t ed25519-sk -O resident -f ~/.ssh/id_yubikey2_ssh_sk -C "$(whoami)-$(date +'%d-%m-%Y')-yubikey2"

  # and another.. git ssh com verificação
  ssh-keygen -t ed25519-sk -O resident -O verify-required -O application=ssh:5nano_gituser -f ~/.ssh/id_5nano_gituser_sk -C "yourgitemail@example.com"

   # and another.. git ssh sem verificação
  ssh-keygen -t ed25519-sk -O resident -O application=ssh:yubikey2_noverify -f ~/.ssh/yubikey2_git_noverify_sk -C "yourgitemail@example.com"
  ```

   - é importante criar as chaves com a opção `application=ssh:....` para dar um nome à chave ou serviço e para não se confundirem com outras chaves e/ou yubikeys.
   - caso a localização para guardar as chaves não seja indicada vai pedir uma localização para as guardar. Por padrão, deve ser em `~/.ssh`.
   - Esta chave privada gravada localmente é, mais uma vez, uma "referência" à chave privada gravada na yubikey
   


### Listar as chaves da Yubikey


- Para consultarmos as credenciais existentes na nossa chave de segurança:

  ```shell
  # Para listar as chaves existentes na yubikey 
  ykman fido credentials list

  # Output exemplo:
  Credential ID     RP ID              Username               Display name         
  5c22bb11aa00bc    google.com         myemail@gmail.com                        
  4d4563aa32er35    ssh:5nano          openssh                   openssh

  # Com mais  detalhes
  ykman fido credentials list --csv
  ```


### Extrair uma resident key da Yubikey para um novo sistema:

- Muitas vezes confundida como uma cópia de chaves, na verdade não é uma cópia que fazemos mas sim uma extração. O que é copiado não é exatamente igual à chave privada residente. Extraímos sim, uma representação da chave privada residente que vai ser usada pela yubikey para a autenticação.  
Estas chaves extraídas são também chamadas de "chave privada derivada" ou "chave privada exportada".
  - Desta forma, a chave privada residente nunca é exposta pois apenas existe na yubikey. Mesmo que o sistema operativo seja "hackeado" as private keys extraídas não são úteis sem a chave física de hardware com a verdadeira resident key.
  - Apesar da chave privada exportada existir no sistema operativo, ela ainda necessita da yubikey para operações críticas.


- Para extrairmos uma resident key:
  ```shell
  # mudar para o directório ssh e extrair:
  cd ~/.ssh && ssh-keygen -K
  ```
  
  - Parece que há inconsistência em diferentes sistemas operativos nos nome dos ficheiros gerados ao serem extraídos. Uma opção será renomear o nome das chaves privadas e publicas extraídas para o `.ssh`.
  - Ao extrair as chaves, mais do que uma vez no mesmo sistema, não grava "por cima" *overwrite* das existentes. Assim é possível ficarmos com múltiplos ficheiros shh. Cada chave extraída vai ser guardada com um nome único.
  - Verificar as permissões das chaves extraídas, a chave privada deve ter as permissões 600 e a chave pública 644.
  - De momento, ao extrair uma chave ssh resident, se tivermos múltiplas na Yubikey, vai extrair-las todas. Ou seja, de momento não me parece existir opção de extrair uma chave específica como existe para o caso de apagar uma chave determinada.


### Apagar uma Resident key de uma Yubikey

- Para apagar uma chave:

  ```shell
  # listar as chaves na yubikey
  ykman fido credentials list

  Credential ID     RP ID              Username               Display name         
  5c22bb11aa00bc    google.com         myemail@gmail.com                        
  4d4563aa32er35    ssh:5nano          openssh                   openssh

  # dando nome atribuido à key
  ykman fido credentials delete  NAMEOFKEY # example ssh:5nano

  # ou dando uma parte inicial da chave
  ykman fido credentials delete 4d4563aa
  ```

<br>

## Outros
### Aviso
A gestão de credenciais, Yubikeys e semelhantes, requer tempo, conhecimento e testes. Este documento foi feito com base em notas pessoais, não me responsabilizo por alguma incorreção ou desatualização.

###

## Bibliografia:
### Docs
  - https://support.yubico.com/hc/en-us/sections/360003997900-Guides
  - https://docs.yubico.com/software/yubikey/tools/ykman/webdocs.pdf
  - https://developers.yubico.com/SSH
  - https://feldspaten.org/2024/02/03/ssh-authentication-via-Yubikeys/
  - https://swjm.blog/the-complete-guide-to-ssh-with-fido2-security-keys-841063a04252
 


### Vids
  - [Yubikey playlist - CodeWreck ](https://www.youtube.com/playlist?list=PLn9t_BnhwY0KXIqloOys7cCDFSHJycrDl)
  - [YubiKey Complete Getting Started Guide! - Learn Linux Tv](https://www.youtube.com/watch?v=INi-xKpYjbE)




<br>

<div dir=RtL> 
:Autor da tradução</br>
<i>Zito Cavaleiro  </i>-   email: zcavaleiro AT protonmail DOT com</br>
Professor de Matemática e Ciências da Natureza</br>
Engenheiro de Automação de Tecnologias de Informação</div>


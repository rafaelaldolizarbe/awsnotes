Após a instalação do AWS CLI pelo .msi é necessário configurar a pasta do usuário com informações IAM contidas na conta estudantil que podem ser adquiridas a partir do passo a passo a seguir:

Em um primeiro momento é necessário iniciar o laboratório de aprendizado da AWS pois sem ele estar inicializado não é possível realizar as configurações das credenciais que por padrão se encontram no diretório do usuário:
 `C:\Users\[nome do usuario]\.aws` 
 ou no linux  `~/.aws` que pode ser lido como:
 `/home/[nome do usuario]/.aws`

![[aws1.png]]

Após a inicialização do laboratório será possível encontrar as informações necessárias para autenticar o aws cli a clicando na aba AWS Details
![[image2-5.png]]

Siga as orientações e copie o conteúdo de texto que se encontra dentro da pasta .aws  no caso do Windows o caminho é `C:\Users\[nome do usuario]\.aws`

Usando uma cli é ideal que você tenha um editor de texto no meu caso neovim
`nvim credentials`

logo você copia o conteúdo da página que se encontra dentro do quadro e salva
`esc`
`:wq`

verifique se o arquivo foi preenchido corretamente através do comando:
`cat credentials`

Após isso qualquer comando que você utilize usando a cli da aws será executado com sucesso. não se esqueça de atualizar esse documento sempre que reiniciar a sessão. tome cuidado para não armazenar esse arquivo em diretórios não ocultos, ou compartilhar o mesmo se não estiver mais usando uma conta estudantil e passar para free tier.

Assim o ambiente está pronto e configurado para ser acessivel tanto pelo AWS CLI quanto pelo terraform.
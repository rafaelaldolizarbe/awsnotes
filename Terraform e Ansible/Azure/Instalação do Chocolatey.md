
É importante considerar que existem dependências que por padrão podem não estar instaladas no PowerShell do computador

Apesar de seguir as orientações presentes no [tutorial](https://docs.chocolatey.org/en-us/choco/setup)

É importante que erros como os que vem a seguir possam ocorrer devido à alguma tentativa anterior :

![[Pasted image 20240330162637.png]]

Por padrão remover a instalação manualmente pode não funcionar!

![[Pasted image 20240330162607.png]]

Nesse caso a instalação do módulo "Microsoft.PowerShell.Archive" não está disponível más pode ser instalado a partir deste [link](https://www.powershellgallery.com/packages/Microsoft.PowerShell.Archive/1.2.5)

E após repetir o procedimento :

![[Pasted image 20240330162837.png]]

O Chocolatey encontra-se instalado e pronto para prosseguir com a instalação do Terraform.
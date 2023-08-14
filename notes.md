#### 14/08/2023

Curso de Redes parte 3: defina as listas de controle e políticas de acesso de usuários

@01-Listas de acesso (ACL)

@@01
Introdução

Olá, pessoal! Vamos dar uma introdução sobre o que veremos na terceira parte do curso de Redes.
Antes de tudo, se você ainda não fez a Parte 1 e nem a Parte 2, recomendamos que você as faça primeiro, pois iremos continuar o mesmo projeto das partes anteriores.

Teremos agora novas requisições dos diretores da empresa Multillidae, onde precisaremos configurar políticas de acesso para que somente os gerentes de finanças e de vendas tenham permissão de acessar o servidor Server-PT Server0.

Por sua vez, os funcionários de venda e de finanças não terão permissão de acesso.

Depois, iremos realizar a tradução desses endereços IPs privados na rede interna para o endereço público, sendo ele o endereço IP que conseguimos nos comunicar com a internet.

Realizaremos a tradução desses endereços privados para os públicos, e estabelecer a comunicação com o roteador do provedor da rede de serviços.

Até as próximas aulas!!

@@02
Implementando servidor

Usando as subredes, conseguimos trabalhar de uma forma mais eficiente com os endereços IPs.
Imagine que temos um novo dia de reunião na empresa, e os diretores disseram que será implementado um servidor interno na empresa, com uma condição...

Esse servidor só poderá ser acessado pelo Gerente de Finanças e pelo Gerente de Vendas. Os demais funcionários desses setores em questão, não terão acesso ao mesmo.

Arquitetura do servidor com acesso para gerentes financas e vendas

A primeira etapa é colocar o servidor no projeto. Para isso, vamos acessar a opção End Devices, e clicaremos na terceira opção e arrastamos:

Servidor

Depois, iremos conectar esse servidor em uma das portas do Switch, utilizando o cabo direto (4 opção).

Clicando em "FastEthernet0", escolhemos a porta FastEthernet0/7. Assim como fizemos com o setor de vendas e de finanças, criaremos uma VLAN para os servidores.

Clicando no Switch 2950-24, na aba "CLI", vamos configurar uma VLAN respectiva para os servidores, pois no futuro, pode ser que os diretores queiram adquirir outros servidores, e assim, eles já vão ficar isolados em uma VLAN respectiva deles.

Para fazer essa configuração, entraremos no modo privilegiado, e depois no modo de configuração:

>
>enable
#configure terminalCOPIAR CÓDIGO
Agora, precisaremos criar uma VLAN respectiva para os servidores. Como já utilizamos a VLAN 10 para o setor de vendas, e a VLAN 20 para o setor de finanças, utilizaremos a VLAN 30 para seguir a ordem de 10 em 10.

#vlan 30
#name SERVIDORES
#^ZCOPIAR CÓDIGO
Utilizamos o comando "Ctrl + Z", para utilizar o comando para ver como estão atribuídas as VLANs nesse switch.

#show vlan briefCOPIAR CÓDIGO
É mostrado que nesse VLAN, temos a VLAN 10, 20 e 30. Só que a VLAN 30 que criamos agora não tem nenhuma interface configurada nele. Precisamos configurar a FastEthernet0/7 para que essa interface trabalhe com a VLAN 30, a VLAN respectiva aos servidores.

VLAN  Name                Status    Ports
----- ------------------  --------  -------------------------------
1     default             active    Fa0/2, Fa0/3, Fa0/4, Fa0/7
                                    Fa0/9...
                                    Fa0/13...
                                    Fa0/17...  
                                    Fa0/21...
10    VENDAS              active    
20    FINANCAS            active    
30    SERVIDORES          active    
1002  fddi-default        active    
1003  token-ring-default  active    
1004  fddinet-default     active    
1005  trnet-default       active    COPIAR CÓDIGO
#configure terminal
#interface fastEthernet 0/7COPIAR CÓDIGO
A primeira etapa é mudar a forma que essa porta irá trabalhar, dizer que ela está conectada no dispositivo final.

#switchport mode accessCOPIAR CÓDIGO
Agora precisamos falar qual a VLAN que está conectada a essa interface:

#switchport access vlan 30COPIAR CÓDIGO
Quando fazemos isso, a porta do switch cai momentaneamente para que ela possa se reconfigurar com as informações da VLAN 30. Usaremos o comando #show vlan brief:

VLAN  Name                Status    Ports
----- ------------------  --------  -------------------------------
1     default             active    Fa0/2, Fa0/3, Fa0/4, Fa0/8
                                    Fa0/9...
                                    Fa0/13...
                                    Fa0/17...  
                                    Fa0/21...
10    VENDAS              active    
20    FINANCAS            active    
30    SERVIDORES          active    Fa0/7
1002  fddi-default        active    
1003  token-ring-default  active    
1004  fddinet-default     active    
1005  trnet-default       active    COPIAR CÓDIGO
Então, aqui podemos ver que a interface Fa0/7 está vinculado a VLAN 30 em SERVIDORES.

Vamos configurar o 1841Router1 para que ele tenha também uma subinterface que seja vinculada à VLAN 30.

Clicando no roteador, na aba "CLI", colocaremos alguns comandos:

>enable
#configure terminalCOPIAR CÓDIGO
Depois, criaremos uma subinterface para a VLAN 30:

#interface fa0/0.3COPIAR CÓDIGO
Agora diremos que a VLAN 30 estará vinculada a ela:

Router(config-subif)#encapsulation dot1Q 30COPIAR CÓDIGO
Legal! Configuraremos agora os endereços IPs. O primeiro será um endereço estático.

Anteriormente, estavamos trabalhando com a subrede 172.16.2.128 que foi alocado para o setor de finanças. Agora, iremos utilizar a terceira sub-rede dessa imagem, para alocar o setor de SERVDORES. (Lembrando que não podemos atribuir a nenhuma máquina ou interface, o endereço 172.16.3.0, e nem o endereço IP referente ao broadcast):

Lista de enderecos

Router(config-subif)#ip address 172.16.3.1 255.255.255.128COPIAR CÓDIGO
Esse é o primeiro endereço IP disponível no intervalo, e a máscara de subrede 255.255.255.128 que foi atribuída para o setor de finanças.

Depois de atribuir esse endereço a essa máscara de subrede, colocaremos no servidor, o endereço IP que esteja dentro da subrede.

Clicando no Server-PT Server0, colocaremos o IP estático 172.16.3.2, a máscara de subrede 255.255.255.128, o gateway 172.16.3.1 que é o valor que foi configurado na subinterface.

Atribuicao de endereco IP

Veremos agora se conseguimos pingar o servidor para o roteador em Command Prompt:

SERVER>ping 172.16.3.1COPIAR CÓDIGO
Muito bem! A sub interface criada estaticamente para a VLAN 30 e o endereço IP configurado estaticamente pro servidor!

Não podemos nos esquecer que os outros switches, também precisam passar informações da VLAN 30, caso algum usuário queira acessar o servidor Server-PT Server0. Entretanto a VLAN 30 ainda não foi configurada nesses switches.

Switches a serem configurados para conversar com a VLAN 30

Clicando no switch da esquerda, na aba "CLI", criaremos a VLAN 30 nesse switch:

>enable
>configure terminal
#vlan 30
#name SERVIDORESCOPIAR CÓDIGO
Faremos o mesmo para o Switch da direita:

>enable
#configure terminal
#vlan 30
#name SERVIDORESCOPIAR CÓDIGO
Vamos estilizar o nosso servidor para mostrar uma página de domínio próprio da Multillidae, para que o gerente de finanças e o gerente de vendas poderem acessar o servidor com uma página mais estilizada.

Na aba "Services", em "HTTP", clicamos na opção (edit) em index.html, e vamos colocar um código HTML:

<html>
    <h1>Servidor Multillidae</h1>
    <br>
    <input type="text" placeholder="Nome">
    <br>
    <input type="password" placeholder="Senha">
    <br>
    <button type="submit">Logar</button>
</html>COPIAR CÓDIGO
Em seguida, clicaremos em "Save". Isso irá deixar um pouco mais claro para o gerente de finanças ou o de vendas, para que eles façam o login sem problemas.

Veremos se conseguimos acessar a página desse servidor interno da empresa.

Em "Ger. Financas", clicaremos na aba "Desktop", e depois no ícone "Web Browser", e colocamos o endereço IP do servidor configurado.

URL: 172.16.3.2COPIAR CÓDIGO
Tela de login do gerente de financas

Veremos se somente o gerente de Finanças está acessando ou se outros usuários estão também acessando. No "Ger. Vendas", faremos os mesmos passos do gerente de finanças.

Este gerente também consegue ter acesso. Pela requisição dos diretores, os funcionários de vendas e de finanças, não devem acessar esse servidor.

Clicando no "Web Browser" de "Func. Finanças" e de "Func. Vendas", e colocando o mesmo endereço IP, também aparecerão a tela de login, isso quer dizer que eles também estão tendo acesso ao servidor.

Isso quer dizer que ainda não temos o sucesso total. Já implementamos o servidor, o isolamos na sua VLAN respectiva, entretanto TODOS os funcionários estão conseguindo acessar à página do login interno da Multillidae.

Na próxima etapa, veremos como podemos evitar que outros funcionários acessem essa página.

@@03
O que seria esse comando?
PRÓXIMA ATIVIDADE

Estamos configurando um Switch da Cisco e precisamos configurar o Switch para que ele saiba que sua porta está conectada a um equipamento final. Qual comando da Cisco nós podemos digitar para informar que determinada porta está conectada a um dispositivo final?


O comando seria switchport mode all
 
O comando switchport mode all não existe
Alternativa correta
O comando seria switchport mode dynamic desirable
 
Alternativa correta
O comando seria switchport mode access
 
O comando switchport mode access indicaria que essa porta está conectada a um dispositivo final, sendo assim a alternativa correta
Alternativa correta
O comando seria switchport mode trunk
 
O comando switchport mode trunk indicaria que essa porta está conectada a outro Switch podendo encaminhar múltiplas Vlans

04
Mãos à obra: Servidor interno
PRÓXIMA ATIVIDADE

Nós tivemos uma reunião com os diretores da empresa e eles informaram que será implementado um servidor interno onde o mesmo só deverá ser acessado pelo gerente de vendas e pelo gerente de finanças. Nosso primeiro passo será instalar esse servidor e isolá-lo em sua própria Vlan, a Vlan 30.
Arrastre para a área de trabalho um servidor, clicando na opção End Devices e depois selecionando a opção Server (terceira opção da esquerda para direita). Posteriormente conecte o Servidor ao Switch conectado ao roteador, vamos usar a porta FastEthernet 0/7 para realizar a conexão. Devemos ter uma imagem parecida com a abaixo:
servidor_interno

Clique no Switch conectado ao servidor e vá até a aba CLI
Entre na parte privilegiada digitando enable e na sequência digite configure terminal para entrar na parte de configuração
Crie a Vlan 30, que será usada para alocação dos servidores. Digite vlan 30 e na sequência insira o nome para essa vlan, digitando name SERVIDORES
Posteriormente, saia da configuração da vlan digitando exit e entre na interface que foi conectada ao servidor, (por exemplo: interface FastEthernet 0/7)
Devemos alterar o modo de operação dessa interface indicando que ela está conectada a um dispositivo final, digitamos: switchport mode access
Na sequência, devemos associar essa interface com a Vlan 30, digitando switchport access vlan 30
Confirme se essa interface está vinculada a Vlan 30, precisamos primeiro ir para parte privilegiada digitando CTRL+Z e depois devemos digitar show vlan brief
Uma vez que isolamos esse servidor em sua respectiva Vlan, precisamos criar uma nova sub-interface no roteador para que essa Vlan possa se comunicar com a Vlan 10 de vendas e a Vlan 20 de finanças.

Clicamos no roteador e na sequência vamos até a aba CLI
Entramos na parte privilegiada digitando enable e na sequência entramos na parte de configuração digitando configure terminal
Para criar essa sub-interface, digitamos interface FastEthernet 0/0.3
Devemos dizer que essa sub-interface estará associada com a Vlan 30, digitamos: encapsulation dot1Q 30
Vamos escolher a próxima sub-rede disponível (172.16.3.0 - 172.16.3.127). Não podemos atribuir endereços IP de sub-rede nem de broadcast para nenhuma máquina, dessa forma, iremos escolher o primeiro endereço IP disponível para a sub-interface. Digitamos ip address 172.16.3.1 255.255.255.128
Clique no servidor e vá até a aba Desktop e em seguida clique em IP Configuration
Podemos atribuir qualquer endereço IP disponível dentro dessa sub-interface (menos o 172.16.3.1 que já usamos para a sub-interface do roteador), vamos pegar então o segundo endereço IP disponível, 172.16.3.2, a máscara deverá ser 255.255.255.128 e o default gateway é o endereço IP da sub-interface do roteador (172.16.3.1). Devemos ter a seguinte configuração.
ip_servidor

No servidor, vá a aba Services -> HTTP e no box index.html clique em edit. Apague todo o código e insira esse no lugar:
<html>
     <h1>Servidor Mutillidae</h1>
     <br>
     <input type="text" placeholder="Nome">
     <br>
     <input type="password" placeholder="Senha">
     <br>
     <button type="submit">Logar</button>
</html>COPIAR CÓDIGO
Clique agora no Switch de vendas (canto inferior esquerdo), vá até a aba CLI
Entre na parte privilegiada digitando enable e posteriormente na parte de configuração digitando configure terminal
Vamos criar a vlan 30 no Switch, informando assim que as portas Trunk desse Switch irão trafegar dados da Vlan 30. Digitamos vlan 30 e na sequência digitamos name SERVIDORES
Repita esses mesmos passos para criação da Vlan 30 no Switch do setor de finanças.
Feito isso, clique em um dos computadores, vá até a aba Desktop e na sequência clique em Web Browser
Por fim, coloque na URL o endereço IP do servidor: 172.16.3.2. Qual o resultado?

Opinião do instrutor

Ao isolarmos o servidor em sua respectiva Vlan e criarmos uma sub-interface no roteador, conseguimos estabelecer a comunicação entre as Vlans de vendas e a Vlan de finanças. E temos a página de login do servidor da Mutillidae:
mutillidae

https://s3.amazonaws.com/caelum-online-public/Redes_II/servidor_interno.JPG

@@05
Conhecendo as listas de acesso

@@06
Listas de acesso
PRÓXIMA ATIVIDADE

O que são listas de acesso?


As listas de acesso são uma forma restringirmos todos os usuários de acessar um recurso da rede somente.
 
Alternativa correta
As listas de acesso são uma forma de restringir o acesso de usuários a servidores somente.
 
Alternativa correta
As listas de acesso seriam uma forma a qual podemos especificar políticas de permissão ou negação de acesso de recursos de uma rede por parte dos usuários.
Alternativa correta
As listas de acesso são uma forma que podemos permitir acesso externo em nossa rede interna, somente.
 
Listas de acesso ou do inglês (ACL) são listas as quais contém políticas de permissão ou negação de acesso por parte de clientes. Dessa forma, conseguimos criar políticas por usuário de quais protocolos e serviços que podem ser utilizados

@@07
Políticas lista de acesso
PRÓXIMA ATIVIDADE

Se nós temos um roteador configurado com uma lista de acesso, porém essa lista de acesso não possui nenhuma configuração relacionada a um pacote que chegou, o que acontecerá com esse pacote?

O pacote seguirá até seu destino, a lista de acesso só avalia os pacotes cujas configurações estejam presentes na lista de acesso.
 
A lista de acesso irá analisar todo pacote que chegar, caso não haja uma configuração específica para esse pacote que chegou, o pacote será descartado por padrão.
Alternativa correta
O pacote seguirá até seu destino, porém a lista vai marcar esse pacote com uma tag e irá monitorar seu retorno para analisar o que pode conter de informação dentro desse pacote.
 
Como não há uma política de configuração na lista de acesso para esse pacote, ele será descartado por padrão. Não ocorre marcação de tags, etc.
Alternativa correta
O pacote não seguirá até seu destino, isso ocorre porque a lista de acesso irá bloquear por padrão qualquer pacote que não possua uma configuração na lista de acesso.
Uma vez que a lista de acesso não possui nenhuma configuração para tratamento de um pacote de informação, a lista irá descartar esse pacote.
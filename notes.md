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

Se nós temos um roteador configurado com uma lista de acesso, porém essa lista de acesso não possui nenhuma configuração relacionada a um pacote que chegou, o que acontecerá com esse pacote?

O pacote seguirá até seu destino, a lista de acesso só avalia os pacotes cujas configurações estejam presentes na lista de acesso.
 
A lista de acesso irá analisar todo pacote que chegar, caso não haja uma configuração específica para esse pacote que chegou, o pacote será descartado por padrão.
Alternativa correta
O pacote seguirá até seu destino, porém a lista vai marcar esse pacote com uma tag e irá monitorar seu retorno para analisar o que pode conter de informação dentro desse pacote.
 
Como não há uma política de configuração na lista de acesso para esse pacote, ele será descartado por padrão. Não ocorre marcação de tags, etc.
Alternativa correta
O pacote não seguirá até seu destino, isso ocorre porque a lista de acesso irá bloquear por padrão qualquer pacote que não possua uma configuração na lista de acesso.
Uma vez que a lista de acesso não possui nenhuma configuração para tratamento de um pacote de informação, a lista irá descartar esse pacote.

#### 15/08/2023

@02-Configurando ACL

@@01
Listas de acesso

Conseguimos implementar o nosso servidor interno, porém, ainda não conseguimos atender as requisições dos diretores. Eles pediram que somente os Gerentes de Finanças e Vendas tenham acesso à página de login do servidor.
A tarefa é configurar as listas de acesso no roteador, para garantir acesso somente aos gerentes de vendas e finanças.

Vamos clicar no roteador 1841 Router1, e na aba CLI:

Na primeira etapa, criaremos a lista de acessos com o comando ip access-list ?

>enable
#configure terminal
#ip access-list ?COPIAR CÓDIGO
Podemos escolher entre a lista estendida ou a standard. A estendida nos permite realizar essa verificação tanto na origem como no destino. Já a standard só faz verificação na origem. Nós queremos permitir que o computador do gerente de finanças e do gerente de vendas, tenham acesso ao servidor Server-PT Server0.

Então, vamos realizar o filtro tanto na origem como no destino, por isso, usaremos a lista estendida:

#ip access-list extended SERVIDOR-GERENTESCOPIAR CÓDIGO
Criamos a nossa lista de acesso, e precisamos informar quais são as políticas de tráfego que serão analisadas pela lista de acesso. Por isso, vamos permitir o acesso dos computadores dos gerentes de vendas e de finanças.

Como vimos anteriormente, quem realiza o transporte da informação é o protocolo TCP, que depois dele, terá a transmissão do protocolo HTTP. No comando #permit tcp, não podemos nos esquecer de passar o endereço IP da nossa origem. No caso, inicializaremos primeiro com o computador do gerente de finanças.

O endereço IP que está relacionado a ele agora é o 172.16.2.131. Entretanto, esse endereço está sendo fornecido de forma dinâmica. Quer dizer que, eventualmente, ele pode ser alterado.

Se colocarmos esse endereço dinâmico na lista de acesso, e esse valor for alterado, a nossa lista acaba não tendo mais utilidade... Para evitarmos esse problema, vamos alterar a forma de trabalhar com endereço IP no computador dos gerentes de finanças e de vendas. Usaremos esses endereços de forma estática, garantindo que esse endereço não seja alterado.

Endereco estatico do gerente de financas

Agora sim podemos colocar o endereço no roteador:

#permit tcp 172.16.2.131COPIAR CÓDIGO
Adicionando o ? no final do comando, aparece para nós um conceito novo a ser tratado: o Source wildcard bits!

Usando o Source wildcard bits, conseguimos indicar para a lista de acesso qual a parte do endereço IP - que tem que ser exatamente igual ao digitado-, e qual parte não importa o valor. Temos que indicar por meio do Source wildcard bits, que a lista de acessos deve considerar como sendo origem, o primeiro intervalo do endereço IP 172, o segundo intervalo sendo 16, o terceiro intervalo sendo 2, e o quarto intervalo não importa. Por isso, utilizaremos 255 para representar o intervalo que não importa, e 0 para os intervalos que importam, ou seja, eles devem ser exatamente iguais ao endereço IP.

#permit tcp 172.16.2.131 0.0.0.255COPIAR CÓDIGO
Então, quando temos uma rede de 100 computadores, por exemplo, não precisamos estar configurando em cada linha, o endereço IP exato de cada um desses computadores para que cada um tenha a permissão. Assim, dizendo que o endereço IP é global, e somente o último intervalo não importa, economizamos muito tempo, pois ele considera TODOS os endereços que começam com 172.16.2..

No entanto, no nosso caso, não queremos que todos tenham permissão. Queremos que esse endereço IP específico 172.16.2.131 tenha permissão de acessar o servidor. Para isso, em vez de colocar 0.0.0.255, colocaremos 0.0.0.0, assim a permissão só será concedida a esse endereço.

Agora, vamos indicar o endereço IP de destino, o endereço IP do servidor.

#permit tcp 172.16.2.131 0.0.0.0 172.16.3.2COPIAR CÓDIGO
Acrescentando um ? no final do comando, especificaremos qual parte desse endereço IP de destino que tem que ser exatamente igual, e qual parte do endereço não importa. Queremos trabalhar exatamente o endereço 172.16.3.2 para esse servidor específico.

#permit tcp 172.16.2.131 0.0.0.0 172.16.3.2 0.0.0.0COPIAR CÓDIGO
Vamos fazer o mesmo processo para o computador do gerente de vendas.

Como podemos ver, o endereço IP do gerente de vendas também está dinâmico, e para evitar que eventualmente ser alocado algum endereço IP para esse computador, mudaremos a configuração para static, pois assim garantiremos que esse endereço IP desse computador não será alterado.

Endereco estatico do gerente de vendas

Realizaremos uma nova permissão para esse outro endereço IP:

#permit tcp 172.16.0.2 0.0.0.0 172.16.3.2 0.0.0.0COPIAR CÓDIGO
Criamos a nossa lista de acesso, e colocamos quais são os endereços que terão permissão de acessar o servidor.

Não podemos nos esquecer do seguinte. Os dois endereços "172.16.2.131" e "172.16.0.2", estão configurados estaticamente. Será preciso informar ao Pool DHCP para que ele exclua esses dois endereços, para que ele não entregue esses mesmos endereços para outros clientes na mesma rede.

#exit
#ip dhcp excluded-address 172.16.2.131
#ip dhcp excluded-address 172.16.0.2COPIAR CÓDIGO
Desta forma, o Pool DHCP não vai considerar mais esses endereços na lista de endereços disponíveis. Vamos ver o resultado da configuração da lista de acessos.

O primeiro a ser testado é o gerente de finanças, e ele ainda possui acesso ao servidor. Entretanto, os funcionários de vendas ainda possuem acesso ao servidor! E por que eles continuam tendo acesso?

Precisamos configurar essa lista de acessos para que ela esteja vinculada com as subinterfaces da VLAN 10 de vendas, e da VLAN 20 de finanças.

Na aba "CLI" de Router1, entraremos na subinterface do setor de vendas:

#interface fastEthernet 0/0.1COPIAR CÓDIGO
Para fazer essa associação da lista com essa interface, utilizaremos o comando ip access-group e também o nome da lista de acesso que queremos vincular com essa subinterface. Depois disso, diremos qual será o sentido que a análise deve ser feita. Utilizamos (in)quando o pacote estiver entrando no roteador por essa subinterface, ou (out) quando ele estiver saindo do roteador por essa subinterface.

Em nosso diagrama, o pacote TCP chegará no primeiro switch, depois ele vai para o switch principal, e daí ele vai entrar no roteador. Sabendo disso, podemos criar uma política de acesso para que o roteador verifique tudo o que estiver entrando nessa subinterface:

#ip access-group SERVIDOR-GERENTES inCOPIAR CÓDIGO
Será feita a mesma configuração para o setor de finanças na VLAN 20 através da subinterface 0/0.2.

#exit
#interface fastEthernet 0/0.2
#ip access-group SERVIDOR-GERENTES inCOPIAR CÓDIGO
Vamos ver se agora temos sucesso em nossa análise com essa lista de acesso.

Depois que o pacotinho TCP saiu do computador do gerente de finanças e chegou até o roteador, passando pela lista de acessos configurada para a VLAN 20, e tendo permissão, o roteador vai mandar o pacotinho adiante para chegar até o servidor, e com isso, o gerente de finanças terá acesso ao servidor.

Vamos analisar o pacotinho TCP que saiu do computador dos funcionários de vendas.

Informacao do pacote TCP dos funcionarios de vendas

A imagem acima nos diz que o pacote não bateu com nenhum critério existente na lista de acessos. Por isso, ele será negado e descartado. Isso quer dizer que os funcionários de vendas não conseguem mais acessar o servidor, e se fizermos o mesmo teste com os funcionários de finanças, veremos que eles também não terão mais acesso ao servidor.

Agora, precisamos testar se a comunicação feita antes não está quebrada. Vamos "pingar" o computador do gerente de finanças (VLAN 20), para os funcionários de vendas (VLAN 10).

Clicando no computador do gerente de finaças, em "Desktop > Command Prompt", colocaremos o comando:

>ping 172.16.0.3COPIAR CÓDIGO
Em seguida, recebemos a mensagem:

Pinging 172.16.0.3 with 32 bytes of data:

Reply from 172.16.2.129: Destination host unreachable.
Reply from 172.16.2.129: Destination host unreachable.
Reply from 172.16.2.129: Destination host unreachable.
Reply from 172.16.2.129: Destination host unreachable.

Ping statistics for 172.16.0.3:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
COPIAR CÓDIGO
Por que perdemos a comunicação entre os dispositivos que estão em uma VLAN diferente?

@@02
Wildcard

Ao escrever em uma lista de acesso:
permit tcp 172.16.0.2  0.0.0.255  200.1.2.3 0.0.0.0COPIAR CÓDIGO
Quais são os endereços IP de origem permitidos e qual o seria o endereço de destino em questão?

Alternativa correta
O endereço IP de origem serão todos os endereços IP que terminarem por 2 e o endereços IP de destino será 200.1.2.3
 
Alternativa correta
O endereço IP de origem será 172.16.0.2 e o endereço IP de destino será 200.1.2.3
 
Alternativa correta
O endereço IP de origem serão todos que iniciarem por 172.16.0, já o endereço IP de destino será 200.1.2.3
 
O número 255 no wildcard representaria qualquer valor naquele intervalo do endereço IP, já o 0 representaria exatamente o valor que digitamos no endereço IP. Com isso o endereço IP de origem será 172.16.0.x e o endereço IP de destino será 200.1.2.3
Alternativa correta
O endereço IP de origem será 172.16.0.2 e o endereço IP de destino serão todos que começarem por 200.1.2.
 
Nós devemos analisar o wildcard bits para poder verificar quais são os endereços IP válidos. A primeira wildcard bits é 0.0.0.255, onde nós tivermos 0 indica que aquele intervalo tem que ser igual ao intervalo do endereço IP que digitamos no campo anterior, onde tivermos 255 indica que aquele intervalo do endereço IP pode ter qualquer valor. Dessa forma, se tivermos:
172.16.0.2  0.0.0.255COPIAR CÓDIGO
Indicamos que os três primeiros intervalos do endereço IP devem ser iguais aos valores inseridos (172.16.0), já o último intervalo do wildcard possui o valor 255, isso indica que nesse intervalo do endereço IP poderá conter qualquer valor. Então no final teremos:

172.16.0.[qualquer valor]

Então os endereços IP de origem que serão aceitos são todos os que iniciarem por 172.16.0

Fazendo a mesma análise para o endereço IP de destino

200.1.2.3 0.0.0.0COPIAR CÓDIGO
Uma vez que temos todos os intervalos em 0 no wildcard, isso indica que o valor do endereço IP deve ser igual ao valor digitado. Dessa forma, o endereço IP de destino será 200.1.2.3

@@03
Permitindo outros tráfegos

Nesta última etapa, fizemos que somente o servidor fosse acessado pelos gerentes de finanças e pelo gerente de vendas, entretanto a nossa comunicação entre os computadores que estão em VLANs diferentes, acabou sendo comprometida. Vamos entender o porquê.
No modo Simulation, realizaremos um ping no computador do gerente de finanças para o computador do funcionário de vendas:

>ping 172.16.0.3COPIAR CÓDIGO
O que chegou no roteador foi o pacote ICMP. Se clicarmos nesse protocolo, temos essa imagem:

Informacoes do pacote ICMP

Na entrada do roteador, temos a lista de acessos, mas essa lista comparou o tráfego com o protocolo ICMP com os dados inseridos. Sabemos que não temos nenhum tratamento para esse protocolo. Por isso, ele não conseguirá passar por não atinge nenhuma das restrições da lista.

Perceba que em nossa lista de acesso, estamos permitindo somente dois tipos de acesso que utilizam o protocolo TCP, cuja a origem está no gerente de vendas e no gerente de finanças para esse servidor. Mas, e todo o resto dos outros protocolos de comunicação? Da maneira como foi configurado, toda a comunicação está perdida!

Então temos que alterar a lista de acessos para permitir todos os outros protocolos de comunicação. O foco da nossa lista de acesso é só o servidor. O restante do tráfego que acontece na rede pode seguir normalmente. Então vamos remover a lista de acesso, e criaremos uma nova lista!

#configure terminal
#no ip access-list extended SERVIDOR-GERENTESCOPIAR CÓDIGO
Usando essa linha de comando, conseguimos remover a lista de acessos que criamos. Criaremos uma nova lista de acessos com algumas informações adicionais.

#ip access-list extended SERVIDOR-GERENTESCOPIAR CÓDIGO
Essa nova lista ainda deve manter a permissão do gerente de vendas e de finanças, para que eles acessem o servidor.

#permit tcp 172.16.2.131 0.0.0.0 172.16.3.2 0.0.0.0
#permit tcp 172.16.0.2 0.0.0.0 172.16.3.2 0.0.0.0COPIAR CÓDIGO
Depois disso, devemos permitir que os outros protocolos estejam habilitados. O comando #permit permite TUDO. Então, antes de permitir tudo, precisamos negar os outros computadores que estão lá no setor de finanças e do setor de vendas, para que eles não tenham acesso ao servidor.

Primeiro, vamos negar todos os outros computadores de finanças, cujo os endereços se iniciam com 172.16.2..

#deny tcp 172.16.2.128 0.0.0.255COPIAR CÓDIGO
E esses computadores serão negados para acessar o servidor 172.16.3.2:

#deny tcp 172.16.2.128 0.0.0.255 172.16.3.2 0.0.0.0COPIAR CÓDIGO
Agora, negaremos os computadores do setor de vendas, seguindo a mesma linha de raciocínio:

#deny tcp 172.16.2.128 0.0.0.255 172.16.3.2 0.0.0.0
#deny tcp 172.16.0.128 0.0.0.255 172.16.3.2 0.0.0.0COPIAR CÓDIGO
Isto quer dizer que se não for nenhum desses endereços (172.16.2.131 e 172.16.0.2), o pacotinho será negado! Feito isso, vamos permitir que os outros protocolos de comunicação trabalhem normalmente:

#deny tcp 172.16.2.128 0.0.0.255 172.16.3.2 0.0.0.0
#deny tcp 172.16.0.128 0.0.0.255 172.16.3.2 0.0.0.0
#permit ip any anyCOPIAR CÓDIGO
Com o #permit ip any any, estamos permitindo qualquer tráfego que não seja destinado ao servidor.

Veremos se o roteador permitirá a comunicação entre VLANs distintas, e permitir o protocolo ICMP de seguir adiante.

No modo Simulation, vamos "pingar" novamente o computador do gerente de finanças (VLAN 20), para os funcionários de vendas (VLAN 10):

>ping 172.16.0.3COPIAR CÓDIGO
Se clicarmos no pacotinho ICMP, teremos a seguinte imagem:

Informacoes do trafego ICMP

Analisando a imagem, vemos que há um critério que bate com o tráfego do pacote ICMP, que é o permit ip any any, pois esse pacote não está sendo destinado ao servidor. Uma vez que colocamos em nossa lista de acessos para ela permitir qualquer coisa, o pacote seguirá adiante.

No modo Realtime, conseguimos ver que a comunicação voltou a ser estabelecida!

Mas, será que somente os gerentes de finanças e vendas ainda conseguem acessar o servidor?

Depois de testarmos os quatro computadores da nossa rede, percebemos que somente os gerente de finanças e vendas tem acesso ao servidor!

@@04
IP any any

Quando temos em uma lista de acesso a seguinte configuração:
permit ip any anyCOPIAR CÓDIGO
Quais seriam os endereços IP de origem e destino permitidos?

Só seriam aceitos endereços IP de duas sub-redes no máximo
 
Alternativa correta
Seriam aceitos qualquer endereço IP na origem e somente o endereços IP da mesma sub-rede no destino.
 
Alternativa correta
Seriam aceitos qualquer endereço IP de origem e de destino
Alternativa correta
Só seriam aceitos endereços IP dentro de uma sub-rede específica
 
Uma vez que estamos colocando any, isso indica que qualquer endereço IP de origem assim como qualquer endereço IP de destino será aceito. Essa configuração é comumente utilizada nas listas de acesso para permitir que todo tráfego que esteja fora do nosso foco da lista de acesso continue funcionando normalmente.

@@05
Comportamento lista de acesso

Em nosso projeto temos uma lista de acesso configurada da seguinte forma:
permit tcp 192.168.0.0  0.0.0.255  200.1.1.2  0.0.0.0
deny tcp 192.168.0.2  0.0.0.0  200.1.1.2  0.0.0.0
permit ip any anyCOPIAR CÓDIGO
Nossa meta é que somente o usuário com endereço IP 192.168.0.2 tenha o acesso negado ao servidor e que todos os demais usuários tenham acesso permitido. Porém o administrador de rede está dizendo que o usuário com endereço IP 192.168.0.2 não está tendo o acesso negado ao servidor 200.1.1.2 e está sendo permitido como todos os demais. Porque isso está acontecendo e como podemos resolver?


Isso ocorre porque colocamos errado os nomes permit e deny na primeira e segunda linha, o correto seria escrever:
permit tcp 192.168.0.2  0.0.0.0  200.1.1.2  0.0.0.0
deny tcp 192.168.0.0  0.0.0.255  200.1.1.2  0.0.0.0
permit ip any any
 
Alternativa correta
Isso occore por conta da linha:
permit ip any any
Essa linha está permitindo todos os demais protocolos de se comunicarem, removendo essa linha teremos o comportamento que buscamos
 
Alternativa correta
Isso ocorre por conta da forma que colocamos os parâmetros na nossa lista de acesso. A lista de acesso é interpretada sequencialmente e com isso, basta invertermos a primeira e a segunda linha que teremos o comportamento deseajado:
deny tcp 192.168.0.2  0.0.0.0  200.1.1.2  0.0.0.0
permit tcp 192.168.0.0  0.0.0.255  200.1.1.2  0.0.0.0
permit ip any any
 
A lista de acesso é parecida com um segurança e faz a análise sequencialmente, como especificado na resposta
Isso ocorre porque a lista de acesso é analisada sequencialmente, ou seja, quando o pacote chega a lista analisa a primeira linha. A primeira linha diz para permitir todo pacote que utiliza o protocolo TCP e cuja origem seja o endereço IP 192.168.0.[qualquer valor]. Dessa forma, quando o pacote do usuário 192.168.0.2 tentar acessar o servidor, a primeira comparação será com essa linha de permit e como o endereço IP 192.168.0.2 estaria contido no permit seu acesso seria liberado, uma vez que o critério foi compatível com a primeira linha, as demais linhas não seriam analisadas. (Lembre-se do segurança da festa, se ele achou nosso nome não tem porque ele ficar olhando os outros nomes na lista).
Para resolver o problema, bastaria inverter a ordem:

deny tcp 192.168.0.2  0.0.0.0  200.1.1.2  0.0.0.0
permit tcp 192.168.0.0  0.0.0.255  200.1.1.2  0.0.0.0
permit ip any anyCOPIAR CÓDIGO
Dessa forma, caso o usuário 192.168.0.2 tentar acessar o servidor ele será comparado a primeira linha da lista de acesso e essa linha diz que esse acesso deve ser bloqueado. Os demais usuários serão comparados a esse primeira linha também, porém eles não são o usuário com endereço IP 192.168.0.2 e portanto a análise passa para a segunda linha, a segunda linha diz que todos os outros endereços IP que comecem por 192.168.0.X teriam acesso permitido ao servidor e dessa forma, estabelecemos nossa meta de somente bloquear o usuário 192.168.0.2 e permitir acesso aos demais.

06
Mãos à obra: Criando listas de acesso

Os diretores da Mutillidae informaram que somente o gerente de finanças e o gerente de vendas devem ter acesso ao servidor, os demais funcionários do setor de vendas e finanças não devem ter acesso. Devemos para isso criar a lista de acesso.
Os endereços IP alocados pelo DHCP podem ser renovados em horas ou alguns dias. Dessa forma, como queremos dar acesso somente ao gerente de vendas e ao gerente de finanças, é importante que os endereços IP desses usuários não sejam alterados o que comprometeria nossa lista de acesso. Vamos então, configurar tais endereços IP estaticamente nos dois computadores e assim garantimos que eles não serão renovados.
Clique no computador do gerente de finanças e insira o endereço IP 172.16.2.131, máscara 255.255.255.128 e default-gateway 172.16.2.129. Devemos ter o seguinte resultado:
ip_estatico_financas

Devemos realizar essa mesma configuração para o computador do gerente de vendas colocamos estaticamente o endereço IP 172.16.0.2, máscara 255.255.254.0 e gateway 172.16.0.1. Teremos o seguinte resultado:
ip_estatico_vendas

Clique no roteador e vá até a aba CLI
Entre na parte privilegiada digitando enable e posteriormente entre na parte de configuração digitando configure terminal. Como configuramos esses endereços estaticamente devemos removê-lo dos pools DHCP para não serem alocado assim para nenhum outro usuário.
Digite ip dhcp excluded-address 172.16.0.2
Na sequência, digite ip dhcp excluded-address 172.16.2.131
Devemos criar agora a lista de acesso permitindo somente que o computador do gerente de vendas e do gerente de finanças acessem o servidor.

O primeiro passo é criar essa lista de acesso, digitamos: ip access-list extended SERVIDOR-GERENTES
Devemos criar essas políticas de acesso permitindo primeiramente o acesso do computador do gerente de finanças, digitamos permit tcp 172.16.2.131 0.0.0.0 172.16.3.2 0.0.0.0
Na sequência permitimos o acesso do gerente de vendas, digitamos: permit tcp 172.16.0.2 0.0.0.0 172.16.3.2 0.0.0.0
Em seguida, devemos negar o acesso dos demais funcionários do setor de vendas e finanças, para negar acesso aos demais funcionários de finanças, digitamos: deny tcp 172.16.2.0 0.0.0.255 172.16.3.2 0.0.0.0 e na sequência negamos o acesso para todos os demais funcionários de vendas ** deny tcp 172.16.0.0 0.0.0.255 172.16.3.2 0.0.0.0**
Por fim, devemos permitir todas as demais comunicações, digitamos: permit ip any any
Devemos agora informar que essa política de acesso deve ser implementada nas sub-interfaces de finanças e vendas no sentido de entrada. Saímos da configuração da lista de acesso digitando exit e na sequência entramos na sub-interface da Vlan de vendas (por exemplo: interface FastEthernet 0/0.1). Para fazermos essa associação com a lista de acesso, digitamos: ip access-group SERVIDOR-GERENTES in
Agora saímos da parte de configuração sub-interface da Vlan de vendas, digitando exit e na sequência entramos na sub-interface do setor de finanças (por exemplo: interface FastEthernet 0/0.2) e fazemos a associação com a lista de acesso digitando: ip access-group SERVIDOR-GERENTES in
Vá aos computadores, na aba dos Web Browsers e tente acessar o servidor. Qual é o resultado?

Opinião do instrutor

Uma vez que alteramos a política de acesso, somente os computadores dos gerentes de vendas e finanças continuarão acessando a página de login presente no servidor. Os demais usuários não terão permissão de acesso e com isso a página de login não será mostrada.
lista_acesso

#### 19/08/2023

@03-Provedor de serviço

@@01
Conectando ao provedor de serviços

Conseguimos obter um grande avanço até aqui, dividindo a rede interna em redes menores por meio das VLANs, e conseguiremos trabalhar de uma forma mais eficiente com os "endereços IP" usando as sub-redes.
Com as Listas de Acessos, definimos que somente o computador do gerente de vendas e o do gerente de finanças, tivessem acesso à página de login que está no servidor. Mas sabemos que sem o uso da internet, fica difícil para uma empresa desenvolver muitos negócios.

Então, a nossa tarefa é realizar a contratação de um serviço de provedor, e ver em detalhes como é feita essa conexão na rede do provedor de serviços para a nossa empresa.

Observe a imagem a seguir:

Diagrama da contratacao do servico de provedor de internet

Suponhamos que fizemos a contratação de serviços de um provedor. O funcionário do provedor vai pegar um ponto de acesso que esteja na rua, por exemplo, que seja da rede dele, e vai trazer um cabo para o nosso estabelecimento.

A conexão que é feita com o nosso estabelecimento é chamada de Ponto de Demarcação! Podemos citar um Wall Jack Ethernet como exemplo de ponto de acesso, em que o provedor de serviços realiza a conexão de fora para dentro do estabelecimento.

Os roteadores mais antigos tinham um modem externo, que se chamava CSU/DSU. Do ponto do demarcador, conectaríamos ao wall jack. Por fim, o wall jack se conectaria ao roteador que está em nossa empresa.

Antigamente, a conexão era feita do CSU/DSU para o roteador, por meio de um cabo chamado de V.35, que era um cabo serial. Atualmente, a conexão usando o cabo serial é pouco utilizada. O CSU/DSU já vem embutido das placas de rede atualmente, então, não precisaremos fazer todo esse caminho. Podemos encurtá-lo, fazendo a conexão do wall jack direto para a porta Rj45 que está no roteador.

Nas representações de diagramas de redes, é comum realizar a conexão de uma rede externa (que pertence a um outro servidor) com um cabo serial, para justamente saber qual é o ponto demarcador da nossa rede, para a rede de uma outra empresa.

Vamos utilizar no nosso projeto, esse cabo serial. Esta é a forma mais comum para fazer esse tipo de representação quando estamos conectando redes de diferentes empresas.

Adicionaremos um roteador, que será representado como o roteador do provedor de serviços. Clicando nesse roteador, teremos a visão física:

Visao fisica do dispositivo roteador do provedor

Precisaremos instalar essa placa serial, mas antes, teremos que desligar esse roteador, clicando no botão:

Desligar o roteador

Depois, clicando na opção "WIC-1T", arrastaremos para o primeiro espaço da placa:

Arrastar WIC1T

Feito isso, vamos instalar a placa serial no roteador, e depois podemos ligá-lo novamente.

Realizaremos o mesmo processo com o roteador que está em nossa empresa. Mas antes de desligá-lo, vamos salvar as configurações que estão na memória volátil.

Para salvar a memória não-volátil, faremos o seguinte:

>enable
#wrCOPIAR CÓDIGO
O comando #wr vem de write, e utilizamos para salvar a configuração. Uma vez salva toda a configuração na memória não-volátil, vamos clicar no roteador, na aba "Physical", desligar o roteador como fizemos no anterior, e arrastar novamente a placa serial WIC-1T. Agora ligamos o roteador novamente, e realizamos a conexão entre esses dois roteadores usando o cabo serial.

De acordo com a imagem abaixo, clicaremos no símbolo de um raio (Connections) que está à esquerda, e depois, no símbolo de um raio vermelho com um relógio, que está à direita.

Conectando ao provedor de servicos

Esse segundo símbolo indica a taxa de transmissão que o provedor de serviços vai estar fornecendo para nós.

Para especificar quem fornecerá essa taxa será o provedor, clicaremos primeiro no roteador do provedor, e escolhemos a rota Serial0/1/0 e conectamos na porta Serial0/1/0 do roteador da nossa empresa.

Roteadores conectados

Agora, precisamos configurar as interfaces dessa conexão. Lembrando que na internet, não podemos utilizar endereços IPs privados. Esses endereços são aqueles que vimos nas outras aulas, que começam com 10, ou com 172.16 a 172.31, ou até mesmo que começavam por 192.168. Esses endereços só podem ser usados para a comunicação interna.

A partir do momento que saímos da rede interna e vamos para a internet, precisaremos de um endereço IP público. Esses endereços são fornecidos pelo provedor de serviços, para que os usuários que estão em nossa rede interna possam ter os seus endereços IPs traduzido para esse IP público (NAT).

A primeira etapa é colocar as interfaces dos roteadores constando como endereços IP públicos. Clicando no roteador do provedor, na aba "CLI", colocaremos os seguintes comandos:

>enable
#configure terminal
#interface serial 0/1/0
#no shutdownCOPIAR CÓDIGO
Depois que entramos no modo de configuração, acessaremos a interface serial e depois, habilitaremos a porta com o comando #no shutdown. Se observarmos no diagrama, veremos que as portas estarão verdes.

Ao criarmos as sub-redes internas da Multillidae, fizemos a melhor eficiência do uso dos endereços IPs. O provedor de serviços será diferente, pois os IPs públicos foram comprados dos órgãos governamentais, por isso, eles devem ser usados da melhor maneira possível. Mas qual é a melhor maneira possível dele atribuir os endereços IPs? Necessitaremos de um endereço IP público para a interface do roteador do provedor, e um outro endereço público para a interface do roteador da empresa.

Vamos verificar qual seria a sub-rede que traria a melhor eficiência para se trabalhar com somente dois endereços.

@@02
IP privado x IP público
PRÓXIMA ATIVIDADE

Onde os IP privados são utilizados em uma rede?

Os IP privados são usados somente para comunicação em uma rede interna
 
Os endereços IP privados são usados somente para comunicação em uma rede interna, diferentemente dos endereços IP públicos que são usados para comunicação na internet
Alternativa correta
Os IP privados são usados somente nas redes dos provedores de serviços
 
Alternativa correta
Os IP privados são usados somente para comunicação na internet
 
Alternativa correta
Os IP privados são usados somente na conexão entre nosso roteador e o modem do provedor de serviços

@@03
Conexão demarcador
PRÓXIMA ATIVIDADE

Como eram realizadas antigamente as conexões dos pontos demarcadores com os roteadores internos de empresas?

 internos de empresas?
Alternativa correta
A conexão era realizada entre o ponto demarcador com um equipamento chamado de CSU/DSU com um cabo serial e novamente ocorria a conexão entre o CSU/DSU e o roteador empresarial com um cabo serial
 
Alternativa correta
A conexão era realizada com um equipamento chamado de CSU/DSU, que era capaz de converter os sinais do provedor de serviços para uma transmissão serial, então ocorria a necessidade do uso de um cabo serial para conexão com a placa do roteador.
Alternativa correta
A conexão era realizada entre o ponto demarcador com um equipamento chamado de CSU/DSU que simplesmente atuava como um Hub, repetindo o sinal recebido por todas as portas e ocorria a conexão de um cabo serial entre o roteador empresarial o CSU/DSU
 
A conexão desse ponto demarcador era realizada com um equipamento chamado de CSU/DSU (Channel Service Unit/Data Service Unit). Tal equipamento era responsável por converter o sinal fornecido pelo provedor de serviços para uma transmissão serial, atuando como um modem. Dessa forma, utilizava-se um cabo serial, sendo o mais comum um cabo chamado de V.35 para conexão com uma placa serial presente no roteador.
Hoje em dia, essa conexão serial foi substituída pela conexão com o cabo de rede que estamos acostumados uma vez que o equipamento CSU/DSU já está presente nas placas de redes dos roteadores empresariais mais modernos, não sendo necessário mais o uso dos cabos seriais. Embora, os cabos seriais hoje em dia não sejam mais utilizados, sua representação é comum em diagramas de rede como referência a conexão a rede de uma outra empresa.

@@04
Configurando sub-redes

Como vimos anteriormente, o provedor de serviços tentará alocar os endereços IPs da forma mais eficiente possível, afinal, ele pagou por esses endereços IPs públicos para os órgãos governamentais.
Em nosso cenário, iremos precisar de um endereço IP público para o roteador do provedor de serviços e também de um IP público para a interface que irá para o roteador da nossa empresa. Vamos voltar ao nosso diagrama do café para ver qual seria a melhor eficiência para entregar esses 2 endereços IPs para os clientes.

Diagrama do café

Os dois grãos de café serão representados pelos dois endereços IPs que iremos precisar. Sabemos que até a prateleira com quatro grãos de café, não podemos colocar nenhum post-it, senão teremos prejuízo. Por isso, usaremos a prateleira com dois grãos. As prateleiras que não tem post-it, colocaremos o bit 0, nas com post-it, colocaremos o bit 1.

Mascara de rede sem nenhuma informacao

Temos a máscara de rede sem nenhuma informação e nós precisamos reservar uma parte da máscara para alocar dois endereços IPs.

Começaremos a análise da esquerda para a direita, e precisamos colocar dois bits 0, pois os 0 na máscara de rede, é referente aos hosts.

Zeros representando hosts

Uma vez que já reservamos esses bits 0 em nossa máscara de rede, o restante pode ser deixado para a parte da rede mesmo. Com isso, descobrimos a melhor eficiência para nós.

Então se lembrarmos da transformação dos números binários para os decimais, teremos a sequência de 8 bits 1.

Sequencia de 8 bits de 1

Faremos algumas contas para calcular o último intervalo. Onde tiver o bit 1, consideramos na soma. Na última posição do intervalo, (da direita para a esquerda), temos a posição com 128 grãos de café. Em seguida, temos a posição de 64 grãos. Somando o valor dessas duas posições, mais 32 que equivale a 32 grãos, mais 16, mais 8, e por fim, o 4. Nas próximas posições, teremos o bit 0, e não consideraremos na soma.

O valor decimal respectivo à sequencia binária é 252.

Ultima sequencia binaria descoberta

Agora, vamos descobrir qual é o endereço máximo disponível para essa máscara de rede que terá a melhor eficiência para entregar esses dois endereços IP. Como estamos usando base binária, então a conta é 2^, 2 elevado a quantidade de bits 0 (hosts). Temos que levar em conta que não podemos atribuir para nenhuma máquina o endereço IP de rede e de broadcast, por isso, subtraímos o valor por 2.

 Endereços Disponíveis: (2^2) - 2 = 2COPIAR CÓDIGO
Essa máscara de rede conseguirá alocar exatamente dois endereços IP. Esse tipo de máscara de rede ("255.255.255.252"), acaba sendo muito utilizada para os links de ponto a ponto, pois iremos precisar somente de 2 endereços IPs, e com essa máscara, conseguiremos obter exatamente a atribuição de dois endereços IPs.

Seguiremos os Três Passos para poder verificar qual seria o incremento da sub-rede.

O primeiro passo era Transformar a máscara em binário.

Transformar a mascara em binário

O segundo passo era Verificar onde ocorre a transição da sequência do bit 1 e do bit 0.

 Verificar a transicao da sequencia de bits
O terceiro passo era transformar a posição do bit 1 em decimal
Transformar a posicao do bit 1 em decimal
Agora, pegaremos um endereço IP público de exemplo 150.1.1.0. Para definir as subredes, colocaremos esse endereço na primeira sub-rede. Para o endereço IP da próxima sub-rede, pegamos o intervalo onde teve a transição do bit 1 para o bit 0, e adicionamos o valor respectivo a posição inicial do bit 1. No nosso caso, esse valor é igual a 4! Com isso, o endereço será 150.1.1.4.
O IP de Broadcast da sub-rede 1 é o endereço da sub-rede 2, menos 1, ou seja, será 150.1.1.3. O IP de Broadcast da sub-rede 2 é o endereço da sub-rede 2, mais 4, e aí nós subtraímos 1. Ficaria 150.1.1.7. Cada sub-rede vai estar totalmente isolada uma da outra.
Poderíamos alocar para o link, a primeira sub-rede do nosso exemplo. Quais os endereços IP disponíveis dentro dela? Temos somente dois IPs disponíveis: 150.1.1.1 e o 150.1.1.2.
Vamos clicar no roteador do provedor, na aba "CLI", usaremos os comandos:
#interface serial 0/1/0
#ip address 150.1.1.1 255.255.255.252COPIAR CÓDIGO
Agora, configuraremos o próximo roteador:
>enable
#configure terminal
#interface serial 0/1/0
#ip address 150.1.1.2 255.255.255.252COPIAR CÓDIGO
Agora vamos testar a conectividade desse roteador, para o roteador de serviços.
Voltando para o modo privilegiado com o "Ctrl + Z" colocaremos:
#ping 150.1.1.1COPIAR CÓDIGO
Quando temos os pontos de exclamação como resposta, significa que a conectividade foi estabelecida com sucesso. Se tentarmos pingar um endereço de outra sub-rede, não vamos ter sucesso, pois são redes distintas e por padrão, eles não se conversam.
Desta forma, utilizando sub-redes, conseguiremos trabalhar com endereços IP de forma muito eficiente. Aqui, conseguimos configurar o endereço IP público e atribuído pelo nosso provedor de serviço, o próximo passo será traduzir os endereços privados dos computadores dos funcionários, para o endereço publico. Isso será feito por meio do NAT.

@@05
Provedor de serviços
PRÓXIMA ATIVIDADE

O que seria o ponto demarcador?

O ponto demarcador seria uma outra nomenclatura usada para listas de acesso, pois com a lista de acesso estamos demarcando os tráfegos que podem entrar e sair de nossa rede.
 
Alternativa correta
O ponto demarcador seria uma ferramenta utilizada por provedores de serviços para continuar a instalação de cabos caso ocorra algum interrupção durante o serviço de instalação.
 
Alternativa correta
O ponto demarcador seria uma limitação entre a rede interna do cliente final e a rede do provedor de serviços, dividindo assim as responsabilidades existentes.
Alternativa correta
O ponto demarcador seria uma nomenclatura utilizada para o Switch root de uma rede, pois ele que vai demarcar quais serão as portas bloqueadas no caso da existência de loops.
 
O ponto demarcador seria um ponto instalado nas premissas do cliente o qual divide as responsabilidades na rede entre o provedor de serviços e o cliente final. Caso ocorra algum problema do ponto de demarcação para a rede interna, seria responsabilidade do cliente. Caso ocorra algum problema do ponto de demarcação para a rede externa, seria responsabilidade do provedor de serviços.
Parabéns, você acertou!

@@06
Mãos à obra: Conectando ao provedor de serviços
PRÓXIMA ATIVIDADE

Iremos agora conectar nossa rede com a do provedor de serviços, ganhando assim acesso a internet.
Arraste para a área de trabalho o roteador modelo 1841. Embora a conexão serial do modem chamado CSU/DSU para o roteador interno não seja mais utilizada nos dias de hoje, essa representação é comum em diagramas para mostrar que estamos nos conectando a uma rede externa, de um outro provedor, empresa, etc. Para isso, nós clicamos nesse novo roteador, vamos até a aba Physical e precisamos desligar esse roteador. Em seguida clicamos na placa de rede serial WIC-1T e arrastamos para o slot vazio do roteador. Por fim devemos, ligar o roteador novamente.
serial

Devemos inserir essa placa serial no roteador interno, porém antes nós devemos salvar as configurações na memória não volátil do roteador. Clique no roteador e vá até a aba CLI. Digite enable para entrar na parte privilegiada e em seguida digitamos wr
Agora podemos desligar o roteador e inserir a placa de rede. Clique na aba Physical, desligue o roteador e arraste a placa WIC-1T para um slot disponível e ligue o roteador.
Na sequência, devemos conectar esses dois roteadores. Clicamos no símbolo do raio e escolhemos a opção Serial DCE (oitava opção da esquerda para a direita). O DCE será o equipamento o qual irá enviar a taxa de transmissão, que virá do provedor de serviços. Clicamos primeiro no roteador do provedor de serviços escolhemos a porta serial que implementamos e na sequência clicamos no roteador interno e escolhemos a porta serial que configuramos. Devemos ter no fim esse cenário:
provedor_servicos

Devemos agora habilitar as portas dos dois roteadores e configurar os endereços IP. O provedor de serviços pagou pelos endereços IP públicos, então ele vai tentar usar da forma mais eficiente possível. No nosso cenário, precisamos somente de dois endereços IP um para o provedor de serviços e outro para a interface serial do roteador interno.

Clique no roteador do provedor de serviços e vá até a aba CLI. Entre na parte privilegiada digitando enable e na sequência entre na parte de configuração digitando configure terminal
Entre na interface serial utilizada para conectar ao roteador interno (por exemplo: interface serial 0/1/0) e habilite essa interface colocando no shutdown e insira o endereço IP 150.1.1.1, digitando: ip address 150.1.1.1 255.255.255.252.
Na sequência, clique no roteador interno digite enable para entrar na parte privilegiada e posteriormente coloque configure terminal para entrar na parte de configuração. Entre em seguida na interface serial (por exemplo: interface serial 0/1/0) e coloque o endereço IP 150.1.1.2, digitando: ip address 150.1.1.2 255.255.255.252.
Teste a conectividade com o roteador do provedor de serviços, digitando: do ping 150.1.1.1
Qual o resultado?

Opinião do instrutor

Uma vez que os endereços IP da interface serial do roteador interno e a interface serial do roteador do provedor de serviços estão dentro da mesma sub-rede a comunicação é estabelecida com sucesso.
ping_provedor

#### 20/08/2023

@04-Network Address Translation (NAT)

@@01
Configurando o NAT

Até aqui, conseguimos configurar os endereços IPs entre o roteador do provedor de serviços e o roteador da nossa empresa Multillidae. Nesta aula, realizaremos a tradução dos endereços IPs privados para o endereço público que foi alocado na interface serial do roteador da Multillidae.
Com o NAT, faremos essas traduções de IPs privados para IPs públicos. Para realizarmos essas configurações, clicaremos no roteador da empresa, na aba "CLI", e executaremos os comandos a seguir. Será necessário criar uma lista de acessos. No final da criação da lista, temos que especificar se esta será uma lista extended ou uma lista standard. No caso, só estamos preocupados em realizar a tradução dos endereços IPs da origem, e não o que eles irão acessar. Por causa disso, usaremos a opção standard:

>enable
#configure terminal
#ip access-list standard NATCOPIAR CÓDIGO
Pronto! Criamos a nossa lista de acessos. Temos que indicar os endereços IPs que queremos realizar essa tradução.

O endereço 172.16.0.0 foi alocado para os funcionários do setor de Vendas, e o endereço 172.16.2.128 foi alocado para os funcionários de Finanças.

Então, para fazer a permissão, usaremos também o Wildcard bits. Lembrando que 0 faz com que a parte do endereço seja exatamente igual, o 255 nos permite colocar qualquer valor. Tanto a rede de Vendas, como a rede de Finanças, as duas inicializam com 172 e o segundo intervalo é 16.

#permit 172.16.0.0COPIAR CÓDIGO
Isso quer dizer que esses dois valores precisam ser exatamente iguais. Entretanto os dois últimos intervalos podem ser qualquer valor. Por isso, a máscara da subrede ficará assim:

#permit 172.16.0.0 0.0.255.255COPIAR CÓDIGO
Em seguida, configuraremos nas interfaces do roteador, a interface interna, onde ficarão os IPs privados, e a interface externa, onde estará o IP público fornecido pelo provedor.

No roteador, na aba "CLI", acessaremos primeiro a interface 0.1, que é a subinterface do setor de vendas:

#exit
#interface fastEthernet 0/0.1COPIAR CÓDIGO
Diremos que essa subinterface é interna:

#exit
#interface fastEthernet 0/0.1
#ip nat inside
#exitCOPIAR CÓDIGO
Depois dessa configuração entraremos na interface 0.2, que é a interface do setor de finanças. Especificaremos também que essa interface está conectada com a parte interna da nossa rede.

#interface fastEthernet 0/0.2
#ip nat inside
#exitCOPIAR CÓDIGO
Entraremos na interface serial que é a externa, e atribuiremos a característica de outside:

#interface serial 0/1/0
#ip nat outside
#exitCOPIAR CÓDIGO
Precisaremos vincular a lista de acessos que permite os endereços que começam com 172.16., sejam traduzidos para o endereço IP público 150.1.1.2. E para fazer essa configuração, faremos:

#ip nat inside source list NAT interface serial 0/1/0COPIAR CÓDIGO
Com esse comando, diremos que será traduzido os endereços IPs que estão na rede (inside), e especificamos quais são esses endereços IPs que estão na lista NAT para a interface serial 0/1/0.

Entretanto, seremos um pouco cuidadoso, porque só temos um endereço IP público, mas temos vários usuários na rede que podem estar acessando a internet ou qualquer outro recurso externo simultaneamente. Para especificar que a configuração da tradução tem que englobar todos esses endereços IPs internos de forma simultânea, colocaremos o overload:

#ip nat inside source list NAT interface serial 0/1/0 overloadCOPIAR CÓDIGO
Com o overload, é dito que todos os usuários podem estar usando simultaneamente o endereço IP público 150.1.1.2. Depois disso, usamos a tecla "Enter" e teoricamente já criamos a lista de acessos com os endereços IPs que devem ser traduzidos. Nós especificamos as interfaces internas e externas, e associamos a nossa lista com esses IPs privados, para ela ser traduzida para a interface serial 0/1/0, que irá conter o endereço público, e também especificamos a forma de tradução overload, dizendo que pode ocorrer mais de um usuário estar usando um recurso da internet ao mesmo tempo.

Veremos se, de fato, a tradução está sendo feita no roteador do provedor, e se ele está traduzindo os pacotinhos que chegam.

Testaremos primeiro a conectividade. Clicando no computador do gerente de finanças, em "Command Prompt", usaremos o comando ping:

> ping 150.1.1.1COPIAR CÓDIGO
Como você pode ver, a conexão está acontecendo normalmente.

Testaremos agora a tradução. Mudaremos do modo Realtime para o Simulation, e realizaremos o Ping novamente no computador do gerente de finanças.

>ping 150.1.1.1COPIAR CÓDIGO
Clicando em "Capture / Forward", veremos que o pacotinho vai ser transferido até chegar ao roteador. Do roteador, deve acontecer a tradução do IP privado. Observe a imagem:

Traducao do endereco Ip privado para o publico

O IP é o 172.16.2.131 que será traduzido para 150.1.1.2.

Endereco IP privado ja traduzido para publico

Repare que agora o IP de origem é 150.1.1.2. Isso significa que quando esse pacotinho passou pelo roteador, houve a tradução!

Tarefa executada com sucesso!

@@02
Funcionalidade do NAT
PRÓXIMA ATIVIDADE

Qual seria a funcionalidade do NAT?

O NAT seria um outro nome utilizado para Vlan nativa, indicando todo o tráfego o qual nenhuma Vlan for especificada.
 
Alternativa correta
O NAT seria a forma utilizada para realizar roteamento em Swithes.
 
Alternativa correta
O NAT seria a forma utilizada para transformar uma rede Classful em Classless.
 
Alternativa correta
O NAT seria a forma utilizada para traduzir endereços IP privados para públicos.
O NAT (Network Address Translation) é usado para realizar a tradução de endereços IP privados, que só podem ser usados em uma rede interna para endereços IP públicos que podem ser usados para acessar a internet.

@@03
Comunicação e o NAT
PRÓXIMA ATIVIDADE

Nós trabalhamos em uma empresa e foi solicitado que configurássemos essa rede empresarial para que fosse possível acessar o servidor da internet. No fim de semana, precisamos acessar esse servidor de uma localidade externa, então acessamos o browser e colocamos o endereço IP 192.168.0.20 que seria o endereço IP do servidor que está interno na rede da empresa. Porém o resultado não foi satisfatório, qual seria uma das possíveis causas?

Uma das possíveis causas seria que o endereço IP 192.168.0.20 é um endereço IP de rede quando atuamos em rede classful e dessa forma, não poderá ser atribuído para nenhuma máquina
 
Alternativa correta
Uma das possíveis causas seria que o endereço IP 192.168.0.20 em redes classful é um endereço de Broadcast.
 
Alternativa correta
Trata-se de um endereço IP privado. Uma vez que o NAT realiza traduções de endereços IP privados para públicos, usuários da internet só conseguirão visualizar o endereço IP público por padrão.
Alternativa correta
Uma das possíveis causas seria que em redes classful esse endereço IP é de uso reservado da classe D e com isso não conseguiremos acessar.
 
O endereço IP 192.168.0.20 é privado. Uma vez que estamos em uma rede externa nós só conseguimos visualizar o endereço IP público da rede da empresa e não conseguimos visualizar os endereços IP privados que estão configurados internamente, isso porque ocorre a tradução de endereços IP privados para públicos através do NAT.
Uma possível solução seria utilizar uma rede virtual privada, permitindo autenticação de usuários para acessar a rede interna e consequentemente o servidor. Tal tecnologia é conhecida como Virtual Private Network (VPN), para saber maiores detalhes: https://pt.wikipedia.org/wiki/Virtual_private_network

https://pt.wikipedia.org/wiki/Virtual_private_network

@@04
Standard x Extended
PRÓXIMA ATIVIDADE

Diferentemente da configuração da lista de acesso que fizemos para o servidor interno que usamos o tipo extended, utilizamos agora a lista de acesso do tipo standard. Porque fizemos essa alteração?

Na configuração do NAT há uma preocupação da origem e dos endereços de destino os quais os dispositivos vão acessar, dessa forma, a escolha do tipo standard ocorreu por conta que esse tipo faz uma análise dos endereços de origem e destino
 
O tipo standard faz análise somente dos endereços de origem e não dos endereços de origem e destino como especificado na resposta
Alternativa correta
Na configuração do NAT há uma preocupação somente com o destino os quais os endereços de origem vão acessar, dessa forma, a escolha do tipo standard ocorreu por conta que esse tipo faz uma análise dos endereços de destino
 
O tipo standard faz análise somente dos endereços de origem e não dos endereços de destino como especificado na resposta
Alternativa correta
Na configuração do NAT não há preocupação com endereços de origem e destino e dessa forma, a utilização do tipo standard é utilizada para informar que nenhum tipo de endereço, nem de origem e nem de destino deverá ser analisado
 
O tipo standard faz análise dos endereços de origem, diferentemente da resposta que diz que não há análise de nenhum tipo de endereço
Alternativa correta
Na configuração do NAT não há uma preocupação de qual destino os computadores irão acessar, por isso estamos só preocupados com a origem dos endereços e o tipo standard analisa somente a origem
 
O tipo standard analisa a origem dos endereços e nesse caso estamos só preocupados com a origem e não com o possível destino que esses endereços planejam acessar

@@05
Mãos à obra: Configurando NAT
PRÓXIMA ATIVIDADE

Dentro da rede interna estamos usando endereços IP privados, os endereços IP privados são usadas somente para comunicação na rede interna, não podendo ser utilizados para comunicação na internet. Dessa forma, devemos traduzir os endereços IP privados para o endereço IP público (150.1.1.2), essa tradução entre endereços IP privados para endereços IP públicos é chamado de Network Address Translation (NAT). Devemos realizar isso no nosso projeto:
Clique no roteador interno da Mutillidae e vá até a aba CLI
Entre na parte privilegiada digitando enable e posteriormente entre na parte de configuração digitando configure terminal
O primeiro passo será criar uma lista de acesso para que possamos informar os endereços IP privados que queremos traduzir para o endereço IP público. Digitamos ip access-list standard NAT
Na sequência, digitamos os endereços IP permitidos na tradução, digitamos permit 172.16.0.0 0.0.255.255 Dessa forma, estamos informando que todos os endereços IP devem começar por 172.16, os demais intervalos podem ter qualquer valor, porque o WildCard bits está 255.
Uma vez que criamos a lista de acesso dos endereços IP privados para serem traduzidos, devemos configurar as interfaces que serão da parte interna e a interface que será da parte externa.
Digite exit para sair da parte de configuração da lista de acesso. Entre na sub-interface do setor de vendas (por exemplo: interface FastEthernet 0/0.1) e especificamos como sendo a parte interna, digitamos: ip nat inside.
Na sequência, devemos sair dessa sub-interface digitando exit e na sequência entramos na sub-interface da Vlan de finanças (por exemplo: interface FastEthernet 0/0.2) e especificamos como sendo a parte interna, digitamos: ip nat inside.
Saímos dessa interface digitando exit e em seguida precisamos entrar na interface serial (por exemplo: interface serial 0/1/0) e especificamos como sendo a parte externa, digitamos: ip nat outside
Por fim devemos associar a lista que criamos para ser traduzida para esse endereço IP público, digitamos: ip nat inside source list NAT interface Serial0/1/0
Clique em um dos computadores e posteriormente vá até aba Command Prompt e digite ping 150.1.1.1 que seria o endereço IP do roteador do provedor de serviços. Em seguida, volte ao roteador e volte a porte privilegiada digitando CTRL+Z e digitando em seguinda show ip nat translations. Qual o resultado?

Uma vez que realizamos a configuração do NAT, a tradução dos endereços IP privados para o endereço IP público 150.1.1.2 é estabelecida com sucesso e dessa forma, nossos usuários irão poder acessar a internet.

@@06
Conclusão

Chegamos ao final da terceira parte do curso, e vimos a questão das configurações de políticas de acesso em nosso roteador.
Configuramos as políticas de acesso para permitir que somente o computador do gerente de Finanças e o computador do gerente de Vendas tivessem acesso aos recursos do servidor, e negamos os computadores dos funcionários de vendas e finanças de acessarem ao servidor.

Vimos a questão das políticas de acesso, na qual a análise é feita sequencialmente, conforme digitamos essas políticas. Se houver algum pacote e não houver nenhum tratamento para esse tráfego, ele será descartado e não passará.

No final, criamos uma política para configurar todas as outras comunicações para continuarem a ser estabelecidas, por meio do permit ip any any.

Depois, usando o NAT, conseguimos fazer a tradução dos endereços IPs privados, que foram alocados nos computadores dos funcionários da empresa, para endereços IPs públicos. Conseguimos também, estabelecer a comunicação com esse roteador do provedor de serviços.

Esperamos você na quarta parte do Curso de Redes!
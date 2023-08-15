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
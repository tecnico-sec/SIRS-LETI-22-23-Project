# SIRS-LETI-22-23-Project

Instituto Superior Técnico, Universidade de Lisboa

**Segurança Informática em Redes e Sistemas**

# Objetivos 

O objetivo do projeto consiste em aprender a instalar e configurar diversos mecanismos de segurança, aprofundando os conhecimentos das aulas teóricas. 
Em concreto, o objetivo do projeto é proteger a rede do banco CAIXINHA. O banco tem a sede em Lisboa e diversas sucursais, das quais vamos considerar apenas duas: Caixinha1 e Caixinha2. A comunicação entre a sede e as sucursais é feita através da Internet e portanto exposta a um risco elevado. O banco está preocupado com a confidencialidade e integridade dos dados do negócio, por exemplo, dos dados das contas dos seus clientes. A rede do banco e a Internet serão emuladas usando o software *Kathará*.

# Entrega

A entrega do projeto tem 3 componentes: 
*	um diagrama da rede do banco; 
*	a configuração da rede; 
*	um questionário a reportar o que foi feito. O questionário é longo e detalhado, logo deve ser preenchido durante a realização do projeto.

# Descrição da rede

A rede do CAIXINHA contém os dispositivos habituais neste tipo de redes: servidores, estações de trabalho, routers, switches (também designados por bridges) e dispositivos de segurança. A configuração básica da rede já está feita e é fornecida sob a forma de um laboratório Kathará, mas ainda não inclui mecanismos de segurança. 

A rede tem quatro partes:
1.	Sede: inicialmente com endereços da gama 99.2.0.0/20;
2.	Sucursal Caixinha1: inicialmente com endereços da gama 99.4.3.0/24;
3.	Sucursal Caixinha2: inicialmente com endereços da gama 99.5.3.0/24;
4.	Internet: representada de forma simplificada por um único router e alguns PCs de testes (que poderiam ser controlados por hackers).

A *sede* tem 4 subredes, interligadas por três routers:
*	uma subrede que contém apenas uma firewall, em concreto, um packet filter (firewallS1);
*	a subrede de serviços (uma DMZ), onde se encontram serviços expostos aos clientes do banco que lhes acedem através da Internet, e que contém: 
    *	um switch (switchS1), 
    *	um servidor web (web-server), 
    *	um detetor de intrusões (ids);
*	a subrede corporate, onde se encontra:
    *	um switch (switchS2),
    *	um servidor de ficheiros (file-server1), 
    *	duas estações de trabalho de colaboradores do banco (ws1, ws2),
    *	um router de acesso à subrede business;
 *	a subrede business onde se encontram os dados críticos do negócio (p.ex. das contas bancárias), onde estão:
    * uma firewall (firewallS2),
    * um switch (switchS3), 
    * um servidor de base de dados (db1) e
    * um servidor de ficheiros (file-server2).

As *sucursais* Caixinha1 e Caixinha2 contém cada uma: 
* um router de acesso à Internet que também serve de firewall (router-fwC1, router-fwC2), 
* um switch (switchC1, switchC2) e 
* apenas uma estação de trabalho (wsC1, wsC2) por simplicidade. 

Quanto ao software instalado nas estações de trabalho e nos servidores:
*	estações de trabalho: não contêm software específico, só o que já vem na imagem;
*	servidor web: executa o Apache2 (que tem de ser configurado), contendo uma página simples de apresentação do banco;
*	servidores de ficheiros e de bases de dados: também não vamos usar nenhum software específico, mas emulá-los usando o comando nc em modo servidor à escuta no porto TCP/X, onde **X = 10000 + número do grupo**. 

Para testar os servidores emulados usando o comando nc executa-se: `echo “mensagem” | nc <ip_destino> <porto_destino>`

Por simplicidade não existe DNS, logo toda a comunicação tem de ser feita usando endereços IPv4.

# Personalização da rede

Acima está dito que os dispositivos têm *inicialmente* determinados endereços IP pois **os endereços IP têm de ser alterados pelo grupo de trabalho. É obrigatório substituir o número *99* pelo número do grupo.** Por exemplo, para o grupo 33, a rede da sede vai ter endereços da gama 33.2.0.0/20.

A primeira tarefa de cada grupo consiste em fazer essa substituição e fazer um diagrama da rede detalhado, usando qualquer software adequado para esse efeito. Têm de ser representados todos os dispositivos, indicados o seu nome e os endereços IP de todas as interfaces de rede que tenham endereços IP. Este diagrama serve para o grupo usar como referência, mas tem também de ser entregue (ver prazos abaixo).

# Testes

Para testar o funcionamento da rede e dos serviços que serão concretizados é fundamental usar ferramentas de teste adequadas. Além de comandos bem conhecidos como o *ping*, o *tcpdump*, o *traceroute*, o *tcpdump* e o *nmap*, recomenda-se o uso do *netcat (nc)* que permite estabelecer conexões TCP e enviar texto sobre essa conexão:
*	Para ficar à escuta numa porta na máquina de destino (servidor): `nc -l -p <porta>`
*	Para enviar pacote TCP executar no client: `echo <string_a_enviar> | nc <ip_destino> <porta_destino>`
*	A string deve aparecer no terminal do servidor.

# Mecanismos de Segurança 

A rede está configurada para o Kathará, mas não estão configurados os mecanismos de segurança. O projeto consiste em implementar 4 tipos de mecanismos de segurança: VPN, SSH, firewall (packet filter) e detetor de intrusões.

## VPN - OpenVPN

Cada uma das sucursais está ligada à sede através de uma VPN. Configure essa VPN usando o software OpenVPN em modo *routing* [2][14]. O router de acesso à Internet da sede será um cliente OpenVPN e o router de acesso de cada uma das sucursais será um servidor OpenVPN. Todo o tráfego entre a sede e as sucursais tem de ser encaminhado através de um túnel entre os dois routers. Portanto, se alguém na Internet tentar escutar essa comunicação:
*  observará tráfego cifrado, logo ilegível;
*  observará como endereços IP de origem e destino os do cliente e servidor OpenVPN, não os endereços IP internos da empresa.

A comunicação tem de ser cifrada usando AES com chaves de 256 bits e modo GCM. As chaves do cliente OpenVPN devem ser geradas no cliente e não no servidor OpenVPN.

Teste se a comunicação está a ser efetivamente encaminhada através da VPN escutando a comunicação na Internet.

## SSH - OpenSSH

O banco tem um administrador de rede que tem de poder trabalhar remotamente no servidor web. O administrador tem uma conta nesse servidor que usa para fazer o acesso usando o protocolo SSH. O administrador viaja muito e por isso, apesar de sob o ponto de vista de segurança não ser muito boa ideia, abriu o acesso por SSH a esse servidor a partir de qualquer ponto do mundo. Por exemplo, pode fazer esse acesso a partir de um PC ligado à Internet, como o *test1* no caso da nossa rede emulada. 

Configure o protocolo SSH fornecido pelo pacote OpenSSH [4][13] de modo a permitir esse acesso. A autenticação tem de ser baseada em criptografia de chave pública, não em password. 
Teste o funcionamento desses protocolos usando os comandos: 
*	*ssh* para fazer login remoto e 
*	*scp* para copiar ficheiros entre os dois computadores.

Teste se a comunicação está a ser efetivamente cifrada, escutando a comunicação na Internet.

## Firewall - iptables

A rede do banco CAIXINHA tem várias firewalls ou packet filters mas vamos configurar apenas as seguintes:
*	firewallS1;
*	firewallS2;
*	router-fwC1.

Configure estas firewalls usando o netfilter / iptables [1][12][15] de modo a concretizar a seguinte política de segurança:
*	Todos os pacotes não explicitamente permitidos pelo resto da política são proibidos.
*	É permitido fazer ping e traceroute entre todas as máquinas (só para efeitos de depuração de erros; em termos de segurança não é boa ideia permitir da Internet executar esses comandos para dentro da rede do banco).
*	Qualquer máquina da Internet e da rede do banco pode aceder ao servidor web do banco usando o protocolo HTTP (TCP/80) e SSH.
*	Só os computadores da subrede corporate podem comunicar com os da subrede business (os da subrede business só podem comunicar com os da subrede corporate).
*	As máquinas da subrede de serviços (DMZ) do banco podem responder aos pedidos que recebem (web).
*	Máquinas da subrede corporate e das redes das sucursais:
    *	podem aceder a servidores web 
        * da Internet, i.e., externos ao banco (via HTTP claro) e 
        * do próprio banco (neste caso via HTTP e SSH);
    *	podem aceder ao servidor de ficheiros file-server1.
*	Nenhum pacote pode entrar na sede ou das sucursais com um endereço IP de origem da gama de endereços dessas subredes (para bloquear IP Spoofing [11]);
*	Nenhum pacote pode sair da sede ou das sucursais com um endereço IP de origem fora da gama de endereços dessas subredes

Teste se cada firewall está a bloquear todo o tráfego que deve bloquear. Para testar se uma firewall está a negar o acesso a uma porta podem testar essa mesma porta usando o netcat.

## Detecção de Intrusões - snort

O detetor de intrusões *snort* [3][17] deve ser configurado de modo a detetar ataques e intrusões na rede. O sistema de deteção de intrusões (IDS) deve ser colocado na máquina designada *ids*. O switch ao qual esse computador está ligado está configurado para funcionar como hub, logo o IDS recebe todo o tráfego que passa por esse switch.

O primeiro passo é a instalação do software snort em si, pois este não está disponível na imagem *quagga* usada em todas as imagens do ficheiro lab.conf. Para o efeito é preciso criar uma nova imagem com o snort seguindo as instruções fornecidas no slide “Installing software inside a VM” [15].

No segundo passo, o snort tem de ser configurado para alertar para um conjunto de ataques. O snort contém um conjunto enorme de regras que é atualizado periodicamente. Estas regras e a configuração do snort estão geralmente disponíveis na pasta /etc/snort. No nosso caso interessam apenas dois ficheiros: o /etc/snort/snort.conf (onde está a configuração do snort) e o /etc/snort/rules/local.rules (onde devem colocar as vossas regras). Nota: no ficheiro /etc/snort/snort.conf é preciso dar à opção *config checksum_mode" o valor *none*.

Modifique as regras disponíveis de modo a gerar os seguintes alarmes (ou seja, as seguintes mensagens na consola ou ficheiro de log):

*	Alarme “1: Acessos à subrede business!”. Um atacante obteve acesso ao servidor web ou a outra máquina da subrede de serviços e tenta comunicar com a subrede business (usando qualquer protocolo), o que não é permitido e deve gerar um alerta pois é preocupante.
*	Alarme “2: Tentativas sucessivas de accesso a login.html!”. Alguém está a fazer muitos pedidos à página *login.html* do web-server, o que pode constituir um ataque de força bruta contra a autenticação no site.

Teste a deteção dos ataques acima começando por definir comandos ou scripts que os executem.

# Entrega

O projeto é entregue no Fénix. Prazos e forma de entrega (sendo XXX o nº do grupo):
*	Até dia 20 de Maio às 17 horas: entregar um diagrama da rede sob a forma de um ficheiro chamado diagramaXXX.pdf no Fénix;
*	Até dia 12 de Junho às 17 horas: entregar todos os ficheiros do laboratório sob a forma de um ficheiro katharaXXX.tgz no Fénix. Incluir o laboratório Kathará com todos os ficheiros .startup e as subpastas. Se alguns comandos não puderem ser automatizados dessa forma, explicar e indicá-los no laboratório. Não é preciso entregar a imagem que contém o snort. Recomenda-se o uso do comanto *tar -Sczvf proj.tgz pasta_proj/* para comprimir eficazmente os ficheiros (*tar -xvf proj.tgz* para descomprimir);
*	Até dia 12 de Junho às 17 horas: entregar o relatório/questionário preenchido sob a forma de um ficheiro chamado relatorioXXX.pdf no Fénix. Esse relatório tem obrigatoriamente criado com base no ficheiro relatorio-questionario.docx fornecido;
*	Discussões nos dias 14 e 15 de Junho.

# Bibliografia

* [1] iptables Tutorial 1.2.2 https://www.frozentux.net/iptables-tutorial/iptables-tutorial.html 
* [2] OpenVPN Howto, https://openvpn.net/index.php/open-source/documentation/howto.html 
* [3] Snort Users Manual 2.9.12 – The Snort Project, 2018 
* [4] OpenSSH, http://www.openssh.org/
* [5] Kathará web page: https://www.kathara.org/  
* [6] Kathará Wiki: https://github.com/KatharaFramework/Kathara/wiki 
* [7] Kathará Labs: https://github.com/KatharaFramework/Kathara-Labs/wiki 
* [8] Kathará man pages: https://www.kathara.org/man-pages/kathara.1.html 
* [9] Guia de Configuração do Kathará: https://github.com/tecnico-sec/Kathara-Setup 
* [10] Guia de Laboratório - Network Routing: https://github.com/tecnico-sec/Kathara-Route 
* [11] Guia de Laboratório - Network Vulnerabilities: https://github.com/tecnico-sec/Kathara-NetVulns 
* [12] Guia de Laboratório - Web Server and Firewall: https://github.com/tecnico-sec/Kathara-WebServer-Firewall 
* [13] Guia de Laboratório - Secure Shell: https://github.com/tecnico-sec/Kathara-SSH 
* [14] Guia de Laboratório - Virtual Private Network: https://github.com/tecnico-sec/Kathara-VPN 
* [15] Miguel Correia. “Kathará”. Slides de Segurança Informática em Redes e Sistemas, LETI, Instituto Superior Técnico, Abril de 2022
* [16] Miguel Correia. "iptables: a brief introduction". Slides de Segurança Informática em Redes e Sistemas, LETI, Instituto Superior Técnico, Maio de 2022
* [17] Miguel Correia. "snort: a brief introduction". Slides de Segurança Informática em Redes e Sistemas, LETI, Instituto Superior Técnico, Maio de 2022



---

Caso venham a surgir correções ou clarificações neste documento, podem ser consultadas no histórico (_History_).

**Bom trabalho!**

[Os docentes de SIRS](mailto:leti-sirs@disciplinas.tecnico.ulisboa.pt)

# BRIDGE \| Infraestrutura Windows Server

Projeto acadêmico da disciplina **Operating Systems / Sistemas
Operacionais** --- FIAP, Global Solution 2026.2.

O BRIDGE propõe uma infraestrutura Windows Server para apoiar a solução
CitizenTech da equipe, com serviços de rede, identidade, hospedagem web,
monitoramento e acompanhamento operacional.

> **Escopo desta entrega:** este repositório documenta o ambiente
> Windows Server configurado e a página demonstrativa publicada no IIS.
> A página atualmente hospedada é um front-end estático em HTML, sem
> integração com o backend, banco de dados ou aplicativo mobile
> original.

## 1. Objetivos

-   Preparar um servidor Windows com endereço IP fixo e nome de host
    definido.
-   Configurar DNS e Active Directory para o domínio do laboratório.
-   Publicar uma página web por meio do IIS.
-   Definir permissões NTFS adequadas para o conteúdo publicado.
-   Monitorar CPU, memória e espaço livre em disco.
-   Configurar um alerta de desempenho para uso elevado de CPU.
-   Configurar a recuperação automática do serviço de publicação web.
-   Consultar os logs de acesso do IIS.

## 2. Ambiente utilizado

  Item                        Configuração
  --------------------------- ----------------------------------------
  Virtualização               Oracle VirtualBox
  Sistema operacional         Windows Server 2022 Desktop Experience
  Nome do servidor            `BRIDGE-SRV`
  Domínio AD / floresta       `bridge.local`
  Nome NetBIOS                `BRIDGE`
  IP IPv4 estático            `192.168.18.250`
  Máscara                     `255.255.255.0` (`/24`)
  Gateway IPv4                `192.168.18.1`
  DNS do servidor             `127.0.0.1` e `::1`
  Servidor DNS encaminhador   `8.8.8.8` e `8.8.4.4`

O servidor foi configurado como o primeiro controlador de domínio de uma
nova floresta `bridge.local`, com os níveis funcionais de floresta e
domínio definidos como Windows Server 2016.

## 3. Serviços e configurações

### 3.1 Rede e identidade

-   Interface de rede configurada com IPv4 estático `192.168.18.250/24`.
-   Hostname definido como `BRIDGE-SRV`.
-   Função DNS instalada.
-   Zona direta `bridge.local` criada.
-   Registro A `BRIDGE-SRV.bridge.local` apontando para
    `192.168.18.250`.
-   Zona reversa IPv4 criada, com registro PTR para o servidor.
-   Encaminhadores DNS configurados para `8.8.8.8` e `8.8.4.4`.
-   Active Directory Domain Services instalado.
-   Floresta e domínio `bridge.local` promovidos no próprio servidor.
-   Logon de domínio validado como `BRIDGE\Administrator`.

**Validações realizadas:** resolução do nome `BRIDGE-SRV.bridge.local`,
resolução reversa do IP `192.168.18.250` e resolução de nome externo
(`google.com`) utilizando o DNS do servidor.

### 3.2 Hospedagem web com IIS

A função **Web Server (IIS)** foi instalada no Windows Server.

A página demonstrativa BRIDGE foi publicada no diretório padrão:

``` text
C:\inetpub\wwwroot\index.html
```

A aplicação foi validada:

-   Localmente, pelo endereço `http://localhost`.
-   A partir do computador pessoal, pelo endereço
    `http://192.168.18.250`.

O acesso externo à VM pela rede local confirmou que o IIS estava
servindo a página por HTTP.

**Sobre a página:** trata-se de uma demonstração estática em HTML, CSS e
JavaScript. Os dados exibidos são demonstrativos e as interações
acontecem no navegador. Não há persistência em banco de dados nem
comunicação com uma API nesta versão hospedada.

### 3.3 Permissões NTFS

As permissões da pasta `C:\inetpub\wwwroot` foram verificadas pela
janela **Advanced Security Settings**.

  Grupo / usuário    Permissão observada
  ------------------ ---------------------
  `SYSTEM`           Full Control
  `Administrators`   Full Control
  `Users`            Read & Execute
  `IIS_IUSRS`        Read & Execute

Essa configuração permite que o processo do IIS leia e execute o
conteúdo publicado, sem conceder a ele controle total sobre a pasta.

### 3.4 Monitoramento de recursos

Foi criado no Performance Monitor (`perfmon`) o conjunto de coleta:

``` text
BRIDGE_Monitoramento
```

Contadores adicionados:

  --------------------------------------------------------------------------------------
  Recurso                 Contador                               Finalidade
  ----------------------- -------------------------------------- -----------------------
  CPU                     `Processor(_Total)\% Processor Time`   Acompanhar o percentual
                                                                 de utilização total da
                                                                 CPU

  Memória                 `Memory\Available MBytes`              Acompanhar a memória
                                                                 física disponível

  Disco                   `LogicalDisk(_Total)\% Free Space`     Acompanhar o percentual
                                                                 de espaço livre
  --------------------------------------------------------------------------------------

O conjunto foi executado para coletar dados e os resultados foram
consultados em **Reports → User Defined**.

### 3.5 Alerta de desempenho

Foi criado o conjunto de alerta:

``` text
BRIDGE_Alerta_CPU
```

Configuração do contador:

-   Objeto: `Processor`
-   Contador: `% Processor Time`
-   Instância: `Total`
-   Condição: `Above`
-   Limite: `80`

O alerta foi iniciado no Performance Monitor. O limite representa uma
condição de uso elevado da CPU; não significa que uma ocorrência acima
de 80% tenha sido necessariamente observada durante a execução do
laboratório.

### 3.6 Recuperação automática do serviço IIS

Na ferramenta `services.msc`, foram configuradas as opções de
recuperação do serviço **World Wide Web Publishing Service (W3SVC)**:

  Opção                    Configuração
  ------------------------ ---------------------
  First failure            Restart the Service
  Second failure           Restart the Service
  Subsequent failures      Restart the Service
  Restart service after    1 minute
  Reset fail count after   1 day

A configuração orienta o Windows a tentar reiniciar o serviço quando uma
falha é detectada.

> A configuração documentada refere-se ao serviço W3SVC. A recuperação
> específica do Application Pool e um teste controlado de falha devem
> ser registrados como etapas adicionais caso sejam realizados.

### 3.7 Logs do IIS

Os logs de acesso foram localizados no diretório:

``` text
C:\inetpub\logs\LogFiles
```

Os arquivos de log do IIS permitem acompanhar requisições recebidas pelo
site, incluindo acessos HTTP.

## 4. Como acessar a página

Com a máquina virtual ligada e o IIS em execução, acesse:

-   Na própria VM: `http://localhost`
-   Em outro computador da mesma rede local: `http://192.168.18.250`

O acesso por IP depende de a VM estar conectada à rede e de a regra de
firewall HTTP permitir conexões de entrada.

## 5. Evidências

As evidências técnicas devem ser incluídas neste repositório,
preferencialmente em uma pasta `evidencias/`, com nomes que identifiquem
cada etapa.

Sugestão de organização:

``` text
BRIDGE/
├── README.md
├── index.html
└── evidencias/
    ├── 01-ip-hostname.png
    ├── 02-dns-zona-direta.png
    ├── 03-dns-zona-reversa.png
    ├── 04-active-directory.png
    ├── 05-iis-site.png
    ├── 06-pagina-localhost.png
    ├── 07-pagina-acesso-rede.png
    ├── 08-permissoes-ntfs.png
    ├── 09-monitoramento-contadores.png
    ├── 10-relatorio-monitoramento.png
    ├── 11-alerta-cpu.png
    ├── 12-recuperacao-w3svc.png
    └── 13-logs-iis.png
```

Ajuste os nomes conforme os arquivos realmente adicionados. Não inclua
evidências de configurações que não foram executadas.

## 6. Limitações e próximos passos

A infraestrutura demonstrada atende à configuração de rede e identidade,
à publicação de uma página estática no IIS, às permissões NTFS
verificadas, à coleta de contadores, ao alerta de CPU configurado, à
recuperação do serviço W3SVC e à consulta dos logs do IIS.

Os seguintes itens não estão demonstrados como concluídos nesta versão:

-   Integração da página hospedada com o backend FastAPI, banco
    PostgreSQL e aplicativo mobile.
-   Persistência real dos dados enviados pela interface.
-   Configuração e validação de recuperação específica do Application
    Pool do IIS.
-   Teste controlado de falha e comprovação da recuperação após a falha.
-   Implementação de File Server, GPO, RDS e DFS.

Esses itens devem ser implementados e documentados somente se fizerem
parte do escopo final exigido pela equipe ou pelo professor.

## 7. Conclusão

O laboratório BRIDGE demonstra a preparação de um ambiente Windows
Server virtualizado, com endereço fixo, DNS, Active Directory e
hospedagem web pelo IIS. Também apresenta configurações de permissões
NTFS, monitoramento de recursos, alerta de desempenho, recuperação
automática do serviço de publicação web e consulta dos logs de acesso.

As capturas de tela e demais evidências deste repositório registram as
etapas efetivamente executadas no ambiente de laboratório.

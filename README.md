# Diagrama Defense in Depth Cantu

![](https://raw.githubusercontent.com/mr-zorbot/cantu-ps/refs/heads/main/cantu.drawio.png)

## Explicação dos controles propostos

### Controles Físicos
1. **Geradores e UPS:** Garantem a continuidade de energia em caso de quedas, evitando indisponibilidade dos ativos, corrupção de dados e danificação do _hardware_. 
2. **Heating, Ventilation, and Air Conditioning (HVAC):** Importantes para manter a temperatura e umidade adequadas para funcionamento dos equipamentos, evitando superaquecimento, incêndios, estática e curtos. 
3. **Salas seguras/monitoradas, cabeamento seguro, HDDs em cofres e racks com fechadura:** Os servidores e equipamentos de rede da organização e seus discos devem ficar em cofres/racks trancados em sala separada do escritório. A sala deve possuir controles preventivos de acesso - como scanner de retina ou fechaduras RFID - para evitar acesso físico não autorizado, cabos devem ser colocados em dutos para evitar interceptação/sabotagem, a sala e seu perímetro devem ser monitorados por cameras, sensores de movimento, etc. Além disso, é válido relebrar que controles de acesso devem ser aplicados em toda a organização, não só na infraestrutura de TI.
4. **Backup Off-site:** Garante a resiliência dos dados em incidentes físicos - como incêndios - e lógicos - como _ransomware_. Pode ser implementado utilizando infraestrutura Microsoft, como o Microsoft Entra ID - para o AD local - e o OneDrive/Sharepoint para o File Server.
5. **Descarte e reutilização adequados de equipamentos, mídias e documentos:** Fundamental para evitar vazamentos de dados, porém, é muito negligenciado em muitas organizações. Documentos devem ser triturados antes do descarte, mídias de armazenamento e computadores devem passar por processo de sanitização de dados caso possam ser reutilizados.
6. **Tela/Mesa limpa e trabalho remoto em locais seguros:** Evita a exposição de dados sensíveis em locais de uso comum. Em caso de trabalho remoto, deve haver restrição a locais específicos para reforçar o controle.
7. **Manutenção adequada dos equipamentos:** Fundamental para evitar incidentes previsíveis, causados por negligência.
8. **Segurança Física de ambientes off-premise:** Apesar de ser responsabilidade do provedor, cabe ao contratante exigir controles contratuais, selecionar provedores certificados e em compliance com a legislação/normas e, quando possível, auditar o contratado.

### Controles de Rede

1. **Segmentação em VLANs:** Fundamental para reduzir a superfície de ataque e limitar movimentação lateral, além de facilitar a gestão. No cenário descrito, a rede deve ser segmentada para separar servidores, impressoras e funcionários (idealmente, segmentar por departamento também).
2. **Filtragem de pacotes:** Fundamental criar regras para bloquear tráfego _inbound_ e _outbound_ indesejados e potencialmente maliciosos - como bit-torrent e tráfego relacionado à C&C - e limitar acesso a serviços críticos.
3. **Sistema de detecção e prevenção de intrusos:** Monitora a rede para identificar e bloquear ataques conhecidos e comportamentos suspeitos. Além de gerar alertas para investigação.
4. **VPN:** Considerando o cenário de trabalho híbrido, é fundamental que os funcionários utilizem VPN ao trabalhar de locais remotos para garantir a confidencialidade dos dados da organização, a identidade do usuário e o acesso aos recursos de rede do escritório. Além disso, o acesso à interface de administração do ambiente Cloud também deve ser realizada via VPN pelos mesmos motivos. 
5. **Utilização obrigatória de protocolos criptografados:** Fundamental para garantir a confidencialidade dos dados em trânsito. Dentre eles, é importante destacar a utilização de LDAPS, SMB over QUIC, IPPS, HTTPS e SSH.
6. **Virtual Private Cloud:** Utilizar VPCs para separar cada elemento do ambiente Cloud em uma sub-rede diferente, expondo apenas o Frontend para a Internet, e controlar o tráfego através de NACLs.
7. **Load Balancer:** Com base no cenário, onde e-commerce em Cloud representa 80% da receita da empresa, é fundamental utilizar um balanceador de carga para garantir a disponibilidade do serviço e mitigar ataques DDoS.
8. **DNS Sinkhole:** Levando em consideranção - principalmente - os riscos de ataques de _phishing_, a organização pode configurar um servidor DNS - como Pi-hole - para redirecionar consultas à domínios maliciosos para IPs inválidos. A lista de domínios bloqueados pode ser alimentada com bases de dados públicas - como Phish Tank - e IOCs coletados pela própria organização.
9. **SPF, DKIM e DMARC:** Visando garantir a autenticidade dos e-mails oriundos do domínio da organização e evitar _spoofing_, é fundamental a configuração correta dos mecanismos de autenticação de e-mail, indicando os servidores autorizados a enviar mensagens em nome da organização (SPF), assinando digitalmente as mensagens (DKIM) e orientando o bloqueio de e-mails que não passaram nessas verificações (DMARC).

### Controles de Endpoint

1. **Gestão de atualizações do SO/Firmware:** Garante que os dispositivos não vão estar vulneráveis à falhas conhecidas.
2. **Hardening do Sistema Operacional:** Desabilitar usuários locais, desinstalar software e dasabilitar serviços desnecessários para reduzir a superfície de ataque.
3. **Restrição de uso de dispositivos removíveis:** O uso arbitrário de mídias removíveis é um dos principais vetores de propagação de _malware_ e vazamentos de dados. Logo, é fundamental implementar controles para bloquear dispositivos desconhecidos, isso pode ser feito utilizando ferramentas como o USBGuard, pelo EDR ou via GPO pelo AD.
4. **Microsoft Defender for Endpoint e Microsoft Defender for Cloud:** Assumindo que o ambiente da organização é Microsoft, o MDE é a melhor solução de EDR para prevenir, detectar e bloquear ataques a nível de _endpoint_. Adicionalmente, para o ambiente Cloud, o MDC amplia essa proteção ao fornecer recomendações de conformidade, detecção de configurações incorretas e monitoramento do _workload_ em tempo real. Além disso, essas ferramentas se integram facilmente ao SIEM Sentinel abordado posteriormente.
5. **Hardening de Contêineres:** Assumindo que a aplicação do e-commerce e suas dependências provavelmente são executadas em Docker, é fundamental garantir que esses contêineres possuam o mínimo de privilégios possível e sejam instâncias de imagens oficiais.

### Controles de Aplicação 

1. **MFA obrigatório para todos os usuários:** Utilizar ao menos mais um fator de autenticação além da senha para reduzir o risco de invasão por credencias comprometidas.
2. **Política de senhas fortes e lockout:** Forçar requisitos de senha com base, preferencialmente, nas recomendações do NIST e bloquear o usuário após sucessivas falhas de autenticação (i.e. possível _brute force_).
3. **Role-Based Access Control:** Agregar os usuários com base nas suas funções e fornecer apenas os privilégios mínimos para desempenhar a função.
4. **ACLs baseadas no RBAC:** Utilizar as _roles_ definidas anteriormente para controlar o acesso a arquivos, pastas, redes e dispositivos.
5. **Restrições de login via GPO e Políticas de Acesso Condicional:** Limitar o acesso, por exemplo, com base em horário de trabalho, dispositivo utilizado, geolocalização, reputação do IP, etc.
6. **Autenticação via AD nos hosts:** Desabilitar os usuários locais e utilizar apenas os usuários de rede.
7. **Microsoft Defender for Identity:** Solução da Microsoft para segurança de identidade, deve ser instalado em todos os controladores de domínio.
8. **Microsoft Defender for Cloud Apps:** CASB da Microsoft, oferece funcionalidades de DLP, Web Filter, restrição de Shadow IT e monitoria de SaaS.
9. **Microsoft Defender for Office 365:** Protege diretamente o e-mail corporativo contra phishing, malware, anexos maliciosos e links perigosos, além de atuar também na proteção do Sharepoint e outras ferramentas de colaboração.
10. **WAF (Web Application Firewall):** Utilizado para filtrar o tráfego e proteger a aplicação do e-commerce contra tentativas de exploração de vulnerabilidades web comuns (SQLi,XSS,Path Traversal, etc.).
11. **Validação de Input:** Evita que entradas de usuário injetem código malicioso.
12. **SAST/DAST:** Análise estática e dinâmica de código para identificar falhas antes de colocar a aplicação em produção.
13. **Segregação de ambientes:** Separar ambientes de desenvolvimento, teste e produção.
14. **Bibliotecas seguras e atualizadas:** Corrigir vulnerabilidades conhecidas em dependências externas, mitigando ataques _supply chain_.
15. **RBAC no Banco de Dados:** Restringir o acesso apenas a _queries_ e operações necessárias.
16. **Versionamento via Git:** Para controle de versionamento e rastreabilidade do código.

### Controles de Dados

1. **Criptografia em repouso e em trânsito:** Para garantir a confidencialidade dos dados armazenados e trafegando na rede.
2. **Hashing e salting no BD para informações sensíveis:** Para proteger credenciais contra ataques de força bruta e _rainbow table_.
3. **ReplicaSet:** Assegura alta disponibilidade e redundância de bancos de dados (já mencionado no contexto).  
4. **Retenção e descarte de dados conforme legislação/normas:** Para reduzir riscos legais e de vazamento, mantendo apenas os dados estritamente necessários dentro dos prazos exigidos por lei.  
5. **RAID:** Oferece redundância, garantindo disponibilidade mesmo na falha de discos.  
6. **FDE/BitLocker em todos os dispositivos:** Para proteger os dados em caso de roubo ou perda de notebooks e estações de trabalho.  
7. **Snapshots das VMs:** Para permitir recuperação rápida de máquinas virtuais em falhas. Devem ser realizadas com frequência e "_golden images_" devem ser criadas/salvas.  
8. **System state backup:** Específicamente para garantir a integridade do AD, visto que restaurar _snapshots_ de DCs provavelmente causará inconsistências. 
9. **Uso obrigatório de S/MIME ou criptografia via Purview:** Para assegurar a confidencialidade e integridade de comunicações por e-mail.  

### Monitoramento e Resposta

1. **Envio de logs para o SIEM (Sentinel) via Syslog/Sysmon/Auditd:** Para centralizar eventos de diferentes fontes em um único ponto de análise.
2. **Microsoft Defender XDR:** Integração das ferramentas citadas anteriormente.
3. **Revisão periódica de acessos:** Para garantir que usuários mantenham apenas os privilégios necessários às suas funções.
4. **Regras de detecção para exploração de vulnerabilidades/ataques específicos, relacionadas às tecnologias utilizadas pela organização:** Por exemplo, regras para identificar ataques _golden ticket_, _kerberoasting_ e replicação no AD; download de arquivos em massa, renomeação em massa (indicativo de _ransomware_) no FS; etc.
5. **DFIR (Digital Forensics and Incident Response):** Consiste em aplicar o método científico na coleta, exame, análise e report de incidentes de segurança. É importante para obter um processo de resposta a incidentes mais consistente e maduro.

### Políticas, Treinamento e Conscientização

1. **Política de backup:** Para estabelecer frequência, retenção e locais de armazenamento.  
2. **Plano de Resposta a Incidentes:** Para guiar a detecção, análise, contenção e recuperação de incidentes de segurança.  
3. **Plano de Recuperação de Desastres:** Para garantir a restauração de sistemas e infraestrutura após falhas graves, assegurando disponibilidade mínima e retomada dos serviços.
4. **Plano de Continuidade de Negócios:** Para manter operações críticas ativas durante e após incidentes, assegurando resiliência organizacional.
5. **Plano de Comunicação:** Para definir canais e responsáveis pela comunicação durante crises, evitando desinformação.
6. **Security Awareness Training:** Para conscientizar usuários sobre boas práticas de segurança, reduzindo riscos relacionados a phishing, engenharia social e uso inseguro de sistemas.  
7. **Simulação de Resposta a Incidentes:** Para testar a eficácia dos planos de resposta e treinar equipes em cenários realistas de ataque.
8. **Simulação de Phishing:** Devem ser realizadas periodicamente para avaliar a resiliência dos usuários contra ataques de engenharia social e reforçar a conscientização.
9. **Política de report correto de phishing:** Para evitar, por exemplo, que os usuários encaminhem o e-mail ao invés do EML (compromentendo o cabeçalho).
10. **NDA (Non-Disclosure Agreement):** Para proteger informações confidenciais da organização contra vazamentos intencionais ou acidentais por funcionários e terceiros.  
11. **Treinamento para desenvolvedores:** Para capacitar desenvolvedores em práticas de desenvolvimento seguro, prevenindo vulnerabilidades de software.  
12. **Política de desenvolvimento seguro:** Para padronizar requisitos de segurança em todo o ciclo de vida do software.
13. **Política de uso seguro de IAs:** Para regulamentar o uso de inteligência artificial, prevenindo vazamento de dados e vulnerabilidades causadas por código vulnerável gerado por IA.  
14. **Cultura Secured DevOps:** Mover a segurança para o início do ciclo de desenvolvimento de software.
15. **GMUD via ITSM:** Para formalizar e controlar mudanças em sistemas e infraestrutura, além de permitir rastreabilidade de alterações.
16. **Gestão de Riscos e Vulnerabilidades:** Para identificar, classificar e mitigar falhas de segurança, assegurando correções dentro de prazos adequados. 
17. **Inventário de ativos:** Mapear periodicamente todos os recursos de TI, garantindo visibilidade e base para aplicar controles de segurança.

### Recomendações de possíveis melhorias

1. **Utilização de um NGFW:** Considerar a aquisição de um Firewall de nova geração que integre IDS/IPS, VPN para acesso remoto com suporte a autenticação via AD/AAD, inspeção profunda de pacotes e threat intelligence.
2. **Adição de um servidor RADIUS:** Considerar a implementação de um servidor RADIUS (NPS de preferência) para centralizar o controle de acesso de rede, integração com o AD para definição de NACLs para VLANs específicas e garantir que apenas funcionários tenham acesso a rede corporativa (via VPN/WPA3-enterprise).
3. **Implementação de cofre de senha on-premise:** Considerar a implementação de um cofre de senhas _self hosted_ - como o Vaultwarden - para evitar repetição de senhas e armazenamento inseguro. 
4. **Zabbix para monitoramento de ativos:** Considerar a implementação de um servidor Zabbix para monitorar o desempenho, disponibilidade e integridade dos ativos em tempo real.

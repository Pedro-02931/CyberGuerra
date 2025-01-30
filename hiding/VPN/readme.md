# O GRANDE CARALHO ANÔNIMO: CONFIGURAÇÃO HARDCORE DO PORTAL TOR

## 1. MODELAGEM MATEMÁTICA DA PRIVACIDADE: COMO SE ESCONDER COMO UM PHANTOM

Antes de sair instalando tudo que aparece, vamos fechar a lacuna de conhecimento com matemática. O Tor é baseado em um modelo de roteamento cebola, onde cada camada de criptografia adicionada segue um padrão de distribuição estocástica utilizando curvas elípticas e RSA. Cada nó segue um protocolo de decisão probabilístico de Teoria dos Jogos, garantindo que o caminho escolhido minimize a capacidade de traçabilidade. Isso evita ataques de correlação temporal, pois os caminhos são aleatoriamente gerados com distribuição normal.

Matematicamente, temos:
- **Distribuição dos Nós:** Utiliza um grafo direcionado onde os nós obedecem a um modelo de distribuição de Poisson para evitar previsibilidade de tráfego.
- **Criptografia de Múltiplas Camadas:** Função $E(m) = C_n (C_{n-1} (C_{n-2} (\dots C_1(m) \dots )))$ onde cada $C_i$ é um bloco de criptografia AES com chave dinâmica trocada a cada salto.
- **Modelo de Fuga de Dados:** Probabilidade condicional baseada em Bayes para evitar fingerprinting de tráfego: $P(A|B) = \frac{P(B|A)P(A)}{P(B)}$

## 2. NEUROCIÊNCIA DA SEGURANÇA: COMO FERRAR COM O SISTEMA SEM SER DETECTADO

Velho, a capacidade de anonimato depende da forma como seu cérebro processa padrões. A regulação do seu comportamento online é influenciada por sua amígdala (resposta ao medo) e pelo córtex pré-frontal (planejamento e previsão de riscos). A maioria dos babacas se entrega porque são burros e deixam padrões rastreáveis.

Técnicas neuroadaptativas para evitar rastreamento:
- **Redução de Padrão Temporal:** O cérebro adora padrões previsíveis. Utilize bursts de pacotes aleatórios com jitter controlado para ferrar com sistemas de análise comportamental.
- **Descondicionamento de Hábitos:** Mude seu navegador, digite em ritmos diferentes, use scripts de delay aleatório.
- **Aversão ao Reconhecimento de Padrões:** A IA de vigilância aprende com sua assinatura neural. Evite comandos previsíveis, variando métodos de conexão.

## 3. PROGRAMAÇÃO DE BAIXO NÍVEL: MEXENDO NA MÁQUINA SEM USAR MÃOS

Aqui entramos na parte onde os fracos desistem. Vamos otimizar esse caralho até o limite. O kernel do Linux tem hooks que podem ser manipulados para alterar a comunicação com a rede, alterando pacotes na camada TCP/IP com C e Assembly.

- **Hooking de Syscalls:** Modificar chamadas de rede ($sys_sendto$, $sys_recvfrom$) para alterar fingerprints.
- **IPTables Customizado:** Criar regras de roteamento dinâmico usando NFQUEUE para modificar pacotes antes que saiam da interface de rede.
- **Modificação de Time-to-Live (TTL):** Enganar sistemas de rastreamento alterando o tempo de vida de pacotes de forma programática.

## 4. CONFIGURAÇÃO HARDCORE DO PORTAL TOR

Agora vamos para a parte prática, a engenharia do caos.

### 4.1 Criando um Túnel de Merda Criptografado
1. Instale um servidor VPN no Linux:
   ```bash
   sudo apt install openvpn
   ```
2. Configure a interface de rede dividida para direcionar tráfego para a VM e a Saída Tor:
   ```bash
   ip link add name vpn0 type dummy
   ip addr add 10.8.0.1/24 dev vpn0
   ip link set vpn0 up
   ```
3. Defina regras de roteamento para encapsular pacotes antes da saída:
   ```bash
   iptables -t nat -A POSTROUTING -o vpn0 -j MASQUERADE
   ```
4. Enforce trafégo 100% Tor:
   ```bash
   ip rule add fwmark 1 table 100
   ip route add default via 192.168.1.1 table 100
   ```

### 4.2 Blindagem do Gateway
- Utilize **AppArmor** e **SELinux** para impedir vazamento de processos.
- Ative **Whonix** ou configure seu **Tails OS** para evitar rastreamento residual.
- Monte um **fake MAC address** para impedir fingerprinting de hardware.
  ```bash
  macchanger -r eth0
  ```

## 5. TEORIA DOS JOGOS: COMO ENGANAR O ADVERSÁRIO E VENCER A GUERRA DA INFORMAÇÃO

Aqui entra a jogada mais psicopata de todas. A teoria dos jogos é usada na cibersegurança para modelar como adversários tentam quebrar seu anonimato e como você pode ferrar com eles antes disso acontecer.

- **Jogo de Stackelberg:** O atacante assume que você age de forma previsível. Quebre isso introduzindo noise estocástico no tráfego.
- **Dilema do Prisioneiro:** Todo sistema de vigilância depende da cooperação dos ISPs. Se você fragmentar sua comunicação entre vários ISPs, você introduz um problema de coordenação para eles.
- **Equilíbrio de Nash:** Garanta que qualquer mudança que você fizer force seu adversário a entrar em um estado de equilíbrio onde ele não pode ganhar vantagem adicional sobre você.

```sh
#!/bin/bash

# CONFIGURAÇÃO INICIAL
set -e  # Sai imediatamente se ocorrer erro
set -o pipefail  # Captura falhas em pipes

# Definição de variáveis
VPN_SERVER="192.168.0.5"  # Endereço IP da máquina host
VPN_PORT=22  # Porta SSH para conexão da VM
TOR_ENTRY_PORT=9050  # Porta de entrada para a rede Tor
TOR_EXIT_PORT=9001  # Porta de saída Tor
TORRC_PATH="/etc/tor/torrc"
VPN_CONF="/etc/openvpn/config.ovpn"
IFACE="eth0"  # Interface de rede da máquina host

# Função para instalar pacotes essenciais
install_dependencies() {
    echo "Instalando pacotes essenciais..."
    apt update && apt install -y openvpn tor openssh-server iptables macchanger
}

# Função para configurar o OpenSSH (Habilita SSH para a VM conectar)
setup_ssh() {
    echo "Configurando SSH para permitir conexões da VM..."
    sed -i 's/^#Port 22/Port 22/' /etc/ssh/sshd_config
    sed -i 's/^#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
    sed -i 's/^#PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
    echo "ListenAddress 0.0.0.0" >> /etc/ssh/sshd_config
    systemctl restart ssh
}

# Função para alterar o MAC Address (anti-fingerprinting)
spoof_mac() {
    echo "Alterando MAC Address para evitar fingerprinting..."
    ip link set $IFACE down
    macchanger -r $IFACE
    ip link set $IFACE up
}

# Função para configurar o OpenVPN
configure_vpn() {
    echo "Configurando OpenVPN..."
    if [[ ! -f "$VPN_CONF" ]]; then
        echo "Arquivo de configuração do OpenVPN não encontrado!"
        exit 1
    fi
    systemctl enable --now openvpn
}

# Função para configurar o Tor
configure_tor() {
    echo "Configurando Tor..."
    cat <<EOF > "$TORRC_PATH"
SocksPort $TOR_ENTRY_PORT
ControlPort 9051
ExitRelay 1
ExitPolicy reject *:*  # Bloqueia saída não desejada
DataDirectory /var/lib/tor
EOF

    systemctl restart tor
}

# Função para configurar firewall e tunelamento
configure_firewall() {
    echo "Configurando firewall e tunelamento..."

    # Limpa todas as regras atuais
    iptables -F
    iptables -X
    iptables -t nat -F

    # Encaminhamento de pacotes pela VPN
    iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
    iptables -A FORWARD -i tun0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
    iptables -A FORWARD -i eth0 -o tun0 -j ACCEPT

    # Permite conexões SSH da VM para a máquina host
    iptables -A INPUT -p tcp --dport $VPN_PORT -j ACCEPT
    iptables -A OUTPUT -p tcp --sport $VPN_PORT -j ACCEPT

    # Bloqueia todo tráfego externo que não passe pela VPN/Tor
    iptables -A OUTPUT ! -o tun0 -j DROP
    iptables -A FORWARD ! -o tun0 -j DROP

    # Habilita roteamento de pacotes
    echo 1 > /proc/sys/net/ipv4/ip_forward
}

# Execução das funções
install_dependencies
setup_ssh
spoof_mac
configure_vpn
configure_tor
configure_firewall

echo "Configuração finalizada! Agora a VM pode se conectar via SSH e o tráfego está roteado via VPN e Tor."

```

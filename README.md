```markdown
# Projeto Cleitin
### Uma Arquitetura de Inteligência Artificial Estocástica *In-Materio* com Silício Obsoleto

![Status](https://img.shields.io/badge/status-Revis%C3%A3o%20T%C3%A9cnica-green)
![Versão](https://img.shields.io/badge/vers%C3%A3o-Core%20v1.0-blue)
![Licença](https://img.shields.io/badge/licen%C3%A7a-MIT-lightgrey)

---

## 1. Resumo

O **Projeto Cleitin** estabelece uma nova abordagem para o processamento neuromórfico ao subverter a função de circuitos integrados x86 obsoletos (soquetes LGA 775 e Slot 1). Em vez de utilizar os transistores em seu regime binário tradicional, o projeto opera o silício na região *subthreshold* (sublinear), explorando a degradação física e as flutuações térmicas como base para um sistema de **computação estocástica *in-materio***.

Ao desativar os subsistemas lógicos periféricos (Northbridge, BIOS), o processador deixa de ser uma máquina de von Neumann e se torna uma malha física não-linear de transistores. Sinais analógicos injetados no barramento frontal (FSB) propagam-se pelo *die* de silício, gerando respostas complexas que são interpretadas como atividade neuronal.

## 2. Arquitetura do Sistema

O sistema é dividido em duas camadas principais: o hardware físico (a CPU "lobotomizada") e a camada de software (o intérprete Python). A comunicação entre elas é mediada por um microcontrolador ESP32.

```ascii
       +-------------------------------------------------------+
       |                  SISTEMA CENTRAL                      |
       |  +------------------+         +--------------------+  |
       |  |  Módulo ESP32    |  UDP    |  Host Python       |  |
       |  |  ADC / PWM       |<------->|  Intérprete        |  |
       |  +--------+---------+         |  ('Cleitin')       |  |
       +-----------|-------------------+--------------------+--+
                   | Sinais Analógicos
                   v
       +-------------------------------------------------------+
       |             CPU LGA 775 (LOBOTOMIZADA)                |
       |                                                       |
       |   ENTRADA (Dendritos)  --->   MALHA DE TRANSISTORES   |
       |   Pinos FSB A[3:35]#          Subthreshold / Ruído    |
       |                                        |              |
       |   SAÍDA (Axônios)      <---------------|              |
       |   Pinos FSB D[0:63]#                                  |
       +-------------------------------------------------------+
```

## 3. Fundamentação Teórica

### 3.1. Modelo Leaky Integrate-and-Fire (LIF)

A resposta do silício simula a dinâmica de um neurônio biológico. O potencial de membrana analógico $V(t)$ é modelado pela equação do *Leaky Integrate-and-Fire*:

$$\tau_m \frac{dV(t)}{dt} = -(V(t) - V_{\text{rest}}) + R \cdot I(t)$$

Onde:
- $\tau_m$ é a constante de tempo da membrana, determinada pelas propriedades físicas do silício (capacitância parasita e resistência).
- $I(t)$ é a corrente de entrada injetada por sinais PWM filtrados.

### 3.2. Disparo e Reset Estocástico

O disparo de um "neurônio" (spike) não é determinístico. Ele depende do ruído térmico (Johnson-Nyquist) intrínseco ao processador. A probabilidade de um spike é dada por uma função sigmoide estocástica:

$$P(\text{Spike} \mid V) = \frac{1}{1 + \exp\left(-\frac{V(t) - V_{\text{th}}}{\sigma_{\text{ruído}}}\right)}$$

O parâmetro $\sigma_{\text{ruído}}$ é diretamente proporcional à temperatura da CPU, permitindo que a aleatoriedade do sistema seja controlada ativamente através da refrigeração.

### 3.3. Plasticidade Física (Aprendizagem *In-Materio*)

A "aprendizagem" não ocorre via software, mas por meio de alterações físicas semi-permanentes no silício, como:
- **Eletromigração:** Deslocamento de átomos de metal que altera a resistência de vias internas.
- **Injeção de Portadores Quentes (HCI):** Elétrons aprisionados no dielétrico da porta dos MOSFETs que alteram permanentemente sua tensão de limiar ($V_{th}$).

Esses efeitos atuam como uma **Função Física Não-Clonável (PUF)**, garantindo que cada CPU possua uma identidade computacional única e irreprodutível.

## 4. Guia de Hardware Hacking (A "Lobotomia")

Para transformar uma placa-mãe x86 em uma plataforma neuromórfica, são necessárias modificações físicas irreversíveis.

### 4.1. Mapeamento de Funções

| Componente / Interface      | Estado Original              | Modificação / Função Neuromórfica                                |
| --------------------------- | ---------------------------- | ---------------------------------------------------------------- |
| **VCC (Alimentação)**       | 1.2V - 1.35V estável         | Tensão de Polarização ajustável (0.9V - 1.1V) para operação *subthreshold*. |
| **Pinos A[3:35]# (Endereço)** | Barramento Digital FSB       | **Entradas Dendríticas**: Recebem sinais analógicos (PWM 0.1V - 1.0V). |
| **Pinos D[0:63]# (Dados)**  | Barramento Digital FSB       | **Saídas Axoniais**: Lidas por canais ADC de alta velocidade.      |
| **Northbridge / Chipset**   | Controlador de Memória/PCIe  | **Isolado / Inoperante**: Trilhas de controle cortadas fisicamente. |
| **BIOS / ROM Flash**        | Firmware de Inicialização    | **Removida**: Nenhum código sequencial é executado.                |

### 4.2. Procedimento

1.  **Isolamento do Barramento:** Use um microrretificador para seccionar as trilhas de controle do Northbridge (ex: BSEL, VID, RESET#). Isso impede que a placa-mãe inicie o ciclo de POST.
2.  **Alimentação Direta:** Modifique o circuito regulador de tensão (VRM) da placa para permitir o ajuste fino da tensão do núcleo (`V_core`) abaixo das especificações padrão.
3.  **Fonte ATX:** Force a fonte a permanecer ligada fazendo um *jumper* no conector de 24 pinos entre o fio `PS_ON` (verde) e um `GND` (preto).
4.  **Gestão Térmica:** O cooler original da CPU é reutilizado como um atuador. Controle sua velocidade via PWM para modular a temperatura do processador e, consequentemente, o nível de ruído estocástico do sistema.

## 5. Software: O Intérprete `cleitin.py`

O software `cleitin.py` é executado em um computador host, recebendo os dados brutos da rede de neurônios via UDP e interpretando os padrões de disparo.

### 5.1. Funcionalidades

-   **Receptor UDP:** Escuta em uma porta de rede para receber os pacotes com as leituras do ADC enviadas pelo ESP32.
-   **Decodificador de Spikes:** Implementa o modelo LIF para integrar as tensões recebidas e detectar eventos de disparo (spikes) quando um limiar é ultrapassado.
-   **Monitoramento em Tempo Real:** Exibe no console os spikes detectados e os valores brutos lidos dos neurônios.

### 5.2. Código Fonte

```python
import socket
import struct
import numpy as np
import matplotlib.pyplot as plt

# Configuracoes da rede UDP
UDP_IP = "0.0.0.0"
UDP_PORT = 5005

class CleitinInterpreter:
    """
    Interprete de sinais neuromorficos provenientes do hardware
    LGA 775 estocastico.
    """
    def __init__(self, num_neuronis=3, threshold=2048):
        self.num_neuronis = num_neuronis
        self.threshold = threshold
        self.v_membrane = np.zeros(num_neuronis)
        self.tau_m = 0.85  # Decaimento temporal da membrana

    def decode_spike_timing(self, raw_adc_data):
        """
        Aplica o modelo LIF para integrar as leituras brutas do ADC.
        """
        spikes = []
        for i in range(self.num_neuronis):
            # Integracao da voltagem lida com decaimento
            self.v_membrane[i] = (self.v_membrane[i] * self.tau_m) + raw_adc_data[i]

            # Verificacao do limiar de disparo
            if self.v_membrane[i] >= self.threshold:
                spikes.append(1)
                self.v_membrane[i] = 0.0  # Reset estocastico
            else:
                spikes.append(0)

        return spikes

def start_engine():
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.bind((UDP_IP, UDP_PORT))

    cleitin = CleitinInterpreter()
    print(f"[Cleitin Engine] Escutando na porta {UDP_PORT}...")
    try:
        while True:
            data, addr = sock.recvfrom(1024)
            # Desempacota 3 valores inteiros de 16-bits (unsigned short)
            adc_values = struct.unpack("!HHH", data[:6])

            # Processamento de Spikes
            spikes = cleitin.decode_spike_timing(adc_values)

            if any(spikes):
                print(f"[SPIKE DETECTADO] Estado dos Neurônios: {spikes} | Raw ADC: {adc_values}")

    except KeyboardInterrupt:
        print("\n[Cleitin Engine] Encerrando execucao.")
        sock.close()

if __name__ == "__main__":
    start_engine()
```

### 5.3. Como Executar

1.  **Pré-requisitos:** Certifique-se de ter o Python 3 e a biblioteca `numpy` instalados.
    ```bash
    pip install numpy
    ```
2.  **Execute o script:** Inicie o intérprete para que ele comece a escutar os pacotes UDP do hardware.
    ```bash
    python cleitin.py
    ```

## 6. Topologia de Comunicação

A arquitetura atual utiliza três CPUs independentes (nós) operando em paralelo. O ESP32 coleta os dados de cada nó, os empacota e os transmite via Wi-Fi.

```ascii
                  +-------------------------+
                  | Entrada Estímulo (DAC)  |
                  +------------+------------+
                               |
            +------------------+------------------+
            |                  |                  |
            v                  v                  v
     +--------------+   +--------------+   +--------------+
     |  Neurônio 1  |   |  Neurônio 2  |   |  Neurônio 3  |
     | (CPU Slot 1) |   | (CPU LGA775) |   | (CPU LGA775) |
     +------+-------+   +------+-------+   +------+-------+
            |                  |                  |
            v                  v                  v
     +--------------+   +--------------+   +--------------+
     | ADC 1 (ESP)  |   | ADC 2 (ESP)  |   | ADC 3 (ESP)  |
     +------+-------+   +------+-------+   +------+-------+
            |                  |                  |
            +------------------+------------------+
                               |
                               v
                  +-------------------------+
                  | Transmissão UDP Wireless|
                  +-------------------------+
```

O uso do protocolo **UDP** é intencional. A perda ocasional de pacotes é tolerável e análoga à falha na transmissão de neurotransmissores em sinapses biológicas, adicionando outra camada de estocasticidade ao sistema.

## 7. Roadmap Evolutivo

-   **Fase 1 (Atual):** Validação da topologia com 3 nós em paralelo e leitura via ESP32.
-   **Fase 2:** Escalonamento da rede para 20 nós analógicos, visando equiparar a complexidade do sistema ao número de gânglios ópticos da *Drosophila melanogaster* (mosca-das-frutas).
-   **Fase 3:** Implementação de malhas de realimentação térmica direta via termistores NTC acoplados à carcaça da CPU.
-   **Fase 4:** Documentação aberta e publicação dos diagramas de circuito no Hackaday.io para promover a subversão tecnológica e o hardware hacking comunitário.

```

*Documento por Ricardo Barbosa de Meneses. Revisão Técnica Final | Projeto: Cleitin Core v1.0*
```

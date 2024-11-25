# MiniDataCloud

### **Projeto Insight Energy: MiniDataCloud**

---

Este documento consolida e versão o projeto **Insight Energy**, integrando funcionalidades de monitoramento, controle de infraestrutura e cloud computing para o ambiente MiniDataCloud no hardware **Satellite L505**.

---

### **1. Nova Arquitetura do Projeto**

#### **Configuração de Hardware: Satellite L505**
- **Processador:** Intel Core i3 M330 (2.13GHz, 2 núcleos, 4 threads, cache L3 3 MB).  
- **Capacidade Operacional:** Virtualização **VT-x** habilitada para ambientes Kubernetes.  
- **Sistema Operacional:** Ubuntu Server, configurado com ferramentas de suporte a cloud e monitoramento:
  - **Ubuntu Cloud Tools:** Pré-configurado para Kubernetes.
  - **Netdata:** Para monitoramento em tempo real.
  - **KVM:** Para gerenciamento de VMs locais.
  
---

#### **Cluster Kubernetes com Replicação Multi-Nuvem**
**Objetivo:** Criar uma infraestrutura híbrida com replicação e escalabilidade automática para clouds públicas.  

- **Ferramentas Necessárias:**
  - **MicroK8s:** Cluster Kubernetes bare-metal.
  - **Velero:** Backup e recuperação de ambientes.
  - **ArgoCD:** Sincronização contínua entre o cluster local e clouds públicas (Azure, GCP, AWS, Oracle, Alibaba).  
  - **MetalLB:** Load Balancer para IPs locais em bare-metal.

**Configuração Inicial:**
```bash
sudo snap install microk8s --classic
microk8s status --wait-ready
microk8s enable dns storage metallb
```
Adicione um range de IPs locais ao MetalLB para balanceamento.  

**Replicação Externa:**
Utilize **Velero**:
```bash
velero install --provider gcp --bucket <bucket-name> --secret-file <credentials-file>
```

---

### **2. Monitoramento Avançado de Infraestrutura**

#### **Monitoramento de Ventoinhas**
**Diagnóstico:**  
As ventoinhas são controladas pela CPU via sensores térmicos, configurados com o **lm-sensors**.  

**Algoritmo de Controle Térmico:**  
Manter a CPU na faixa entre o ponto de orvalho (20°C) e 40°C, com RPM variável proporcional à temperatura.

**Código do Algoritmo:**
Veja [aqui](#algoritmo-controle-termico).

#### **Netdata:**  
1. **Configuração de Sensores:**
   ```bash
   sudo apt install lm-sensors
   sudo sensors-detect
   ```
2. **Adicionando Gráficos de RPM e Temperatura:**
   Configure o Netdata para capturar e exibir os dados das ventoinhas e núcleos.  

---

#### **Monitoramento Climático Externo**
1. **Localização do Servidor:**
   Obtenha a localização geográfica por IP usando:
   ```bash
   curl http://ipinfo.io
   ```
2. **Dados Climáticos Externos:**  
   Integre a API do **OpenWeatherMap** para coletar dados de temperatura, umidade e pressão da estação mais próxima.

**Exemplo de Integração:**
```python
import requests
API_KEY = "sua_api_key"
LAT, LON = "latitude", "longitude"

def get_weather():
    url = f"http://api.openweathermap.org/data/2.5/weather?lat={LAT}&lon={LON}&appid={API_KEY}&units=metric"
    response = requests.get(url).json()
    return response['main']['temp'], response['main']['humidity']
```

---

### **3. Monitoramento Energético e Treemaps**

#### **Algoritmo de Estimativa de Consumo:**
O consumo energético será calculado em tempo real com base em:
- **Utilização da CPU:** Dados do Intel RAPL.  
- **Consumo de RAM e Disco:** Fórmula aproximada:
  ```plaintext
  Consumo (Watts) ≈ [(CPU Util % x TDP) + (RAM Util x 0.3) + (Disco I/O x 0.2)]
  ```

#### **Treemaps Dinâmicos:**
- Crie hierarquias de consumo (por servidor, núcleo, aplicação) usando **D3.js** ou **Plotly**.
- Configure o Netdata para exportar dados para **Grafana** com **InfluxDB** como backend.

**Exemplo de Exportação do Netdata:**
```bash
sudo apt install influxdb
netdata export influxdb
```

---

### **4. Monitoramento da Fonte de Energia**
**Fonte de Alimentação:** IT Blue LEW 2562 (19V, 4.5-5A).  
Integre sensores externos para monitoramento de voltagem e corrente, com exportação de dados para o Netdata.  

**Cálculo de Custos Energéticos:**  
```plaintext
Custo (R$/h) = Consumo Total (kWh) x Tarifa (R$/kWh)
```

---

### **5. Algoritmos e Códigos**

#### **Algoritmo Controle Térmico**
```python
# Algoritmo já fornecido acima.
```

---

### **6. Integração Multi-Nuvem com Segurança**

**Backup e Recuperação com Velero:**  
Garanta que todos os dados de replicação estejam seguros utilizando backups automáticos com criptografia.  

**Monitoramento de Segurança:**  
Implemente políticas de segurança baseadas no CIS Kubernetes Benchmark.  

---

### **7. Versão Atual do Projeto**

- **Versão:** `v2.1.0`  
- **Componentes Integrados:** Kubernetes, Netdata, Velero, Monitoramento Térmico, Cloud Pública.

Este projeto entrega um **MiniDataCloud** eficiente e modular, com foco em monitoramento, segurança e replicação distribuída.

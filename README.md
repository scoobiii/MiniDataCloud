# MiniDataCloud - **Projeto Insight Energy**

## **Versão do Projeto: v2.1.0**

Este documento descreve a atualização e versão do projeto **MiniDataCloud**, integrando ferramentas para escalabilidade, monitoramento e controle de energia em uma infraestrutura híbrida de cloud computing e servidores bare-metal.

---

### **1. Arquitetura do Projeto**

#### **Configuração de Hardware: Satellite L505**
- **Processador:** Intel Core i3 M330 (2.13GHz, 2 núcleos, 4 threads, cache L3 3 MB).
- **Capacidade de Virtualização:** VT-x habilitada para ambientes Kubernetes.
- **Sistema Operacional:** Ubuntu Server, configurado com suporte a ferramentas de cloud e monitoramento:
  - **Ubuntu Cloud Tools:** Pré-configurado para Kubernetes.
  - **Netdata:** Monitoramento em tempo real.
  - **KVM:** Gerenciamento de VMs locais.

---

### **Fase 1: MAAS Bare-metal com MongoDB Local**

#### **MAAS Bare-metal**
O MAAS é configurado para gerenciar servidores físicos locais. Durante esta fase inicial, o foco é o gerenciamento de servidores bare-metal sem a necessidade de racks ou controladores regionais.

**Comando para instalação do MAAS**:
```bash
sudo apt install maas
```

#### **MongoDB Local**
A instalação do MongoDB local permite o gerenciamento de dados em um ambiente controlado. Para esta fase, a replicação de dados pode ser evitada.

**Comando para instalação do MongoDB**:
```bash
sudo apt install mongodb
```

**Configuração do Banco de Dados MongoDB:**
- Criação de bancos de dados para armazenar logs e monitoramento de dados de consumo energético e desempenho dos servidores.

---

### **Fase 2: MAAS Region + Rack com Kubernetes**

#### **MAAS Region + Rack**
O MAAS é expandido para configurar um ambiente multi-nuvem, com um controlador de região gerenciando a nuvem pública e o controlador de rack gerenciando os servidores bare-metal locais.

**Comando para instalação MAAS Region + Rack**:
```bash
sudo maas init region+rack
```

#### **Kubernetes em Nuvem e Bare-metal**
**MicroK8s** é utilizado para configurar o cluster Kubernetes em servidores bare-metal e nuvem pública.

**Comando para instalação do Kubernetes (MicroK8s)**:
```bash
sudo snap install microk8s --classic
microk8s status --wait-ready
microk8s enable dns storage metallb
```

#### **Replicação Multi-Nuvem com Velero**
**Velero** é utilizado para backup e recuperação da infraestrutura, garantindo a consistência dos dados entre ambientes híbridos.

**Comando para instalação do Velero**:
```bash
velero install --provider gcp --bucket <bucket-name> --secret-file <credentials-file>
```

---

### **2. Monitoramento Avançado de Infraestrutura**

#### **Monitoramento de Ventoinhas**
Instale o **lm-sensors** para monitorar a temperatura da CPU e as ventoinhas.

**Comando para instalação do lm-sensors**:
```bash
sudo apt install lm-sensors
sudo sensors-detect
```

#### **Netdata: Monitoramento em Tempo Real**
Utilize **Netdata** para monitorar o uso de CPU, memória, e o consumo de energia, com integração a gráficos de temperaturas e ventoinhas.

    http://192.168.15.21:19999/=false&chartName-val=menu_system&local--chartName-val=menu_system&local-overview-sidebarTab-val=chartIndexing&sidebarOpen-bool=true&sidebarTab-val=filters&integrationsModalOpen=&selectedIntegration=deploy-docker&selectedIntegrationTab=&local-overview-tocSearch-val=temperature&tocSearch-val=temperature&threshold=0.01&local-nodesView-nodeIdToGo-val=menu_Live&nodeIdToGo-val=menu_Live&local-nodesView-sidebarTab-val=chartIndexing&local-nodesView-sidebarNodeId-val=d954d2a8-aac9-11ef-a914-70f1a13895fe&sidebarNodeId-val=d954d2a8-aac9-11ef-a914-70f1a13895fe&local--sidebarOpen-bool=true

---

### **3. Monitoramento Climático Externo e Controle Térmico**

#### **Integração com API do OpenWeatherMap**
Recupere dados de temperatura e umidade externos para otimizar o controle térmico interno dos servidores.

**Exemplo de código para integrar OpenWeatherMap**:
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

### **4. Estimativa de Consumo Energético**

Utilize a seguinte fórmula para calcular o consumo de energia baseado em dados de CPU, RAM e I/O de disco:

```plaintext
Consumo (Watts) ≈ [(CPU Util % x TDP) + (RAM Util x 0.3) + (Disco I/O x 0.2)]
```

#### **Cálculo de Custos Energéticos:**
```plaintext
Custo (R$/h) = Consumo Total (kWh) x Tarifa (R$/kWh)
```

---

### **5. Treemaps Dinâmicos para Visualização de Consumo Energético**

- Use **D3.js** ou **Plotly** para criar treemaps dinâmicos visualizando consumo de energia por servidor, núcleo e aplicação.
- Exporte dados do **Netdata** para **Grafana** e armazene-os no **InfluxDB**.

**Comando para exportação do Netdata para InfluxDB**:
```bash
sudo apt install influxdb
netdata export influxdb
```

---

### **6. Monitoramento de Segurança e Backup**

#### **Backup com Velero**
Utilize **Velero** para garantir backups automáticos, com criptografia de dados para maior segurança.

**Comando para instalação do Velero**:
```bash
velero install --provider gcp --bucket <bucket-name> --secret-file <credentials-file>
```

#### **Monitoramento de Segurança**
Implemente políticas de segurança baseadas no **CIS Kubernetes Benchmark**.

---

### **7. Versão Atual do Projeto: v2.1.0**

- **Componentes Integrados:** Kubernetes, MAAS, Velero, Netdata, Monitoramento Térmico, Cloud Pública.
- **Objetivos:** Gerenciamento de infraestrutura híbrida com escalabilidade, alta disponibilidade e monitoramento energético.

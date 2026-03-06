# Guia Completo -- Disaster Recovery com Azure Site Recovery (ASR)

O processo de DR com ASR é dividido em **7 grandes fases**, desde o
planejamento até o failback.

------------------------------------------------------------------------

# 1️⃣ Planejamento e Preparação Estratégica da Infraestrutura

Embora o ASR possa criar recursos automaticamente, **a melhor prática é
provisionar manualmente a infraestrutura de destino** para manter
controle arquitetural e evitar erros em cenários críticos.

## 1.1 Criar Grupo de Recursos (Resource Group)

-   Criar um RG na região secundária (ex: Norte da Europa).
-   Organizar todos os recursos de DR dentro dele.

## 1.2 Criar a Virtual Network (VNet) de Destino

-   Criar VNet na região secundária.
-   Espelhar a arquitetura da origem.
-   Criar sub-redes separadas:
    -   Subnet-App
    -   Subnet-DB

## 1.3 Criar e Configurar Network Security Group (NSG)

-   Criar NSG na região secundária.
-   Configurar regras de entrada:
    -   Porta 80 (HTTP)
    -   Porta 443 (HTTPS)
    -   Porta 1433 (SQL)
-   Associar NSG às sub-redes.

## 1.4 Criar Storage Account de Cache (Item Crítico)

-   Criar na região de origem.
-   Desabilitar **Soft Delete**.
-   O ASR usa o cache para receber snapshots antes de enviar para a
    região de DR.

## 1.5 Criar IP Público (Opcional)

-   Provisionar Public IP na região secundária se necessário.

------------------------------------------------------------------------

# 2️⃣ Criar e Configurar o Recovery Services Vault

## 2.1 Criar Recovery Services Vault

-   Criar na região secundária.
-   Utilizar o mesmo RG do DR (boa prática).

## 2.2 Habilitar Site Recovery

-   Dentro do Vault, selecionar "Site Recovery".
-   Escolher proteção para Azure VMs.

## 2.3 Definir Origem

-   Selecionar região de origem.
-   Selecionar grupo de recursos.
-   Selecionar VMs a proteger.

------------------------------------------------------------------------

# 3️⃣ Personalização da Replicação

## 3.1 Definir Destino

-   Assinatura
-   RG de DR
-   VNet criada
-   Sub-redes corretas

## 3.2 Apontar Storage de Cache

-   Selecionar Storage Account criada na origem.

## 3.3 Configurar Política de Replicação

-   Retenção padrão: 24 horas.
-   Frequência de replicação: \~5 minutos (RPO).

## 3.4 App Consistency (Opcional)

-   Habilitar snapshots consistentes com aplicação.
-   Pode incluir estado da memória.

## 3.5 Mobility Service

-   Extensão instalada nas VMs para gerenciar replicação.

------------------------------------------------------------------------

# 4️⃣ Sincronização Inicial

-   Cópia completa do VHD.
-   Pode levar mais de 1 hora.
-   VMs devem estar ligadas.
-   Não impacta produção (uso de snapshots).

------------------------------------------------------------------------

# 5️⃣ Teste de Failover

## 5.1 Executar Test Failover

-   Criar VMs de teste em rede isolada.
-   Validar SO, NIC e serviços.

## 5.2 Cleanup Test Failover

-   Remover recursos de teste para evitar custos.

------------------------------------------------------------------------

# 6️⃣ Failover Real

## 6.1 Iniciar Failover

-   Selecionar VMs protegidas.

## 6.2 Escolher Recovery Point

-   **RPO (Recovery Point Objective):** Perda máxima aceitável de dados.
-   **RTO (Recovery Time Objective):** Tempo máximo aceitável para
    restabelecimento.

## 6.3 Processo Automático

-   Azure desliga origem.
-   Sobe instâncias na região secundária.

------------------------------------------------------------------------

# 7️⃣ Pós-Failover

## 7.1 Commit

-   Confirma ponto de recuperação definitivo.

## 7.2 Re-protect

-   Inverte fluxo de replicação (Destino → Origem).

## 7.3 Failback

-   Retorna operação para região principal quando estabilizada.

------------------------------------------------------------------------

# Diagrama Estrutural do Disaster Recovery (ASR)

## REGIÃO A (Origem - ex: East US)

-   Máquinas Virtuais (Web e SQL)
-   Discos (VHDs)
-   Mobility Services
-   Snapshots
-   Storage Account de Cache (Soft Delete desabilitado)

------------------------------------------------------------------------

➡ Transferência de Dados via Backbone Azure (HTTPS 443)

------------------------------------------------------------------------

## REGIÃO B (Destino - ex: North Europe)

-   Recovery Services Vault
-   Discos Replicados
-   VNet de DR (Apps-DR / DB-DR)
-   Network Security Group (NSG)
-   IP Público de DR
-   Automation Account (Opcional)

------------------------------------------------------------------------

# Resumo dos Fluxos

**Sincronização:**\
VM (Origem) → Snapshot → Cache Storage → Vault (Destino)

**Failover:**\
Vault → Cria VMs no Destino → Associa IP/NSG → Desliga Origem

**Reprotect:**\
VM (Destino) → Cache (Destino) → Sincronização de volta para Origem

------------------------------------------------------------------------

# Melhores Práticas

-   Provisionar manualmente rede e segurança.
-   Testar DR periodicamente.
-   Monitorar RPO/RTO.
-   Usar App Consistency para cargas críticas.
-   Automatizar tarefas pós-failover.

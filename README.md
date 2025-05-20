
# Guia de Criação e Configuração de VM Ubuntu no Azure com Docker

## 1. Criar Grupo de Recursos

```bash
az group create --name rg-vmubuntu-challenge --location brazilsouth
```

## 2. Criar VM Ubuntu

```bash
az vm create \
  --resource-group rg-vmubuntu-challenge \
  --name vm-ubuntu \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --vnet-name nnet-Linux \
  --nsg nsgsr-linux \
  --public-ip-address pip-ubuntu \
  --authentication-type password \
  --admin-username admlnx \
  --admin-password Chall@557505
```

## 3. Criar regras NSG para Portas

### 3.1. Regra NSG para Porta 80 (HTTP)

```bash
az network nsg rule create \
  --resource-group rg-vmubuntu-challenge \
  --nsg-name nsgsr-linux \
  --name port_80 \
  --protocol tcp \
  --priority 1010 \
  --destination-port-range 80
```
### 3.2. Regra NSG para Porta 8080

```bash
az network nsg rule create \
  --resource-group rg-vmubuntu-challenge \
  --nsg-name nsgsr-linux \
  --name port_8080 \
  --protocol tcp \
  --priority 1020 \
  --destination-port-range 8080
```
### 3.3. Regra NSG para Porta 443 (HTTPS)

```bash
az network nsg rule create \
  --resource-group rg-vmubuntu-challenge \
  --nsg-name nsgsr-linux \
  --name port_443 \
  --protocol tcp \
  --priority 1030 \
  --destination-port-range 443
```

## 4. Conectar na VM via SSH

```bash
ssh admlnx@4.201.201.207
```

## 5. Instalar Docker na VM

- Atualizar sistema:

```bash
sudo apt update
sudo apt upgrade -y
```

- Instalar pacotes de pré-requisito:

```bash
sudo apt install -y apt-transport-https ca-certificates curl gnupg software-properties-common
```

- Criar diretório para a chave GPG do Docker:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

- Adicionar chave GPG oficial do Docker:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

- Adicionar repositório Docker:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

- Atualizar índice de pacotes novamente para incluir repositório Docker:

```bash
sudo apt update
```

- Instalar Docker e plugins recomendados:

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- Verificar se Docker está ativo:

```bash
sudo systemctl status docker
```

## 6. Scripts Testes

### Redimensionar VM

```bash
#!/bin/bash

echo "Entre com o nome do Grupo de Recursos:"
read resourceGroupNamex

echo "Informe o nome da VM:"
read vmnamex

echo "Escolha o novo tamanho:"
select vmsizex in "01 CPU / 2GB RAM" "02 CPUs / 8GB RAM" "04 CPUs / 16GB RAM"; do
  case $vmsizex in
    "01 CPU / 2GB RAM") export vmsizex="Standard_B1ms";;
    "02 CPUs / 8GB RAM") export vmsizex="Standard_D2s_v3";;
    "04 CPUs / 16GB RAM") export vmsizex="Standard_B4ms";;
  esac
  break
done

az vm resize \
  --resource-group "$resourceGroupNamex" \
  --name "$vmnamex" \
  --size "$vmsizex"
```

### Abrir Porta na VM com Prioridade Aleatória

```bash
#!/bin/bash

echo "Entre com o nome do Grupo de Recursos:"
read resourceGroupNamex

echo "Informe o nome da VM:"
read vmnamex

echo "Número da porta a ser aberta:"
read vmopenportx

# Gera uma prioridade aleatória entre 100 e 4195 (máximo permitido pelo Azure é 4096)
priority=$(( RANDOM % 3997 + 100 ))

az vm open-port \
  --resource-group "$resourceGroupNamex" \
  --name "$vmnamex" \
  --port "$vmopenportx" \
  --priority "$priority"
```

## 7. Comando para Deletar IP Público (Se Necessário)

```bash
az network public-ip delete --name vm-lnxubuntu-chall-southbrazil-001-ip --resource-group rg-mkt-chall-001
```

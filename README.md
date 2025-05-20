
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

## 3. Criar Regra NSG para Porta 80

```bash
az network nsg rule create \
  --resource-group rg-vmubuntu-challenge \
  --nsg-name nsgsr-linux \
  --name port_80 \
  --protocol tcp \
  --priority 1010 \
  --destination-port-range 80
```

## 4. Conectar na VM via SSH

```bash
ssh admlnx@4.201.155.153
```

## 5. Instalar Docker na VM

- Atualizar sistema:

```bash
sudo apt update
sudo apt upgrade -y
```

- Instalar pacotes de pré-requisito:

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg -y
```

- Adicionar chave GPG oficial do Docker:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

- Adicionar repositório Docker:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

- Atualizar pacotes e instalar Docker:

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
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

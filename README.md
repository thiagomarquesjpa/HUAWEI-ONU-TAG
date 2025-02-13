# Configuração da OLT Huawei para Envio de VLAN como TAG pela ONU

## Objetivo
Configurar a OLT Huawei para entregar VLANs como TAG na porta Ethernet da ONU.

## VLANs utilizadas
- VLAN 900 e 901 destinadas ao cliente.

## Interface de Uplink
- Slot da OLT: **0/8**
- Porta: **0**

## Passos de Configuração

### 1. Criar um DBA Profile para controle de velocidade
Exemplo para 1 Gbps:
```shell
 dba-profile add profile-id 400 profile-name "1GB" type4 max 1024000
```

### 2. Configurar as VLANs na OLT
```shell
 vlan 900 to 901 smart
 port vlan 900 to 901 0/8 0
```

### 3. Criar o Service Profile
```shell
 ont-srvprofile gpon profile-id 190 profile-name "SRV-ONU_TRANSPARENT"
   ont-port eth 1
   ring check enable
   port vlan eth 1 transparent
   commit
```

### 4. Criar o Line Profile
```shell
 ont-lineprofile gpon profile-id 190 profile-name "LINE-ONT_TRANSPARENT"
   fec-upstream enable
   mapping-mode port
   tcont 1 dba-profile-id 400
   gem add 1 eth tcont 1
   gem mapping 1 0 eth 1
   commit
   quit
```

### 5. Autorizar a ONU na OLT
Substituir `xxxxxxxxxxxxx` pelo Serial Number da ONU.
```shell
 ont add 0 0 sn-auth xxxxxxxxxxxxx omci ont-lineprofile-id 190 ont-srvprofile-id 190 desc transporte_cliente_XXXXX
```

### 6. Criar o Service Port da ONU
```shell
 service-port vlan 900 gpon 0/0/0 ont 0 gemport 1 multi-service user-vlan 900 tag-transform transparent
 service-port vlan 901 gpon 0/0/0 ont 0 gemport 1 multi-service user-vlan 901 tag-transform transparent
```

## Resultado
Com essa configuração, a porta Ethernet da ONU entregará as VLANs **900 e 901** como TAG para o cliente.

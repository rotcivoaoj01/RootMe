
# Write-up: TryHackMe - RootMe

Este laboratório consistia em um CTF focado em upload de arquivos maliciosos (Reverse Shell) e escalação de privilégios.

## 1. Reconhecimento (Recon)

Iniciei verificando os serviços rodando no alvo com o Nmap:

```bash
nmap -sS -T5 10.113.182.238
```

A varredura revelou duas portas abertas: a **22 (SSH)** e a **80 (HTTP)**. Em seguida, utilizei o **Gobuster** para enumerar diretórios no servidor web, encontrando as pastas `/panel` e `/uploads`.

```bash
gobuster dir -u http://10.113.182.238 -w /usr/share/wordlists/Discovery/Web-Content/common.txt
```

<img width="958" height="984" alt="1" src="https://github.com/user-attachments/assets/a46e51dc-f442-4f3f-827b-92bf8b3d47d9" />


## 2. Exploração (Upload e Reverse Shell)

Acessei o painel de upload e enviei um script de **Reverse Shell** com a extensão `.php5`, que passou pelo filtro de segurança.

<img width="898" height="981" alt="3" src="https://github.com/user-attachments/assets/37dfbfc4-a25e-49aa-9b04-4b4547bd5026" />


O arquivo ficou disponível no diretório `/uploads`.

<img width="898" height="981" alt="4" src="https://github.com/user-attachments/assets/31ac1107-9c75-409c-a98d-6a6d830d121c" />


Antes de subir, eu preparei o script alterando o IP e a porta para apontar para a minha máquina usando o `nano`. Embaixo, é possível ver que já consegui acessar e ler a primeira flag de usuário.

```bash
nano php-reverse-shell.php5
```

<img width="958" height="984" alt="2" src="https://github.com/user-attachments/assets/4d55de03-cdfe-49a0-9993-960c8e2a405f" />


Para receber a conexão, abri um listener na minha máquina atacante e acessei o arquivo pelo navegador. O servidor executou o script e me enviou a shell. Em seguida, iniciei a busca por binários com permissão SUID.

<img width="885" height="249" alt="7" src="https://github.com/user-attachments/assets/c7dae895-b7ac-4320-9e28-a02002790ede" />


## 3. Primeira Flag (User)

Já com a shell, procurei o arquivo `user.txt` no sistema:

```bash
find / -name user.txt 2>/dev/null
```

<img width="378" height="44" alt="5" src="https://github.com/user-attachments/assets/b479606f-68db-4fe1-b8eb-6919e5933dc4" />


## 4. Escalação de Privilégios (Root)

Agora o objetivo era me tornar root.

<img width="898" height="981" alt="6" src="https://github.com/user-attachments/assets/112e7289-01ee-479e-9d7c-604c3740557f" />


A busca por SUID revelou o binário `/usr/bin/python`. Consultei o site **GTFOBins** e encontrei o comando exato para explorar essa falha.


<img width="908" height="582" alt="9" src="https://github.com/user-attachments/assets/d90f4efc-9917-4580-bf52-ef698b1bb729" />

Executei o comando e obtive acesso total como root (`uid=0`).

```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

Por fim, li a flag de root:

```bash
cat /root/root.txt
```

<img width="891" height="407" alt="8" src="https://github.com/user-attachments/assets/146e7a15-aa1c-4b7b-abe3-f1383ca05e20" />


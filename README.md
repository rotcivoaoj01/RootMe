***

# Write-up: TryHackMe - RootMe

Este laboratório consistia em um CTF focado em upload de arquivos maliciosos (Reverse Shell) e escalação de privilégios.

## 1. Reconhecimento (Recon)

Iniciei verificando os serviços rodando no alvo com o Nmap:

```bash
nmap -sS -T5 10.113.182.238
```

A varredura revelou duas portas abertas: a **22 (SSH)** e a **80 (HTTP)**, indicando um servidor web.

![Nmap](Screenshot From 2026-11-16 00-07-53.png)

Em seguida, utilizei o **Gobuster** para enumerar diretórios e arquivos no servidor web:

```bash
gobuster dir -u http://10.113.182.238 -w /usr/share/wordlists/Discovery/Web-Content/common.txt
```

A enumeração encontrou uma página `index.php`, o diretório `panel` (que continha a funcionalidade de upload) e o diretório `uploads` (onde os arquivos enviados ficam salvos).

![Gobuster](Screenshot From 2026-11-16 00-08-06.png)

## 2. Exploração (Upload e Reverse Shell)

Para obter acesso, utilizei um script de **Reverse Shell** (PentestMonkey) e alterei o IP e a porta no código para apontar para a minha máquina.

```bash
nano php-reverse-shell.php5
```

![Nano](Screenshot From 2026-11-16 00-08-26.png)

Acessei o painel de upload e enviei o arquivo com a extensão `.php5`, que passou pelo filtro de segurança. O arquivo ficou disponível no diretório `/uploads`.

![Uploads](Screenshot From 2026-11-16 00-08-39.png)

![Upload Sucesso](Screenshot From 2026-11-16 00-08-46.png)

Para receber a conexão, abri um listener na minha máquina atacante e acessei o arquivo malicioso pelo navegador. O servidor web executou o script e me enviou a shell!

![Listener](Screenshot From 2026-11-16 00-08-50.png)

## 3. Primeira Flag (User)

Já com a shell, procurei o arquivo `user.txt` no sistema:

```bash
find / -name user.txt 2>/dev/null
```

![Find User](Screenshot From 2026-11-16 00-27-08.png)

Encontrei e li a primeira flag no diretório `/var/www`:

![User Flag](Screenshot From 2026-11-16 00-27-25.png)

## 4. Escalação de Privilégios (Root)

Para escalar privilégios, procurei por binários com o bit **SUID** ativado:

```bash
find / -user root -perm -4000 2>/dev/null
```

![Find SUID](Screenshot From 2026-11-16 00-27-50.png)

A saída revelou o binário `/usr/bin/python` com permissão SUID. Consultei o site GTFOBins e encontrei o comando para explorar essa falha:

*(Se você tiver a imagem do GTFOBins, insira o nome dela aqui)*

Executei o comando e obtive acesso total como root (`uid=0`), confirmando com o comando `id`.

```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

![Root Access](Screenshot From 2026-11-16 00-08-50.png) *(Nota: Coloque aqui a imagem onde o `id` retorna root e a flag é lida, caso tenha um print específico dela)*

Por fim, li a flag de root:

```bash
cat /root/root.txt
```

---
**Desafio RootMe concluído com sucesso!** 🚀

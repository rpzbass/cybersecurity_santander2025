# Desafio I CyberSecurity 2025
```
A abordagem que utilizei para o desafio I do Cyber Security 2025, serão detalhadas nos próximos passos abaixo, porem para evitar a “verbosidade” omitirei detalhes técnicos primários de configuração do Virtual Box das VM’s (KALI) e (Metasploitable2).
```
## Configurações 
```
O essecial, deixei a interface como host-only ou rede interna para a comunicação das VM's.
```


![This is an alt text.](/images/rede_interna_vbox.png "Rede interna VirtualBox")

<br>

###
```
Garantindo que as duas VM’s estão na mesma subnet 192.168.1.0/24, segue a vm’s Kali com o IP 192.168.1.119 e Metasploitable2 192.168.1.202 com o comando:
```
<br>

<div style="background: #020005ff; padding: 10px; border-radius: 5px; font-family: monospace;">
<span style="color: #00ff00;"> ifconfig </span>
</div>

<br>

![Sistema Kali](/images/sistema_kali.png "Sistema Kali")

<br>


![This is an alt text.](/images/sistema_metasploitable.png "Metasploitable")

<br>

### 
```
 Primeiramente para não ter que ficar digitando IP, eu incluo o IP do Metasploitable no arquivo /etc/hosts,conforme a ilustrado.
```
![This is an alt text.](/images/etc_hosts.png "etc/hosts")

<br>

## RECONHECIMENTO

```
Agora sim, para efetuar o reconhecimento dos serviços utilizados no nosso servidor vulnerável e para ficar mais realista, utilizei o comando nmap: 
```

<br>

<div style="background: #020005ff; padding: 10px; border-radius: 5px; font-family: monospace;">
<span style="color: #00ff00;"> nmap -sS -A metasploit.local </span>
</div>

<br>

```
Uma observação apenas, na imagem mostra apenas uma parte do resultado, pois em um dos meus testes acabei utilizando outra abordagem mais longa, explorando o serviço vsftpd 2.3.4 ,que é uma possibilidade, mas considere o serviço SMB no resultado, acabei não colocando um print atualizado.
```


![This is an alt text.](/images/scan.png "Recon")


## ENUMERANDO SMB

```
Seguindo, decidi explorar o serviço SMB como sugerido, utilizando o seguinte comando:
```

<div style="background: #020005ff; padding: 10px; border-radius: 5px; font-family: monospace;">
<span style="color: #00ff00;">enum4linux -a metasploit.local > log_resultado_enum.txt </span>
</div>

<br>

```
Utilizando o comando cat log_result_enum.txt  com parâmetros de filtro como grep, obtive uma saída mais limpa, com os seguintes usuários potenciais.
```
<br>

![This is an alt text.](/images/potenciais_usuarios_smb.png "grep user")

<br>

```
Utilizando echo e o parâmetro -e  para caracteres de escape  como “\n” que é interpretado como nova linha, obtivemos como saída uma lista de usuários.

```

<br>

![This is an alt text.](/images/usuarios_list.png "grep user")

<br>

```
Considerando que é um desafio de replicação e entendimento da metodologia, utilizamos novamente o mesmo comando para criar uma lista de possíveis senhas previsíveis, porem em um cenário real, podemos utilizar wordlists conhecidas da internet como rockyou ou as que ficam no próprio Kali Linux, utilizando as que estejam no contexto de senha.
```

<br>

![This is an alt text.](/images/senhas_spray.png "grep user")

<br>

## EXPLORAÇÃO SMB

```
Concluído a fase anterior, efetuamos a fase de exploração utilizando a ferramenta Medusa conforme sugerido em aula, as flags significam -h para server, -U lista de usuários, -P lista de senhas, -M modulo do protocolo SMB no caso smbnt, -t para número de logins a ser testado, -T PARA número total de hosts a ser testado,  pode também ser utilizado a hydra,  muito semelhante, segue o comando:
```

<div style="background: #020005ff; padding: 10px; border-radius: 5px; font-family: monospace;">
<span style="color: #00ff00;">medusa -h metasploit. local -U usuários.txt -P senhas_splay.txt -M smbnt -t 2 -T 50  v </span>
</div>

<br>

```
Faltou uma marcação no print, mas repare que encontramos o  usuário e senha (msfadmin) e (user).
```

![This is an alt text.](/images/exploracao_smb.png "exploracao smb")


```
Validando acesso, utilizamos o comando smbclient conforme print.
Podemos observar que tivemos acesso alguns compartilhamentos, validando a exploração.
```

<br>


![This is an alt text.](/images/validando_user.png "validando acesso smb")

<div style="background: #020005ff; padding: 10px; border-radius: 5px; font-family: monospace;">
<span style="color: #00ff00;">smbclient -L \\metasploit.local -U user </span>
</div>

<br>

## ENUMERANDO FTP


```
Repare que retomando aquela parte da enumeração smb, podemos notar que existe outro usuário, foco no usuário ftp, pois é esse que utilizaremos para ter sucesso ao ataque.
```


![This is an alt text.](/images/grep_user_result_enum.png "validando acesso smb")


```
Utilizando um código em python , efetuaremos um bruteforce, por questões de espaço mostrei parcialmente o script, pois se trata de algo básico, nada inovador, segue:
```

![This is an alt text.](/images/codigo_brute_force_ftp.png "validando acesso smb")


## EXPLORAÇÃO FTP

```
 É possivel usar os modulos auxiliares da ferramenta msfconsole para brute force via ftp, porem utilizei o script proprio. Alem disso, utilizei a lista rockyou.txt com a ferramenta obtendo o seguinte resultado.
```

![This is an alt text.](/images/ftp_exploracao.png "Exploração ftp")

```
Validando o acesso.
```

![This is an alt text.](/images/validando_acesso_ftp.png "validando acesso ftp")

## EXPLORAÇÃO HTTP

```
Por motivos óbvios, utilizarei como tentativa de exploração o usuário “admin”, em um cenário real também  aplicaria-se, pois esse é largamente utilizado em ambientes de produção. Nessa exploração utilizaremos a ferramenta Hydra, porem para configurar ela é necessário obter alguns parâmetros,como a função de desenvolvimento do navegador que para acionar basta pressionar f12, ou com o  próprio burbsuite,  consegue-se facilmente o “PATH de autenticação” que é o caminho de que precisamos para a flag http-post-form.  Outros parâmetros também são necessários, como os nomes exatos dos campos e variáveis “^USER^” e “^PASS^” do html seguido de “:” e a mensagem de falha que sucede para o programa entender quando errou ou quando teve êxito e tambem ativando as flags de verbosidade.
```
<div style="background: #020005ff; padding: 10px; border-radius: 5px; font-family: monospace;">
<span style="color: #00ff00;">hydra -l admin -P rockyou.txt metasploit.local http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^:Login failed" -vV</span>
</div>
<br>

![This is an alt text.](/images/encontrado_com_hydra_dvwa.png "exploracao http")

```
Observe que encontramos o usuário “admin” seguindo da senha “password”,  validando o acesso, teremos o seguinte resultado.
```

![This is an alt text.](/images/validando_entrada_sistema_dvwa.png "testando credenciais obtidas")



```
935cc82ef2748ac36d8c208173df154a
```
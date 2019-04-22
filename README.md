# Trabalho desenvolvido durante a disciplina de Servidores de Rede e Aplicações

Os procedimentos foram feitos em GNU LINUX Debian Jessie 64

Foi usado o Vagrant, que é um produto de software de código aberto para criação e manutenção de 
ambientes de desenvolvimento de software virtual portáteis, por exemplo, para o VirtualBox,
KVM, Hyper-V, contêineres do Docker, VMware e AWS

O arquivo Vagrantfile desenvolvido neste trabalho é usado como parâmetro de configurações para o Vagrant.
Nele nós criamos os servidores NFS e LDAP juntamente com os clientes



***Instalação do servidor LDAP***

```
# aptitude -y install slapd ldap-utils 
```
Insira a senha para o Admin que está sendo instalado

Usando seu editor de preferência crie um arquivo chamado ***base.ldif*** e insira os seguintes dados. 
Escolha os termos 'dn: ou=people,dc=*** dc=***' conforme preferir


```
dn: ou=people,dc=users,dc=auth
objectClass: organizationalUnit
ou: people

dn: ou=groups,dc=users,dc=auth
objectClass: organizationalUnit
ou: groups  
```
Em seguida entre com o comando:

```
# ldapadd -x -D cn=admin,dc=users,dc=auth -W -f base.ldif 
```
Será pedida a senha que foi estipulada anteriormente.
A saída deverá ser:

```
adding new entry "ou=people,dc=users,dc=auth"

adding new entry "ou=people,dc=users,dc=auth"
```

***Adicionando contas de usuário***

Os grupos, usuários e senhas já deverão existir no sistema, pois serão extraidos neste procedimento

crie um arquivo chamado ***ldapuser.ldif*** e insira os seguintes dados:

```
dn: uid=jessie,ou=people,dc=users,dc=auth
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: Jessie
sn: Debian
userPassword: {SSHA}xxxxxxxxxxxxxxxxx
loginShell: /bin/bash
uidNumber: 1000
gidNumber: 1000
homeDirectory: /home/jessie

dn: cn=jessie,ou=groups,dc=users,dc=auth
objectClass: posixGroup
cn: Jessie
gidNumber: 1000
memberUid: jessie
```
Entre com o comando:

```
# # ldapadd -x -D cn=admin,dc=users,dc=auth -W -f ldapuser.ldif  
```
Será pedida a senha que foi estipulada anteriormente.
A saída deverá ser:

```
adding new entry "uid=jessie,ou=people,dc=users,dc=auth"

adding new entry "cn=jessie,ou=groups,dc=users,dc=auth"
```
Adicionando usuários e grupos em passwd / group local ao diretório LDAP.

crie um arquivo chamado ***ldapuser.sh*** e insira os seguintes dados:

```
#Preste atenção aos termos dc=users,dc=auth, pois podem ser substituidos conforme sua preferência
#!/bin/bash

SUFFIX='dc=users,dc=auth'
LDIF='ldapuser.ldif'

echo -n > $LDIF
GROUP_IDS=()
grep "x:[1-9][0-9][0-9][0-9]:" /etc/passwd | (while read TARGET_USER
do
    USER_ID="$(echo "$TARGET_USER" | cut -d':' -f1)"

    USER_NAME="$(echo "$TARGET_USER" | cut -d':' -f5 | cut -d',' -f1 )"
    [ ! "$USER_NAME" ] && USER_NAME="$USER_ID"

    LDAP_SN="$(echo "$USER_NAME" | awk '{print $2}')"
    [ ! "$LDAP_SN" ] && LDAP_SN="$USER_ID"

    LASTCHANGE_FLAG="$(grep "${USER_ID}:" /etc/shadow | cut -d':' -f3)"
    [ ! "$LASTCHANGE_FLAG" ] && LASTCHANGE_FLAG="0"

    SHADOW_FLAG="$(grep "${USER_ID}:" /etc/shadow | cut -d':' -f9)"
    [ ! "$SHADOW_FLAG" ] && SHADOW_FLAG="0"

    GROUP_ID="$(echo "$TARGET_USER" | cut -d':' -f4)"
    [ ! "$(echo "${GROUP_IDS[@]}" | grep "$GROUP_ID")" ] && GROUP_IDS=("${GROUP_IDS[@]}" "$GROUP_ID")

    echo "dn: uid=$USER_ID,ou=people,$SUFFIX" >> $LDIF
    echo "objectClass: inetOrgPerson" >> $LDIF
    echo "objectClass: posixAccount" >> $LDIF
    echo "objectClass: shadowAccount" >> $LDIF
    echo "sn: $LDAP_SN" >> $LDIF
    echo "givenName: $(echo "$USER_NAME" | awk '{print $1}')" >> $LDIF
    echo "cn: $(echo "$USER_NAME" | awk '{print $1}')" >> $LDIF
    echo "displayName: $USER_NAME" >> $LDIF
    echo "uidNumber: $(echo "$TARGET_USER" | cut -d':' -f3)" >> $LDIF
    echo "gidNumber: $(echo "$TARGET_USER" | cut -d':' -f4)" >> $LDIF
    echo "userPassword: {crypt}$(grep "${USER_ID}:" /etc/shadow | cut -d':' -f2)" >> $LDIF
    echo "gecos: $USER_NAME" >> $LDIF
    echo "loginShell: $(echo "$TARGET_USER" | cut -d':' -f7)" >> $LDIF
    echo "homeDirectory: $(echo "$TARGET_USER" | cut -d':' -f6)" >> $LDIF
    echo "shadowExpire: $(passwd -S "$USER_ID" | awk '{print $7}')" >> $LDIF
    echo "shadowFlag: $SHADOW_FLAG" >> $LDIF
    echo "shadowWarning: $(passwd -S "$USER_ID" | awk '{print $6}')" >> $LDIF
    echo "shadowMin: $(passwd -S "$USER_ID" | awk '{print $4}')" >> $LDIF
    echo "shadowMax: $(passwd -S "$USER_ID" | awk '{print $5}')" >> $LDIF
    echo "shadowLastChange: $LASTCHANGE_FLAG" >> $LDIF
    echo >> $LDIF
done

for TARGET_GROUP_ID in "${GROUP_IDS[@]}"
do
    LDAP_CN="$(grep ":${TARGET_GROUP_ID}:" /etc/group | cut -d':' -f1)"

    echo "dn: cn=$LDAP_CN,ou=groups,$SUFFIX" >> $LDIF
    echo "objectClass: posixGroup" >> $LDIF
    echo "cn: $LDAP_CN" >> $LDIF
    echo "gidNumber: $TARGET_GROUP_ID" >> $LDIF

    for MEMBER_UID in $(grep ":${TARGET_GROUP_ID}:" /etc/passwd | cut -d':' -f1,3)
    do
        UID_NUM=$(echo "$MEMBER_UID" | cut -d':' -f2)
        [ $UID_NUM -ge 1000 -a $UID_NUM -le 9999 ] && echo "memberUid: $(echo "$MEMBER_UID" | cut -d':' -f1)" >> $LDIF
    done
    echo >> $LDIF
done
)
```
Entre com o comando: 

```
#bash ldapuser.sh 
```
Os usuários, grupos e senhas serão importados para o arquivo ***ldapuser.ldif*** 

Agora entre com o comando:

```
# ldapadd -x -D cn=admin,dc=users,dc=auth -W -f ldapuser.ldif 
```

Após entrar com a senha do Admin, a saída do comando deverá ser neste formado

```
adding new entry "uid=joao,ou=people,dc=users,dc=auth"
```

***Configurando Clientes LDAP***

Vamos agora configurar os clientes para usar o serviço de autenticação LDAP

Entre com os comandos de instalação

```
apt -y install libnss-ldap libpam-ldap ldap-utils 
```
Será pedido o ***LDAP server URI:*** 

Nesse caso entrarei com os dados já cadastrados no servidor e continuamos com o OK

```
ldap://users.auth/
```

Em seguida espacifique o sufixo

```
dc=users,dc=auth
```

Especifique a versão do LDAP que deseja, no caso usamos o ***3***

```
3
```
Especifique o sufixo para a conta de administrador do LDAP

```
LDAP account for root:
cn=admin,dc=users,dc=auth
```

Defina a senha para a conta de administrador do LDAP
```
LDAP root account password:                                             
                                                                        
********
```
Será demonstrado uma orientação para que seja configurado o arquivo ***nsswitch.conf*** posteriormente

A próxima opção permitirá que os utilitários de senha que usam o PAM alterem senhas. Escolha ***YES***
Na opção seguinte será perguntado se o banco de dados LDAP requer login, escolha ***NO***



Especifique o sufixo da conta de administrador do LDAP e continue com o OK
```
LDAP administrative account:                                             
                                                                           
cn=admin,dc=srv,dc=world_

```
Especifique senha para a conta de administrador do LDAP

```

LDAP administrative password:                                             
                                                                            
********
```

Daqui em diante teremos que editar alguns arquivos
O primeiro dele será o ***/etc/nsswitch.conf ***

Na linha 7, adicione ***ldap*** 
O arquivo deve ficar dessa forma:

```
passwd:compat ldap
group:compat ldap
shadow:compat ldap
```

O próximo arquivo a ser editado será o ***/etc/pam.d/common-password ***

Na linha 26 retire a parte ( remove 'use_authtok' ), deverá ficar assim

```
password     [success=1 user_unknown=ignore default=die]     pam_ldap.so try_first_pass
```

Devemos alterar também o arquivo ***/etc/pam.d/common-session *** adicionando no final do arquivo os seguintes comandos

```
session optional        pam_mkhomedir.so skel=/etc/skel umask=077
```

Reinicie o sistema e use os usuários cadastrados no servidor LDAP para logar

***Configurando o Servidor NFS***

Instalando os programas necessários

```
#apt -y install nfs-kernel-server 
```

Após a instalação a configuração do servidor seria editar o arquivo exports localizado em ***/etc/exports** conforme o seguinte exemplo:


```
/home/user/teste_nfs 192.168.0.104(rw) 
```
Usamos a opção (rw) que permite solicitações de leitura e gravação em um volume NFS. Existem inumeras outras possibilidades, porém não fazem parte do escopo pedido.

Após o procedimento é necessário reiniciar o serviço, faça da seguinte forma:

```
service nfs-kernel-server restart
```
Para se conectar ao servidor podemos usar comandos como o seguinte exemplo:

```
sudo mount -t nfs 192.168.0.104:/mnt/arquivos /ArquivosCompartilhados
```


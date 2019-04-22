# Trabalho desenvolvido durante a disciplina de Servidores de Rede e Aplicações

Os procedimentos foram feitos em GNU LINUX Debian Jessie 64

Foi usado o Vagrant, que é um produto de software de código aberto para criação e manutenção de 
ambientes de desenvolvimento de software virtual portáteis, por exemplo, para o VirtualBox,
KVM, Hyper-V, contêineres do Docker, VMware e AWS



Instalação do servidor LDAP

```
# aptitude -y install slapd ldap-utils 
```
Insira a senha para o Admin que está sendo instalado

Usando seu editor de preferência crie um arquivo chamado ***base.ldif*** e insira os seguintes dados 
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
# ldapadd -x -D cn=admin,dc=srv,dc=world -W -f base.ldif 
```
Será pedida a senha que foi estipulada anteriormente.
A saída deverá ser:

```
adding new entry "ou=people,dc=users,dc=aut"

adding new entry "ou=people,dc=users,dc=aut"
```

Adicionando contas de usuário

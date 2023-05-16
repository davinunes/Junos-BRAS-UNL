Criei uma vm simplificada com freeradius3 para utilizar somente neste LAB, utilizando estes tutoriais: 
- https://blog.remontti.com.br/2066
- https://blog.remontti.com.br/4085

Download da VM para eve-ng/pnetlab:
- [link](https://drive.google.com/drive/folders/1gc4C__nOHeNEQkKmJoyEvYnSyj7TLSgT?usp=sharing)

Credenciais necessárias para possiveis testes: 

| Plataforma | Usuário | Senha | 
| ------------- | --- | --- | 
| linux | root | eve | 
| phpmyadmin | radius | SENHA-DO-USER-RADIUS |
| mysql | root | SUA-SENHA-USER-ROOT | 


Para fazer que o radius aceite *qualquer ip* neste lab, não sendo necessário adicionar o cliente no banco de dados, eu editei o arquivo `/etc/freeradius/3.0/clients.conf` e adicionei o seguinte:
 
```php
client LAB {
        ipaddr          = 10.0.0.0/8
        secret          = radius
}
```

Porém, o recomendado é adicionar o cliente no banco:
```sql
INSERT INTO `nas` (`nasname`, `shortname`, `type`, `ports`, `secret`, `server`, `community`, `description`) VALUES
-- Juniper --
('10.3.1.2', 'Junos', 'other', NULL, 'radius', NULL, NULL, 'JUN PPPoE')
```

Nas tabelas radcheck, radusergroup e radgroupreply inseri estes dados:

```sql
INSERT INTO `radcheck` (`id`, `username`, `attribute`, `op`, `value`) VALUES
(1, 'teste1', 'Cleartext-Password', ':=', 'teste1'),
(2, 'teste2', 'Cleartext-Password', ':=', 'teste2');

INSERT INTO `radusergroup` (`username`, `groupname`, `priority`) VALUES
('teste1', 'FTTH_025M', 1),
('teste2', 'FTTH_050M', 1);

INSERT INTO `radgroupreply` (`id`, `groupname`, `attribute`, `op`, `value`) VALUES
(1, 'FTTH_025M', 'ERX-Service-Activate:1', '=', '\"IPV4(20M,30M)\"'),
(2, 'FTTH_050M', 'ERX-Service-Activate:1', '=', '\"IPV4(30M,50M)\"'),
(3, 'FTTH_050M', 'ERX-Service-Activate:2', '=', '\"IPV6(30M,50M)\"');

INSERT INTO nas (nasname, shortname, type, secret, server, community, description)
VALUES ('10.0.0.2', 'Junos', 'cisco', 'radius', NULL, NULL, 'Roteador de laboratorio');
```

Podemos testar no Junos se a conexão com a Radius está ok:


Tudo certo, funcionando!
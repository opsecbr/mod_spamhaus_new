# mod_spamhaus_new

Apache 2.4 security: mod_spamhaus_new is an Apache module that uses DNSBL in order to block spam relay via web forms, 
preventing URL injection, block http DDoS attacks from bots and generally protecting your web service 
denying access to a known bad IP address. 

This module is based on mod_spamhaus but has been updated for actual web server configurations and to
support a list of domains, which are NOT spam blocked so customers can reach you even if they got a 
dynamic IP which is on a spam list.

Default configuration takes advantage of the Spamhaus Block List (SBL) and the Exploits Block List (XBL)
querying sbl-xbl.spamhaus.org but you can use a different DNSBL, for example local rbldnsd instance of 
sbl-xbl (increasing query performance). Spamhaus's DNSBLs are offered as a free public service for 
low-volume non-commercial use. 

To check if you qualify for free use, please see:
Spamhaus DNSBL usage criteria (https://www.spamhaus.org/organization/dnsblusage.html)

©2022 Kaufmann Automotive GmbH
https://www.kaufmann-automotive.ch

Copyright: Copyright (C) 2008 Luca Ercoli  <luca.e [at] seeweb.it>
                         2018 Rainer Kaufmann <info [at] kaufmann-automotive.ch>

## O QUE É O mod_spamhaus_new

mod_spamhaus_new is an Apache module that uses DNSBL in order to block spam relay via web forms, preventing URL injection, block http DDoS attacks from bots and generally protecting your web service denying access to a known bad IP address. Default configuration takes advantage of the Spamhaus Block List (SBL) and the Exploits Block List (XBL) querying sbl-xbl.spamhaus.org but you can use a different DNSB, for example local rbldnsd instance of sbl-xbl (increasing query performance). Spamhaus's DNSBLs are offered as a free public service for low-volume non-commercial use. To check if you qualify for free use, please see: Spamhaus DNSBL usage criteria (http://www.spamhaus.org/organization/dnsblusage.html)

## COMPILAÇÃO

### Pré-requisitos

* Apache 2.4.X - http://www.apache.org/
Other versions may work but have not been tested
* Docker 27.x.X

### Compilando

Para realizar a compilação sem ter que instalar pacotes no sistema operacional, vamos utilizar um container do docker.
Presumindo que você tenha o `docker` instalado em seu computador, ou no computador que será utilizado para compilação, execute os seguintes comandos:

```
$ git clone https://github.com/opsecbr/mod_spamhaus_new.git
$ cd mod_spamhaus_new
$ docker run -it --rm -w /mod_spamhaus_new -v ${PWD}:/mod_spamhaus_new ubuntu:22.04 bash
# apt-get update && apt-get install build-essential apache2-dev -y
# make && exit
$ ls -l src/.libs/mod_spamhaus_new.so
```

A biblioteca foi compilada e esta disponível em `src/.libs/mod_spamhaus_new.so` dentro do diretório do projeto que foi clonado.

## INSTALAÇÃO

Os comandos abaixo tem o objetivo de instalar e criar os arquivos de configuração básicos do módulo, sendo compatíveis para execução em sistemas operacionais Ubuntu 22.04/24.04, com o Apache2 2.4.* já instalado.

Considernado que você ainda esta dentro do diretório do projeto que foi clonado, execute os comandos:

```
$ sudo cp -a src/.libs/mod_spamhaus_new.so /usr/lib/apache2/modules/mod_spamhaus_new.so
$ echo 'localhost' | sudo tee /etc/spamhaus.unaffected
$ echo '127.0.0.1' | sudo tee /etc/spamhaus.wl
$ echo "LoadModule spamhaus_new_module /usr/lib/apache2/modules/mod_spamhaus_new.so" | sudo tee /etc/apache2/mods-available/spamhaus_new.load
$ cat <<EOF | sudo tee /etc/apache2/mods-available/spamhaus_new.conf
<IfModule mod_spamhaus_new.c>

    # HTTP methods that should be checked
    MS_METHODS POST,PUT,OPTIONS,CONNECT,GET

    # Whitelist of IP addresses and address ranges
    MS_WhiteList /etc/spamhaus.wl

    # List of domains that should not be affected
    MS_UnaffectedDomains /etc/spamhaus.unaffected

    # Used DNS-based blackhole list
    MS_Dns `dnsbl.rbl.com`

    # IP cache size
    MS_CacheSize 4096

    # IP cache entry validity
    MS_CacheValidity 300

    # Error message that is presented to the user
    MS_CustomError "Access Denied! Your IP address is blacklisted because of malicious behavior in the past."

</IfModule>
EOF
$ sudo a2enmod spamhaus_new
$ sudo apache2ctl configtest
$ sudo systemctl restart apache2
```

> [!NOTE]
> O caminho para mod_spamhaus_new.so depende da instalação do seu Apache2.

> [!IMPORTANT]
> Lembre-se de ajustar o nome da DNSBL/RBL que será utilizada alterando o parâmetro `MS_Dns` no arquivo `/etc/apache2/mods-available/spamhaus_new.conf`.

## CONFIGURAÇÕES

### MS_Methods

    Syntax:  MS_Methods POST,PUT,OPTIONS
    Default: POST,PUT,OPTIONS
    
    The values admitted are the httpd's methods (GET,POST,etc)
    Module verify remote ip address if the method used by the user is present
    in the value passed to this variable. Methods must be comma-separated

### MS_WhiteList

    Syntax:  MS_WhiteList /etc/spamhaus.wl
    Default: no value
   
    Path of whitelist file.
    After you've edit it, you mustn't reload apache. This file will be read only
    when 'data modification time' change. You can add an individual IP address or
    subnets with CIDR.

### MS_UnaffectedDomains

    Syntax:  MS_UnaffectedDomains /etc/spamhaus.unaffected
    Default: no value

    Path of unaffected domains file. Format: www.example.com
    Domains listed in this file are not spamhaus-checked, so e.g. visitors can do 
    an order in an online shop or can contact you without being locked out.
    This can be helpful if you have customers that had bad luck and got a dynamic
    IP which is already in a spam-list.

### MS_DNS

    Syntax:  MS_DNS sbl-xbl.spamhaus.org
    Default: sbl-xbl.spamhaus.org
           
    Name server to use for verify is an ip is blacklisted.
    Using a local rbldnsd instance of sbl-xbl, you can increase query performance

### MS_CacheSize

    Syntax:    MS_CacheSize 4096
    Default:   2048
    Max value: 16384
    
    This directive can manage the number of cache entries.

### MS_CacheValidity

    Syntax:    MS_CacheValidity 86400
    Default:   172800

    This directive defines the number of seconds how long cached MS_DNS entries are valid.

### MS_CustomError

    Syntax:   MS_CustomError "My custom error message"
    Default:  "Access Denied! Your IP address is blacklisted because of malicious behavior in the past."

    A custom error message that allows you to replace default error message with one you create

## Synopsis:

```
<IfModule mod_spamhaus_new.c>

# HTTP methods that should be checked
MS_METHODS POST,PUT,OPTIONS,CONNECT 

# Whitelist of IP addresses and address ranges
MS_WhiteList /etc/spamhaus.wl

# List of domains that should not be affected
MS_UnaffectedDomains /etc/spamhaus.unaffected

# Used DNS-based blackhole list 
#MS_Dns local.rbldnsd.instance.of.sbl-xbl

# IP cache size
MS_CacheSize 4096

# IP cache entry validity
MS_CacheValidity 86400

# Error message that is presented to the user
#MS_CustomError "My custom error message"

</IfModule>
```

## LICENÇA

  This program is free software; you can redistribute it and/or modify
  it under the terms of the GNU General Public License as published by
  the Free Software Foundation; either version 2 of the License, or
  (at your option) any later version.

  This program is distributed in the hope that it will be useful,
  but WITHOUT ANY WARRANTY; without even the implied warranty of
  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  GNU General Public License for more details.

  You should have received a copy of the GNU General Public License
  along with this program; if not, write to the Free Software
  Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301  USA

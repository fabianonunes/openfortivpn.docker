# FortiClient VPN no Linux com token Aladdin eToken Pro

Clone este repositório e construa a imagem:

```bash
docker build -t openfortivpn:latest .
```

Crie o `alias` de execução adicionando o seguinte trecho ao arquivo de inicialização do seu shell (`~/.bashrc` se você usa Bash; `~/.zshrc`, se ZSH):

```bash
# `sudo` é opcional se seu usário pertencer ao grupo `docker`
alias vpn="sudo docker run --rm -ti --network=host --privileged -v ~/.config/openfortivpn:/vpn -v /etc/resolv.conf:/etc/resolv.conf openfortivpn"
```

> **Atenção!** A criação do `alias` não afeta os terminais que já estavam abertos. Portanto, após ajustar
o arquivo de inicialização do shell, abra um outro terminal ou recarregue-o com `source ~/.bashrc` ou `source ~/.zshrc`.

Inicie a VPN:

```bash
vpn
```

E só! 🤓

## Atalhos

* `vpn reconfigure`: abre formulário de configuração da VPN
* `vpn edit`: permite edição manual do arquivo de configuração

## FAQ

### Por que utilizar --network=host?

Para a VPN funcionar, o `openfortivpn` cria uma interface `ppp` e adiciona
rotas IP estáticas à tabela de roteamento do kernel. Por exemplo, ele pode
rotear todas as conexões com destino a 172.16.0.0/12 para a interface `ppp0`.

Se não utilizássemos `--network=host`, essas rotas só funcionariam dentro do
próprio container.

### Por que subir o container com --privileged?

O `openfortivpn` precisa de permissões para criar uma interface `ppp0` via
`pppd` e para acessar o token via USB. Para isso, precisa de acesso ao
`/dev` do host. Além disso, o `pppd` requer a _capability_ `NET_ADMIN` para
funcionar.

Embora seja possível conceder permissões de acesso para cada dispositivo individualmente
via `--device=/dev/ppp --device=/dev/bus/usb/xxx/yyy` e adicionar a _capability_
com `--cap-add=NET_ADMIN`, utilizar --privileged é muito mais simples.

Na prática, rodar o `openfortivpn` dentro de um container com `--privileged`
e `--network=host` é a **mesma coisa** que rodar `sudo openfortivpn` diretamente
no host.

### Por que preciso montar o /etc/resolv.conf dentro do container?

Além de criar uma interface `ppp` e adicionar rotas IP, o `openfortivpn`
também precisa configurar o DNS para que o cliente possa acessar os domínios
da rede sob a VPN.

### Qual a função do utilitário `vpnconfig`?

Nada mais do que um formulário que permite criar um arquivo de configuração do
`openfortivpn` sem passar por toda aquela cerimônia de identificação de
certificados.

Ele detecta automaticamente os certificados elegíveis do token bem como o
hash do certificado do servidor e os guarda nos respectivos atributos do arquivo
de configuração. Caso haja mais de um certificado elegível no token, o usuário
pode escolher qual usar.

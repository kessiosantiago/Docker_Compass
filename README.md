# Docker_Compass

Para realizar as etapas descritas, siga os passos abaixo:

1. **Instalação e Configuração do Docker no Host EC2 Amazon Linux 2:**
   a. Conecte-se à sua instância EC2 Amazon Linux 2 usando SSH.
   b. Atualize os pacotes do sistema:

```bash
sudo yum update -y
```

   c. Instale o Docker:

```bash
sudo amazon-linux-extras install docker
```

   d. Inicie e habilite o serviço Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

   e. Adicione seu usuário ao grupo `docker` para que você possa executar comandos Docker sem precisar de `sudo`:

```bash
sudo usermod -aG docker <seu_usuário>
```

   Lembre-se de sair da sessão SSH e fazer login novamente para que as alterações tenham efeito.

2. **Efetuar Deploy de uma Aplicação Wordpress com Container de Aplicação e RDS MySQL:**

   a. Crie um arquivo `docker-compose.yml` para orquestrar seu ambiente Docker:

```yaml
version: '3'
services:
  wordpress:
    image: wordpress
    environment:
      WORDPRESS_DB_HOST: <endpoint_RDS>
      WORDPRESS_DB_USER: <seu_usuário_DB>
      WORDPRESS_DB_PASSWORD: <sua_senha_DB>
    ports:
      - "80:80"
    volumes:
      - wordpress:/var/www/html
  volumes:
    wordpress

   b. Execute o seguinte comando para iniciar os containers:

```bash
docker-compose up -d
```

3. **Configuração da Utilização do Serviço EFS AWS para Estáticos do Container de Aplicação Wordpress:**

   a. Crie um sistema de arquivos EFS na AWS e anote o ID do sistema de arquivos.

   b. Monte o sistema de arquivos EFS na sua instância EC2. Você pode seguir a documentação da AWS para fazer isso.

   c. Atualize o arquivo `docker-compose.yml` para adicionar um novo serviço que montará o EFS no contêiner:

```yaml
services:
  efs-mount:
    image: busybox
    volumes:
      - efs:/mnt/efs
    command: sh -c "while true; do sleep 1000; done"
volumes:
  efs:
    driver: local
    driver_opts:
      type: nfs
      o: addr=<endpoint_EFS>,nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2
```

   Substitua `<endpoint_EFS>` pelo ponto de montagem do EFS.

   d. Reinicie o Docker Compose:

```bash
docker-compose up -d
```

   Isso montará o EFS no contêiner `efs-mount`.

4. **Configuração do Serviço de Load Balancer AWS para a Aplicação Wordpress:**


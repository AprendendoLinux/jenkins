# Imagem Jenkins com Docker Compose

Esta é uma imagem Docker personalizada baseada na imagem oficial `jenkins/jenkins:lts`, com ferramentas adicionais (`jq`, `curl`, `python3-pip` e `docker-compose`) para suportar pipelines CI/CD que requerem essas dependências. Este Dockerfile é projetado para ser usado em ambientes onde o Jenkins precisa interagir com o Docker Compose e outras ferramentas para automação de builds e deploys.

## Descrição

A imagem estende a imagem oficial do Jenkins LTS, instalando:
- **jq**: Para manipulação de JSON em scripts de automação.
- **curl**: Para realizar requisições HTTP em pipelines.
- **python3-pip**: Para suportar scripts Python e instalação de pacotes via pip.
- **docker-compose**: Para gerenciar aplicações multi-contêiner em pipelines Jenkins.

A imagem alterna entre o usuário `root` para instalar as dependências e o usuário `jenkins` para execução, garantindo segurança e conformidade com as práticas do Jenkins.

## Requisitos

- Docker instalado na máquina host.
- Acesso à internet para baixar pacotes e o binário do `docker-compose` durante a construção da imagem.

## Como usar

### No DockerHub

1. **Puxar a imagem**:
   ```bash
   docker pull <seu-nome-de-usuário>/<nome-da-imagem>
   ```

2. **Executar o contêiner**:
   ```bash
   docker run -d -p 8080:8080 -p 50000:50000 \
     -v jenkins_home:/var/jenkins_home \
     <seu-nome-de-usuário>/<nome-da-imagem>
   ```

   - `-p 8080:8080`: Expõe a interface web do Jenkins.
   - `-p 50000:50000`: Expõe a porta para agentes Jenkins.
   - `-v jenkins_home:/var/jenkins_home`: Persiste os dados do Jenkins em um volume.

3. Acesse o Jenkins em `http://localhost:8080` e siga as instruções para configuração inicial.

### No GitHub

1. **Clonar o repositório**:
   ```bash
   git clone https://github.com/AprendendoLinux/test-ipv6.git
   cd test-ipv6
   ```

2. **Construir a imagem**:
   ```bash
   docker build -t <seu-nome-de-usuário>/<nome-da-imagem> .
   ```

3. **Executar o contêiner**:
   Use o mesmo comando de execução descrito acima.

## Dockerfile

```dockerfile
FROM jenkins/jenkins:lts

USER root
RUN apt-get update && \
    apt-get install -y jq curl python3-pip && \
    apt clean && \
    apt clean all && \
    curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && \
    chmod +x /usr/local/bin/docker-compose

USER jenkins
```

### Explicação do Dockerfile

- **Base**: Usa a imagem `jenkins/jenkins:lts` como base, garantindo a versão mais recente e estável do Jenkins.
- **USER root**: Alterna para o usuário `root` para instalar pacotes e configurar o sistema.
- **Instalação de dependências**:
  - `apt-get update && apt-get install -y jq curl python3-pip`: Instala `jq`, `curl` e `python3-pip`.
  - `apt clean && apt clean all`: Limpa o cache do `apt` para reduzir o tamanho da imagem.
  - `curl -L ...`: Baixa o binário do `docker-compose` (versão 1.29.2) e o instala em `/usr/local/bin/docker-compose`.
  - `chmod +x`: Torna o binário do `docker-compose` executável.
- **USER jenkins**: Reverte para o usuário `jenkins` para garantir que o Jenkins seja executado com privilégios apropriados.

## Uso em Pipelines

Esta imagem é ideal para pipelines Jenkins que requerem:
- Manipulação de JSON com `jq` (ex.: parsing de respostas de APIs).
- Requisições HTTP com `curl`.
- Scripts Python com dependências instaladas via `pip`.
- Gerenciamento de contêineres com `docker-compose` (ex.: orquestração de serviços em ambientes de desenvolvimento ou testes).

Exemplo de pipeline que usa `docker-compose`:
```groovy
pipeline {
    agent { docker { image '<seu-nome-de-usuário>/<nome-da-imagem>' } }
    stages {
        stage('Deploy com Docker Compose') {
            steps {
                sh 'docker-compose up -d'
            }
        }
    }
}
```

## Notas

- **Versão do Docker Compose**: A imagem instala a versão 1.29.2 do `docker-compose`. Para usar outra versão, modifique a URL no `curl` no Dockerfile.
- **Persistência de dados**: Use um volume para `/var/jenkins_home` para evitar perda de configurações do Jenkins.
- **Segurança**: Certifique-se de que o host Docker está configurado corretamente para permitir que o contêiner Jenkins execute comandos `docker-compose`. Isso pode exigir a montagem do socket Docker (`/var/run/docker.sock`) no contêiner.

## Contribuição

Sinta-se à vontade para abrir issues ou pull requests no repositório [AprendendoLinux/test-ipv6](https://github.com/AprendendoLinux/test-ipv6) para sugerir melhorias ou relatar problemas.

## Licença

Este projeto está licenciado sob a [MIT License](LICENSE).

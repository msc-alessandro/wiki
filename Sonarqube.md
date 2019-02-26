## Dependências de instalação

O Sonarqube depende da instalação do Java JDK, essa instalação pode se dar nas versões OpenJDK e do Oracle JDK. A versão instalada neste repositório é

```
JDK >= 8.0
```

## Dependências de execução

O Sonarqube tem a dependência opcional de um servidor de banco de banco de dados, este servidor é necessário para migração e backup dos dados, visto que o banco de dados H2, padrão do Sonar, não permite essas operações.

```
Postgresql
MySQL (Não Recomendado)
Oracle
Microsoft SQL Server
```

## Configurações de Banco de Dados

A receita presente neste repositório realiza a instalação do PostgreSQL Server, além criar e mapear o usuário do sistema, `sonar`, ao banco de dados `sonar_db`. o Sonarqube depende das configurações presentes abaixo para se conectar corretamente ao banco de dados. Essas configurações estão presentes no arquivo `cookbooks/sonarqube/attributes/default.rb`.

```
default['sonarqube']['jdbc']['username'] = 'sonar'
default['sonarqube']['jdbc']['password'] = 'sonar'
default['sonarqube']['jdbc']['url'] = 'jdbc:postgresql://localhost:5432/sonar_db'
```

## Configurações de Firewall

O Sonarqube, por padrão, expoe a porta 9000 para o protocolo TCP. Para alterar esta configuração é necessário alterar os arquivos:


Em `cookbooks/sonarqube/attributes/default.rb`:

```
default['sonarqube']['web']['port'] = 9000
```

Em `cookbooks/firewalld/recipes/default.rb`:

```
firewalld_port '9000/tcp' do
    action :add
    zone   'public'
end
```

## Convergindo o Sonarqube

Para convergir o sonarqube basta executar o comando `rake converge:sonarqube-server`, caso seja necessário escolher o ambiente de em que o sonar vai ser instalado é necessário passar a variável `CHAKE_ENV=<ambiente>`.
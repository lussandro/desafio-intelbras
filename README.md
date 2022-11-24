# Projeto Wordpress + Mysql

## Ambiente ##
* SO  = Ubuntu 22.04

* Docker = 20.10  

* Minikube = v1.28.0

* Kubectl = v1.25

## Descrição ## 

O presente projeto tem por finalidade fazer o deploy de um ambiente funcional com wordpress e mysql.

Este projeto utilizou como cluster de kubernetes o minikube.

No arquivo mysql-secret.yaml temos a senha do banco criptografada usando base64 você pode alterar a mesma a seu gosto. 

A senha padrão é "intelbras2022". Para alterar  a mesma caso esteja utilizando Linux, você pode utilizar o comando: echo -n 'intelbras2022' | base64

o qual vai gerar uma nova senha criptografada em base64 que você pode colar no campo  "MYSQL_ROOT_PASSWORD" no arquivo mysql-secret.yaml.

Foi criado inicialmente para o o projeto um um volume persistente de 1Gb, caso precise de mais espaço, pode alterar o mesmo, apenas mudando o campo: storage no arquivo mysql-volume.yaml

No arquivo mysql.yaml, temos o deploy do nosso banco, de forma padrão, quaisquer alterações referentes à versão do mysql, etc.. devem ser ajustadas nesse arquivo.

No arquivo mysql-services.yaml temos o deploy do serviço do mysql para que o banco possa ser acessado pela aplicação.

No arquivo mysql-database.yaml criamos um configmap para criação automatica do banco de dados do wordpress.

No arquivo wp-volume.yaml é feito a configuração do volume persistente para o nosso wordpress.

No arquivo wordpress.yal é feito o deploy da nossa aplicação do wordpress

Nos arquuivos wp-services.yml e wp-ingress.yaml, temos o deploy do serviço e do ingress para acesso externo ao nosso wordpress.

O arquivo .htaccess possui as configurações para que somente os ips permitidos acessem o painel de configuração do wordpress. 

Existe um Dockerfile para criar uma imagem personalizada do Wordpress com o .htaccess customizado, você deve alterar o arquivo .htaccess colocando o seu ip
e fazer o builda da imgem e hospedar em algum repositório como exemplo abaixo eu hospedei no meu dockerhub.
    docker build -t lussandro/wordpress-intelbras .
    docker push lussandro/wordpress-intelbras

Após fazer o push da imagem para o seu repositório, você deve alterar o arquivo wordpress.yaml mudando o nome da imgagem

## RODANDO O ṔROJETO ## 
1 - Inicialmente precisamos aramazenar a senha do banco nas variaveis de ambiente do kubernetes - 
    kubectl apply -f mysql-secret.yaml
2 - Após criarmos o secret com a senha do banco, vamos criar o volume para o banco de dados mysql
    kubectl apply -f mysql-volume.yaml
3 - Após armazenarmos a senha do banco e criarmos o serviço, já podemos criar o nosso banco propiamente dito:
    kubectl apply -f mysql.yaml
4 - Vamos criar um cnfigmap para criação automatica do banco ao rodarmos o deploy do nosso mysql:
    kubeclt apply -f mysql-database.yaml
5 - Com o nosso banco criado, partimos para a criação do serviço do mysql para termos acesso ao mesmo:
    kubectl apply -f mysql-services.yaml

Feito isso, vamos conferir se está tudo funcionando:

    $ kubectl get all
    NAME              READY   STATUS    RESTARTS   AGE
    pod/mysql-7q6xr   1/1     Running   0          22m

    NAME                    TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
    service/kubernetes      ClusterIP   10.96.0.1     <none>        443/TCP    53m
    service/mysql-service   ClusterIP   10.98.51.95   <none>        3306/TCP   14m

    NAME                    DESIRED   CURRENT   READY   AGE
    replicaset.apps/mysql   1         1         1       22m

Pronto, o nosso banco está configurado, agora vmos a configuração do wordpress

1 - Vamos criar o volume para armazenar os dados do Wordpress:
    kubectl apply -f wp-volume.yaml

2 - Agora vamos fazer o deploy do wordpress, lembre-se de usar a iagem que você buildou alterando a linha 18 do arquivo wordpress.yaml
    
    kubectl apply -f wordpress.yaml

3 - Vamos agora criar o nosso serviço do tipo NodePort para o wordpress?
    
    kubectl apply -f wp-service.yaml

4 - Agora vamos criar o ingress para acesso a nossa pagina:

    kubectl apply -f wp-ingress.yaml

Após criado o nosso ingress vamos aguardar alguns minutos e vamos ver qual o ip que foi exportado para o nosso serviço:

    lussandro@aspire:~/desafio-intelbras$ kubectl get ingress
    NAME         CLASS   HOSTS                 ADDRESS        PORTS   AGE
    wp-ingress   nginx   wp-intelbras.com.br   192.168.49.2   80      79m

Nesse caso o no ip externo é 192.168.49.2
Pronto! Agora é só acessar no nosso navegador o endereço:
    http://192.16849.2/



# Projeto Wordpress + Mysql
O presente projeto tem por finalidade fazer o deploy de um ambiente funcional com wordpress e mysql.
Este projeto utilizou como cluster de kubernetes o minikube.
Como executar o projeto:
No arquivo mysql-secret.yaml temos a senha do banco criptografada usando base64 você pode alterar a mesma a seu gosto. 
A senha padrão é "intelbras2022". Para alterar  a mesma caso esteja utilizando Linux, você pode utilizar o comando: echo -n 'intelbras2022' | base64
o qual vai gerar uma nova senha criptografada em base64 que você pode colar no campo  "MYSQL_ROOT_PASSWORD" no arquivo mysql-secret.yaml.
Foi criado inicialmente para o o projeto um um volume persistente de 1Gb, caso precise de mais espaço, pode alterar o mesmo, apenas mudando o campo: storage no arquivo mysql-volume.yaml
No arquivo mysql.yaml, temos o deploy do nosso banco, de forma padrão, quaisquer alterações referentes à versão do mysql, etc.. devem ser ajustadas nesse arquivo.
No arquivo mysql-services.yaml temos o deploy do serviço do mysql para que o banco possa ser acessado pela aplicação.


RODANDO O ṔROJETO
1 - Inicialmente precisamos aramazenar a senha do banco nas variaveis de ambiente do kubernetes - 
    kubectl apply -f mysql-secret.yaml
2 - Após criarmos o secret com a senha do banco, vamos criar o volume para o banco de dados mysql
    kubectl apply -f mysql-volume.yaml
3 - Após armazenarmos a senha do banco e criarmos o serviço, já podemos criar o nosso banco propiamente dito:
    kubectl apply -f mysql.yaml
4 - Com o nosso banco criado, partimos para a criação do serviço do mysql para termos acesso ao mesmo:
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

Como visto acima, estamos com o mysql rodando, com o serviço do mysql rodando na porta 3306.
Agora é hora de criarmos as tabelas do banco, para isso precisamos do nome do nosso pod.

    $ kubectl get pods
    NAME          READY   STATUS    RESTARTS   AGE
    mysql-7q6xr   1/1     Running   0          26m

Como pudemos reparar acima o nosso pod chama-se mysql-7q6xr
Agora vamos acessar o console do nosso pod via terminal para criarmos o banco:

    kubectl exec -it mysql-7q6xr -- bash

Após acessar o console do nosso pod vamos acessar o console do mysql:

    mysql -u root -p 
     (digite a senha que foi criada no secrets)
Agora criamos no nosso banco chamado wordpress:

    CREATE DATABASE wordpress;
Agora vamos sair do mysql

    exit
E saimos do POD:
    Pressione: Ctrl + d

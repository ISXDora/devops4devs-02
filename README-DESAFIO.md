# Documentação desafio - FilmReviews Inc 
## Deploy local no kubernetes - k3d

### Create cluster

```k3d cluster create meucluster --servers 3 --agents 3 -p "30000:30000@loadbalancer"```

### List Clusters k3d

```k3d cluster list```

![Create cluster kubernetes](https://github.com/ISXDora/github-repo/assets/66398122/3ffe613e-c922-4ee3-b67e-53acc4c9be0d)

### Build and Push Image to DockerHub

```docker build -t isadoraxavierr/review-filmes:v1 --push -f src/Review-Filmes.Web/Dockerfile ./src```

![Build and push image docker](https://github.com/ISXDora/github-repo/assets/66398122/1f9268d9-4adb-47ae-8e92-20d6c990f82c)

### Manifest Kubernetes

- Deployment Postgres and Service 

![Declaração de objetos kubernetes postgres](https://github.com/ISXDora/github-repo/assets/66398122/ae8d4210-abd1-48cb-9c56-d585395d5237)

- Deployment Application Web and Service

![Declaração de objetos kubernetes app web](https://github.com/ISXDora/github-repo/assets/66398122/404b4cb7-bc4a-425a-b4c4-9485f1778d9e)

- Aplicando configurações aos recursos do Kubernetes 

```kubectl apply -f k8s/deployment.yaml```

![Aplicando configurações no kubernetes](https://github.com/ISXDora/github-repo/assets/66398122/3413d484-1f84-4dea-80e5-7bc7a47ab7b3)

- Testando conexão local com o postgres 

```kubectl port-forward service/postgres 5432:5432```

![Captura de tela de 2024-07-07 13-50-27](https://github.com/ISXDora/github-repo/assets/66398122/f7ef3d2e-d6a9-4448-8b71-7199c7b87541)
![Captura de tela de 2024-07-07 13-51-46](https://github.com/ISXDora/github-repo/assets/66398122/10ab5e08-0d39-489b-9f02-ede4239ed257)

- Visualizando os recursos do kubenetes

```kubectl get all```

![Captura de tela de 2024-07-07 13-55-29](https://github.com/ISXDora/github-repo/assets/66398122/93fda75f-b295-4ebd-8087-b4ba3166572e)

- Visualizando aplicação web no browser 

![Captura de tela de 2024-07-07 13-57-37](https://github.com/ISXDora/github-repo/assets/66398122/a04e7338-8ad9-4ffb-a577-31e34b0bc440)

- Testes de resiliência 

  - Para exibir a simulação de ambiente resiliente, vamos testar o comportamento do cluster ao deletar um pod. 

    Listando pods 

    ```kubectl get pods```
    
    ![Captura de tela de 2024-07-07 14-17-24](https://github.com/ISXDora/github-repo/assets/66398122/5eba5d5f-2921-4a92-9e14-1b6b97051112)

    Deletando o pod e observando como ele é recriado quase que imediatamente

    ```kubectl delete pod reviewfilmes-5794f4f47d-ztrft && watch 'kubectl get pods'```

    ![Peek 07-07-2024 14-26](https://github.com/ISXDora/github-repo/assets/66398122/99d227f5-2b0f-414f-a9db-ebbb533b7cd3)

- Testes de escalabilidade

  - Agora nós vamos simular a capacidade de escala da estrutura
  
  Alteramos a declaração do manifesto para aumentar a quantidade de réplicas do pod

  ![Captura de tela de 2024-07-07 14-40-44](https://github.com/ISXDora/github-repo/assets/66398122/a5148bca-9565-4d45-8e03-39c5ee857278)

  Aplicamos as novas configurações do manifesto ao kubernetes 

  ```kubectl apply -f k8s/deployment.yaml```

  ![Captura de tela de 2024-07-07 14-43-32](https://github.com/ISXDora/github-repo/assets/66398122/4ad856fc-5903-4bcb-9757-07fff7bc2c20)

  Em seguida, visualizamos os pods que foram adicionados, percebendo que ele cria apenas a diferença da quantidade declarada no manifesto

  ![Captura de tela de 2024-07-07 14-44-44](https://github.com/ISXDora/github-repo/assets/66398122/da974850-bc2a-4022-875c-51dda68fcc7f)

- Testes de build e deploy com downtime 0

  Nosso teste agora vai consistir em editar a aplicação, gerar um novo build e implantar essa nova imagem em um novo replicaSet com o intuito de 
  realizar a implantação sem retirar a aplicação do ar, enquanto os pods da versão anterior continuam no ar até que sejam todos substituídos pelos pods com a versão atualizada.

  Editando arquivo da aplicação

  ![Captura de tela de 2024-07-07 15-00-47](https://github.com/ISXDora/github-repo/assets/66398122/57b8c9ab-74e2-46aa-9adb-529b62c449d0)

  Gerando novo build and push da imagem docker para o dockerhub 

  ```docker build -t isadoraxavierr/review-filmes:v2 --push -f src/Review-Filmes.Web/Dockerfile ./src```

  ![Captura de tela de 2024-07-07 15-04-15](https://github.com/ISXDora/github-repo/assets/66398122/5af69b06-426d-47af-83bb-d1f78a730248)

  Vamos incluir a nova versão da imagem docker na declaração do manifesto deployments do kubernetes, 
  em seguida aplicar as configurações do manifesto e observar que ele criou um novo replicaset e vai substituindo
  os pods um a um, a medida que cria um novo, exclui o anterior.


  ```kubectl apply -f sk8/deployment.yaml && watch 'kubectl get replicaset,pod'```

  ![Peek 07-07-2024 15-13](https://github.com/ISXDora/github-repo/assets/66398122/db92d69c-2101-4bc4-8371-efac96b8055a)


  ![Captura de tela de 2024-07-07 15-18-36](https://github.com/ISXDora/github-repo/assets/66398122/348a0bc4-13d9-46e1-adbc-77a49d1e81c1)

- Testes de troca de versão de forma declarada

  Alteramos a declaração da imagem no manifesto kubernetes para a versão: v1. 

  ![Alteração do manifesto-imagem docker](https://github.com/ISXDora/docker-postgres/assets/66398122/8023a24c-e5c1-4b32-b484-349c6e41c01c)

  Em seguida aplicamos as configurações com o comando:

  ```kubectl apply -f sk8/deployment.yaml```

  ![Gif troca de versão de forma declarada](Peek%2007-07-2024%2015-36.gif)

- Testes de reversão da versão anterior da aplicação através do recurso rollout do kubernetes

  Verificamos o histórico de versões implantadas com o comando:

  ```kubectl rollout history deployments reviewfilmes```

  E podemos aplicar o comando para reverter a implantação para a última versão com o comando:

  ```kubectl rollout undo deployments reviewfilmes```

  ![Gif rollout kubernetes](Peek%2007-07-2024%2015-55.gif)




    






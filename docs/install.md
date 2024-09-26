Para instalar o Harbor em um cluster Kubernetes, o método mais comum é usar o Helm, que facilita o processo de instalação e gerenciamento de aplicações em Kubernetes. O Harbor é uma plataforma de registro de imagens de container, com suporte a autenticação, gerenciamento de usuários, replicação de registros, escaneamento de vulnerabilidades, entre outros recursos.

Aqui estão os passos detalhados para a instalação do Harbor:
1. Pré-requisitos
- Cluster Kubernetes: Você já deve ter um cluster Kubernetes em execução (como Kind, Minikube, ou um cluster em nuvem como EKS da AWS, GKE do Google Cloud, etc.).
- Helm: Helm deve estar instalado para facilitar a instalação do Harbor. Se ainda não tiver, instale o Helm com o comando:
```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
2. Adicionar o Repositório do Helm para o Harbor
- Primeiro, adicione o repositório Helm do Harbor:
```
helm repo add harbor https://helm.goharbor.io
helm repo update
```
3. Criar um Namespace para o Harbor
- É uma boa prática isolar o Harbor em seu próprio namespace. Crie um namespace chamado harbor:
```
kubectl create namespace harbor
```
4. Configurar os Valores de Instalação (Opcional)
- Você pode customizar a instalação do Harbor criando um arquivo values.yaml. Isso permite configurar opções como o tipo de persistência, certificados TLS, domínios personalizados, etc. Aqui está um exemplo básico de um arquivo values.yaml:

```
externalURL: https://harbor.yourdomain.com

harborAdminPassword: "YourAdminPassword"

persistence:
  persistentVolumeClaim:
    registry:
      size: 100Gi
    chartmuseum:
      size: 10Gi
    jobservice:
      size: 1Gi
    database:
      size: 20Gi
    redis:
      size: 2Gi
    trivy:
      size: 5Gi

ingress:
  enabled: true
  hosts:
    core: harbor.yourdomain.com
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  tls:
    - secretName: harbor-ingress
      hosts:
        - harbor.yourdomain.com
```
Se você não quiser personalizar muito, pode instalar o Harbor com os valores padrões e depois modificar o que for necessário.

5. Instalar o Harbor com o Helm
- Com o repositório adicionado e o namespace criado, agora você pode instalar o Harbor. Se estiver usando o arquivo values.yaml, você pode aplicá-lo diretamente:
```
helm install harbor harbor/harbor --namespace harbor -f values.yaml
```
- Se não quiser usar um arquivo values.yaml personalizado, pode fazer uma instalação simples com o seguinte comando:
```
helm install harbor harbor/harbor --namespace harbor
```
6. Verificar o Status da Instalação
- Para verificar se todos os componentes do Harbor foram instalados corretamente e estão rodando, use o seguinte comando:
```
kubectl get pods -n harbor
```
Certifique-se de que todos os pods estejam no estado Running ou Completed.
7. Configurar o Ingress (Opcional)
- Se você configurou o Harbor para usar um Ingress e apontou para um domínio específico (como harbor.yourdomain.com), você precisa configurar o DNS para apontar para o IP do serviço Ingress (ou LoadBalancer, dependendo do seu setup).

- Para verificar o serviço do Ingress e encontrar o IP externo, execute:
```
kubectl get svc -n harbor
```
Agora, atualize o seu DNS com o IP do serviço Ingress.
8. Acessar o Harbor
- Depois que a instalação estiver completa e o DNS apontado para o IP do Ingress, você poderá acessar o Harbor através do navegador no domínio configurado.
- - O nome de usuário padrão do administrador é admin.
- - A senha do administrador será a que você configurou no arquivo values.yaml ou a padrão que o Helm gerou (se não configurou a senha).

9. TLS com Cert-Manager (Opcional)
- Se você quiser usar HTTPS com cert-manager para provisionar certificados TLS automaticamente, você pode instalar o cert-manager e configurar um emissor para obter certificados de uma CA como o Let's Encrypt.
- Instale o cert-manager no cluster:
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.10.1/cert-manager.yaml
```
- Crie um emissor do Let's Encrypt:
```
    apiVersion: cert-manager.io/v1
    kind: ClusterIssuer
    metadata:
      name: letsencrypt-prod
    spec:
      acme:
        server: https://acme-v02.api.letsencrypt.org/directory
        email: youremail@example.com
        privateKeySecretRef:
          name: letsencrypt-prod
        solvers:
        - http01:
            ingress:
              class: nginx
```
- Com o emissor configurado, o Harbor agora usará TLS com certificados automáticos, conforme configurado no values.yaml.

10. Gerenciar o Harbor
- Depois da instalação, você pode começar a usar o Harbor para:
- - Fazer push/pull de imagens Docker para o registro.
- - Criar e gerenciar projetos e repositórios.
- - Habilitar a replicação entre múltiplos registros Harbor.
- - Configurar políticas de escaneamento de vulnerabilidades com Trivy.
- - Configurar autenticação com LDAP, OAuth, ou SSO, dependendo das necessidades do seu ambiente.

11. Atualizar ou Desinstalar o Harbor
- Para atualizar o Harbor, execute o comando Helm upgrade com o repositório atualizado:
```
helm repo update
helm upgrade harbor harbor/harbor --namespace harbor -f values.yaml
```
- Para desinstalar o Harbor:
```
helm uninstall harbor --namespace harbor
```

Conclusão
- Instalar o Harbor no Kubernetes com o Helm é simples e flexível. O Harbor oferece uma solução poderosa para gerenciar registros de imagens de containers, com autenticação, escaneamento de vulnerabilidades e replicação. Integrando-o com o Kubernetes e usando Helm, você pode facilmente configurar e gerenciar essa plataforma.

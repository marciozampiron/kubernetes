# Lab – Migrando de Kubernetes Ingress para Gateway API

Este repositório contém um laboratório prático para mostrar como migrar de **Kubernetes Ingress** para a **Gateway API** (GatewayClass, Gateway e HTTPRoute).

A ideia é comparar lado a lado:

- Um app exposto com **Ingress** (modelo “clássico”).
- O mesmo app exposto com **Gateway API**, seguindo o padrão moderno recomendado pela comunidade Kubernetes.

---

## Objetivos de aprendizado

Ao final deste lab, você será capaz de:

- Explicar, em alto nível, a diferença entre:
  - **Ingress + Ingress Controller**
  - **Gateway API** (GatewayClass, Gateway, HTTPRoute)
- Entender por que a Gateway API é considerada a evolução da Ingress API para tráfego L7 em Kubernetes.
- Ler e interpretar manifests YAML de Gateway API.
- Migrar um exemplo simples de **Ingress → Gateway API** em um cluster de teste.

---

## Arquitetura do exemplo

Fluxo simplificado:

```text
Cliente HTTP
   │
   ├─ (antes) Ingress  ───▶ Service (ClusterIP) ───▶ Pods (Deployment)
   │
   └─ (depois) Gateway + HTTPRoute ─▶ Service ─▶ Pods
```

- O app é um web server simples rodando em um `Deployment`.
- Um `Service` do tipo `ClusterIP` expõe o app internamente no cluster.
- Primeiro, usamos um **Ingress** para expor esse Service.
- Depois, usamos **Gateway + HTTPRoute** para fazer o mesmo roteamento usando a Gateway API.

---

## Estrutura de arquivos

```text
.
├── README.md
└── manifests
    ├── 01-app-deploy-service.yaml    # Namespace, Deployment e Service do app de exemplo
    ├── 02-ingress.yaml               # Exposição HTTP usando Ingress
    └── 03-gateway-httproute.yaml     # Exposição HTTP usando Gateway API (Gateway + HTTPRoute)
```

---

## Pré-requisitos

- Conhecimentos básicos de Kubernetes:
  - Pods, Deployments, Services, Namespaces.
  - Uso do `kubectl`.
- Um cluster Kubernetes de laboratório:
  - Pode ser kind, k3d, minikube ou outro cluster de testes.
- `kubectl` instalado e apontando para o cluster.
- Um **Ingress Controller** instalado (por exemplo, NGINX, Traefik, etc.), para que o recurso `Ingress` funcione.
- Um **Gateway API controller** instalado e com as CRDs da Gateway API aplicadas:
  - Por exemplo, um controlador baseado em Envoy, NGINX, Cilium, Istio, etc., que implemente `GatewayClass`, `Gateway` e `HTTPRoute`.

> ⚠️ Os manifests de Gateway API deste lab usam `apiVersion: gateway.networking.k8s.io/v1`, que está GA para GatewayClass, Gateway e HTTPRoute. Certifique-se de que seu controlador suporta esta versão.

---

## 1. Deploy do app e Service

Aplique o arquivo com o Namespace, Deployment e Service:

```bash
kubectl apply -f manifests/01-app-deploy-service.yaml
```

Verifique se os Pods e o Service subiram:

```bash
kubectl get ns ingress-gateway-lab
kubectl get deploy,svc -n ingress-gateway-lab
```

Você deve ver um `Deployment` (por exemplo `demo-web`) e um `Service` do tipo `ClusterIP`.

---

## 2. Exposição via Ingress

Aplique o recurso de Ingress:

```bash
kubectl apply -f manifests/02-ingress.yaml
```

Verifique o Ingress e o endereço:

```bash
kubectl get ingress -n ingress-gateway-lab
kubectl describe ingress demo-web-ingress -n ingress-gateway-lab
```

Dependendo do ambiente:

- Em **cloud**: o Ingress Controller pode provisionar um IP público ou um LB.
- Em **kind/minikube/k3d**: você pode precisar de:
  - `port-forward`,
  - addon específico,
  - ou ajustar seu `/etc/hosts` apontando `demo.example.com` para o IP/no localhost.

Teste o acesso:

```bash
curl -H "Host: demo.example.com" http://<IP_DO_INGRESS>/
```

> Em ambiente local, muitas vezes `curl -H "Host: demo.example.com" http://127.0.0.1:<porta>` funciona, dependendo da configuração.

---

## 3. Exposição via Gateway API (Gateway + HTTPRoute)

Agora vamos expor o mesmo Service usando Gateway API.

Aplique os manifests:

```bash
kubectl apply -f manifests/03-gateway-httproute.yaml
```

Liste os recursos da Gateway API:

```bash
kubectl get gatewayclass
kubectl get gateways -n ingress-gateway-lab
kubectl get httproutes -n ingress-gateway-lab
```

Verifique o status do Gateway:

```bash
kubectl describe gateway demo-gateway -n ingress-gateway-lab
```

O controlador de Gateway deve preencher o status com o(s) endereço(s) (IP ou hostname).

Teste o acesso via Gateway API, de forma semelhante ao Ingress:

```bash
curl -H "Host: demo.example.com" http://<IP_DO_GATEWAY>/
```

A resposta deve ser a mesma do app de exemplo.

---

## 4. Migrando Ingress → Gateway API

Fluxo sugerido para migração (simples, como neste lab):

1. Deploy da aplicação e Service (já existente).
2. Ingress funcionando em produção (estado atual).
3. Criar e validar **GatewayClass**, **Gateway** e **HTTPRoute** apontando para o mesmo Service.
4. Validar que o tráfego funciona via Gateway API.
5. Gradualmente:
   - Atualizar DNS / apontadores para usar o endpoint do Gateway.
   - Acompanhar métricas, logs e erros.
6. Depois de estabilizar:
   - Remover o Ingress antigo.

Em ambientes reais, você pode usar ferramentas como **ingress2gateway** para converter manifests de Ingress para Gateway API e seguir uma migração faseada.

---

## 5. Limpeza do ambiente

Quando terminar os testes, você pode remover todos os recursos do lab:

```bash
kubectl delete -f manifests/03-gateway-httproute.yaml
kubectl delete -f manifests/02-ingress.yaml
kubectl delete -f manifests/01-app-deploy-service.yaml
```

Confirme que os recursos sumiram:

```bash
kubectl get all -n ingress-gateway-lab
kubectl get gatewayclass
kubectl get gateways,httproutes -A
```

---

## 6. Boas práticas e dicas

- Comece migrando serviços menos críticos para ganhar confiança.
- Entenda bem o controlador de Gateway API escolhido (features suportadas, limitações, CRDs extras).
- Evite depender de anotações específicas de Ingress:
  - Em vez disso, use os campos e extensões nativas da Gateway API.
- Tenha observabilidade:
  - Logs do controlador,
  - Métricas de tráfego,
  - Eventos (`kubectl get events`) para debugar problemas.
- Mantenha uma documentação clara por um tempo com os dois modelos (Ingress e Gateway API) durante a transição.

---

## 7. Referências

- Documentação oficial de Ingress – Kubernetes.
- Documentação oficial da Gateway API – Kubernetes.
- Blog posts e guias de migração de Ingress para Gateway API.

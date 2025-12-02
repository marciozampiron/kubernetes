# Kubernetes Pod Priority, PriorityClass e Preemption – Lab Prático

Este repositório contém os manifests usados em uma aula prática sobre **Pod Priority**, **PriorityClass** e **Preemption** no Kubernetes.

A ideia é mostrar, na prática, como:
- Definir classes de prioridade.
- Atribuir prioridades distintas para workloads.
- Observar preempção (desalojamento) de Pods de baixa prioridade quando o cluster está sem recursos.

---

## Objetivos de Aprendizado

Ao final do laboratório, o aluno será capaz de:

- Explicar o que é **PriorityClass** e como ela se relaciona com **Pod Priority**.
- Configurar o campo `priorityClassName` em Pods/Deployments.
- Entender o que é **Preemption** e quando ela acontece.
- Diferenciar **PriorityClass** de **QoS** (Guaranteed, Burstable, BestEffort).
- Aplicar boas práticas básicas de priorização de workloads em um cluster Kubernetes.

---

## Pré-requisitos

- Conhecimentos básicos de Kubernetes:
  - Pods, Deployments, Namespaces, Services.
  - Uso do `kubectl`.
- Ambiente de cluster de testes:
  - Pode ser kind, k3d, minikube, ou outro cluster de laboratório.
- `kubectl` instalado e configurado apontando para o cluster.

Sugestão mínima de recursos para enxergar preempção:
- 1 nó com ~2 vCPUs e ~2–4 GiB de RAM.

---

## Estrutura do Repositório

```text
.
├── README.md
└── manifests
    ├── 01-priorityclasses.yaml      # Define diferentes PriorityClasses de exemplo
    ├── 02-pods-priorities.yaml      # Deployments com prioridades alta, média e baixa
    └── 03-preemption-demo.yaml      # Cenário de preempção com pods “enchendo” o nó

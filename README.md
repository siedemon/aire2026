# Lab-1: Basic Agentic Infrastructure

> Розгортання agentgateway + kagent на AKS для курсу LLMOps

---

## Зміст

- [Огляд](#огляд)
- [Рівні складності](#рівні-складності)
- [Передумови](#передумови)
- [Початківці — standalone режим](#початківці--standalone-режим)
- [Файли репозиторію](#файли-репозиторію)

---

## Огляд

Лаба охоплює розгортання [agentgateway](https://agentgateway.dev) — opensource LLM/MCP/A2A gateway написаного на Rust

```
┌─────────────────────────────────────────────────────┐
│                     Клієнт / curl                   │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│              agentgateway  :3000                    │
│   cors · localRateLimit · backendAuth · jwt         │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│           Google Gemini API (gemini-2.5-flash-lite) │
└─────────────────────────────────────────────────────┘
```

---

## Рівні складності

| Рівень | Що розгортається | Де |
|---|---|---|
| **Початківці** | agentgateway binary + Gemini | локально |
---

## Передумови

### Загальні

- Google Gemini API Key → [aistudio.google.com/api-keys](https://aistudio.google.com/api-keys)

## Початківці — standalone режим

### 1. Встановити agentgateway

```bash
curl -sL https://agentgateway.dev/install | bash
agentgateway --version
```

### 2. Отримати API ключ

```bash
export GEMINI_API_KEY="ключ-з-aistudio"
```

### 3. Налаштувати config.yaml

```bash
cat > ~/agentgateway-config.yaml << 'YAML'
# yaml-language-server: $schema=https://agentgateway.dev/schema/config
binds:
  - port: 3000
    listeners:
      - routes:
          - policies:
              cors:
                allowOrigins: ["*"]
                allowHeaders: ["*"]
              localRateLimit:
                - maxTokens: 10
                  tokensPerFill: 10
                  fillInterval: 60s
            backends:
              - ai:
                  name: gemini
                  provider:
                    gemini:
                      model: gemini-2.5-flash-lite
                  policies:
                    backendAuth:
                      key: "$GEMINI_API_KEY"
YAML
```

### 4. Запустити gateway

```bash
agentgateway -f agentgateway-config.yaml
```

Відкрити UI: **http://localhost:15000/ui/**

### 5. Перевірити доступ до LLM

```bash
curl -s http://localhost:3000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-2.5-flash-lite",
    "messages": [{"role": "user", "content": "Hello!"}]
  }' | python3 -m json.tool
```

### Policy довідка

| Policy | Рівень | Що робить |
|---|---|---|
| `cors` | route | дозволяє cross-origin запити |
| `localRateLimit` | route | обмеження запитів (token bucket) |
| `backendAuth` | backend | API ключ до провайдера |
| `jwtAuth` | route | JWT автентифікація |
| `basicAuth` | route | Basic автентифікація |
| `apiKey` | route | API key автентифікація |


    - name: agentgateway-proxy
      namespace: agentgateway-system
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /v1/chat/completions
      backendRefs:
        - name: google
          namespace: agentgateway-system
          group: agentgateway.dev
          kind: AgentgatewayBackend
```

### Тест

# Запит
curl -s http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-2.5-flash-lite",
    "messages": [{"role": "user", "content": "Hello!"}]
  }' | python3 -m json.tool
```

# файли-репозиторію

README.md                  # цей файл
lab1/
├── agentgateway-config.yaml   # standalone конфіг для початківців
└── beginner/
    └── *.png            # скріншоти запуску та результатів відпрацюваня команд

```

## Посилання

- [agentgateway docs](https://agentgateway.dev/docs/)
- [agentgateway Kubernetes mode](https://agentgateway.dev/docs/kubernetes/main/)
- [kagent docs](https://kagent.dev/docs/)
- [Google AI Studio](https://aistudio.google.com/api-keys)
- [AKS release notes](https://github.com/Azure/AKS/releases)
- [Gateway API](https://gateway-api.sigs.k8s.io/)

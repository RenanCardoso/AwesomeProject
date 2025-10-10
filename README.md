# 📱 React Native + CI/CD com GitHub Actions e Firebase App Distribution

Este projeto foi criado com [**React Native**](https://reactnative.dev) via [`@react-native-community/cli`](https://github.com/react-native-community/cli) e inclui uma pipeline **CI/CD completa** para builds automatizados usando **GitHub Actions** e **Firebase App Distribution**.  

---

## 🧭 Índice

1. [Iniciando o projeto](#-iniciando-o-projeto)
2. [Iniciar o Metro Bundler](#1-iniciar-o-metro-bundler)
3. [Executando o app](#2-executando-o-app)
4. [Modificando o app](#-modificando-o-app)
5. [CI/CD com GitHub Actions](#️-cicd-com-github-actions)
   - [Workflow de validação (Testes, Lint e Typescript)](#️-workflow-de-validação-pr-check)
   - [Guia Android](./docs/CI_CD_ANDROID.md)
   - [Guia iOS](./docs/CI_CD_IOS.md)
   - [Criar usuários de teste](./docs/CRIAR_USUARIOS_DE_TESTE_FIREBASE_APP_DISTRIBUTION.md)
6. [Recursos úteis](#-recursos-úteis)

---

## 🚀 Iniciando o projeto

> **Antes de começar**, siga o guia oficial do [React Native - Environment Setup](https://reactnative.dev/docs/environment-setup) até o passo **"Creating a new application"**.

### 1. Iniciar o Metro Bundler

```bash
# npm
npm start

# ou yarn
yarn start
```

### 2. Executando o app

Com o Metro rodando, abra outro terminal e execute:

#### Android
```bash
npm run android
# ou
yarn android
```

#### iOS
```bash
npm run ios
# ou
yarn ios
```

---

## 🛠 Modificando o app

Edite o arquivo `App.tsx` e salve suas mudanças.  
Para recarregar:

- **Android:** pressione `R` duas vezes ou use `Ctrl + M`.
- **iOS:** pressione `Cmd + R`.

---

## ⚙️ CI/CD com GitHub Actions

Este projeto traz uma pipeline completa para **builds automatizados**, **validação de código**, e **entrega de versões de teste via Firebase App Distribution** — tudo dentro do **GitHub Actions**.

## ⚙️ Workflow de validação (PR Check)

Arquivo: `.github/workflows/pr-check.yml`
```yaml
name: PR Check

on:
  push:
    branches:
      - main

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: ".nvmrc"
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check
      - run: npm run test --ci --coverage
```

💡 **Como funciona:**  
- Executa lint, testes e TypeScript a cada push na branch principal.
- Usa Ubuntu para reduzir custos.
- Pode ser expandido para rodar testes E2E com Detox ou Appium.

---

### 🔗 Guias disponíveis:

| Plataforma | Link |
|-------------|------|
| **Android** | [CI/CD Android com GitHub Actions + Firebase](./docs/CI_CD_ANDROID.md) |
| **iOS** | [CI/CD iOS com GitHub Actions + Firebase](./docs/CI_CD_IOS.md) |
| **Firebase (Testadores)** | [Criar usuários e grupos no Firebase App Distribution](./docs/CRIAR_USUARIOS_DE_TESTE_FIREBASE_APP_DISTRIBUTION.md) |
---

## 📚 Recursos úteis

- [React Native Docs](https://reactnative.dev)
- [GitHub Actions Docs](https://docs.github.com/actions)
- [Firebase App Distribution Docs](https://firebase.google.com/docs/app-distribution)
- [wzieba/Firebase Distribution Action](https://github.com/wzieba/Firebase-Distribution-Github-Action)

---

## 🎉 Conclusão

Você agora possui uma estrutura moderna e gratuita de **CI/CD** para React Native:
- PRs passam por testes e lint ✅  
- Builds são geradas e distribuídas automaticamente 📦  
- Testadores recebem e instalam via Firebase App Distribution 📲  

---

Feito com ☕ e ❤️ por [Renan](https://github.com/RenanCardoso).
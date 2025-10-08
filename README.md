# 📱 React Native + CI/CD com GitHub Actions e Firebase App Distribution

Este projeto foi criado com [**React Native**](https://reactnative.dev) via [`@react-native-community/cli`](https://github.com/react-native-community/cli), e inclui uma pipeline **CI/CD completa** para builds automatizados usando **GitHub Actions** e **Firebase App Distribution**.  

---

## 🧭 Índice

1. [Iniciando o projeto](#-iniciando-o-projeto)
2. [Executando o app](#2-executando-o-app)
3. [Modificando o app](#-modificando-o-app)
4. [CI/CD com GitHub Actions](#️-cicd-com-github-actions)
   - [Pré-requisitos](#-pré-requisitos)
   - [Configuração do Firebase](#-configuração-do-firebase)
   - [Configurando o Keystore (Android)](#-configurando-o-keystore-android)
   - [Criando os Secrets no GitHub](#-criando-os-secrets-no-github)
   - [Workflow de validação (PR Check)](#-passo-1--workflow-de-validação-em-pr)
   - [Build de Preview Android](#-passo-2--build-de-preview-automatizado-para-android)
   - [Build de Preview iOS](#-passo-3--build-de-preview-automatizado-para-ios)
   - [Acesso dos testadores](#-passo-4--acesso-dos-testadores)
5. [Recursos úteis](#-recursos-úteis)

---

## 🚀 Iniciando o projeto

> **Atenção:** Antes de começar, siga o guia oficial do [React Native - Environment Setup](https://reactnative.dev/docs/environment-setup) até o passo **"Creating a new application"**.

### 1. Iniciando o Metro Bundler

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

- **Android:** pressione `R` duas vezes ou use o menu do Dev (`Ctrl + M` / `Cmd + M`).
- **iOS:** pressione `Cmd + R`.

---

## ⚙️ CI/CD com GitHub Actions

Com este guia, você vai configurar **builds automatizados**, **testes de qualidade** e **distribuição via Firebase App Distribution**, tudo dentro do GitHub.

---

## 🚀 O que é o Firebase App Distribution?  

O **Firebase App Distribution** permite enviar versões de teste do seu app diretamente para dispositivos de testadores, sem precisar publicar na Google Play ou App Store.  
É ideal para **builds de preview, homologação ou QA**.  

---

### ✅ Pré-requisitos

- **React Native 0.73+**
- Scripts configurados no `package.json`:
```json
{
  "scripts": {
    "test": "jest --passWithNoTests",
    "lint": "eslint src --ext .js,.jsx,.ts,.tsx",
    "type-check": "tsc --noEmit",
    "build:android": "cd android && ./gradlew assembleRelease",
    "build:ios": "cd ios && xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release"
  }
}
```
- Arquivo `.nvmrc` fixando a versão do Node (ex: `18.18.0`).
- Repositório hospedado no GitHub.

---

### 🔧 Configuração do Firebase

#### 1. Criando o projeto
- Vá até o [Console do Firebase](https://console.firebase.google.com);
- Clique em "Criar um novo projeto do Firebase";
- Adicione um app Android;
- Baixe o arquivo `google-services.json` e crie o secret `GOOGLE_SERVICES_JSON_ANDROID` colando exatamente todo o conteúdo do google-services.json (começando com `{` e terminando com `}`).
- Complete o **wizard de configuração**. 

#### 2. Configuração do Gradle

**`android/build.gradle`:**
```gradle
buildscript {
  dependencies {
    classpath("com.google.gms:google-services:4.4.3")
  }
}
```

**`android/app/build.gradle`:**
```gradle
apply plugin: 'com.google.gms.google-services'

dependencies {
  implementation platform('com.google.firebase:firebase-bom:34.2.0')
}
```

#### 3. Gerando os secrets do Firebase

- **FIREBASE_APP_ID_ANDROID** → obtido do `google-services.json` (`mobilesdk_app_id`)
- **FIREBASE_SERVICE_ACCOUNT_JSON** → gere uma chave privada em  
  *Configurações do projeto → Contas de serviço → Gerar nova chave privada* -> Salve o conteúdo do JSON gerado no secret. 

  (Para gerar essa chave, a role mínima necessária é **Firebase App Distribution Admin**).

---

### 🔑 Configurando o Keystore (Android)

```bash
# gerar keystore
keytool -genkey -v -keystore my-upload.keystore -alias upload -keyalg RSA -keysize 2048 -validity 10000

# converter para base64
base64 -w 0 my-upload.keystore
```

Armazene o resultado no secret `ANDROID_KEYSTORE_BASE64`.

---

### 🔐 Criando os Secrets no GitHub

No repositório → `Settings > Secrets and variables > Actions`:

| Nome | Descrição |
|------|------------|
| `GOOGLE_SERVICES_JSON_ANDROID` | Conteúdo do google-services.json (começando com `{` e terminando com `}`). |
| `FIREBASE_APP_ID_ANDROID` | ID do app Firebase |
| `FIREBASE_SERVICE_ACCOUNT_JSON` | JSON completo da conta de serviço |
| `ANDROID_KEYSTORE_BASE64` | Keystore codificado em Base64 |

---

## 🧩 Passo 1 — Workflow de validação em PR

Crie o arquivo `.github/workflows/pr-check.yml`:

```yaml
name: PR Check

on:
  push:
    branches:
      - master

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version-file: ".nvmrc"
      - name: Install deps
        run: npm ci
      - name: Lint
        run: npm run lint
      - name: Type-check
        run: npm run type-check
      - name: Test
        run: npm run test --ci --coverage
```

💡 **Como funciona:**  
- Executa lint, testes e TypeScript a cada push na branch principal.
- Usa Ubuntu para reduzir custos.
- Pode ser expandido para rodar testes E2E com Detox ou Appium.

---

## 📦 Passo 2 — Build de Preview automatizado para Android

Crie `.github/workflows/preview-android.yml`:

```yaml
name: Preview Android

on:
  workflow_run:
    workflows: ["PR Check"]
    types: [completed]

jobs:
  build-preview:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' && github.event.workflow_run.head_branch == 'master' }}
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version-file: ".nvmrc"
          cache: "yarn"

      - name: Install dependencies
        run: yarn install --frozen-lockfile

      - name: Setup JDK 17
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Setup google-services.json
        run: echo '${{ secrets.GOOGLE_SERVICES_JSON_ANDROID }}' > android/app/google-services.json

      - name: Build APK
        run: |
          cd android
          ./gradlew assembleRelease --stacktrace --info

      - name: Upload to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID_ANDROID }}
          serviceCredentialsFileContent: ${{ secrets.FIREBASE_SERVICE_ACCOUNT_JSON }}
          groups: testers
          file: android/app/build/outputs/apk/release/app-release.apk
```

💡 **Como funciona:**
- Dispara **após** o PR Check com sucesso.
- Gera o `.apk` e envia automaticamente ao Firebase App Distribution.
- Testers recebem o app direto no celular.

---

## 🍏 Passo 3 — Build de Preview automatizado para iOS

> 💻 Requer runner **macOS** (plano pago do GitHub ou self-hosted).

Workflow sugerido:
```yaml
name: Preview iOS
on:
  workflow_run:
    workflows: ["PR Check"]
    types: [completed]

jobs:
  build-preview-ios:
    runs-on: macos-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: ".nvmrc"
      - run: yarn install --frozen-lockfile
      - name: Build iOS
        run: |
          cd ios
          xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release -sdk iphoneos
      - name: Upload to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID_IOS }}
          serviceCredentialsFileContent: ${{ secrets.FIREBASE_SERVICE_ACCOUNT_JSON }}
          groups: testers
          file: ios/build/MyApp.ipa
```

---

## 📱 Passo 4 — Acesso dos testadores  

📌 Certifique-se de criar pelo menos um grupo de testador e que os usuários de teste que receberam o aplicativo pelo Firebase estejam devidamente criados e vinculados a este grupo antes de iniciar o primeiro build da pipeline. [Clique aqui para ver o guia de como criar grupo de testadores e usuários de teste no Firebase App Distribution.](./docs/CRIAR_USUARIOS_DE_TESTE_FIREBASE_APP_DISTRIBUTION.md)

1. O testador recebe um e-mail com o convite do Firebase.  
2. Ele aceita o convite e faz login com uma conta Google.  
3. Pode instalar o app via:  
   - **Link do e-mail recebido**  
   - **[Página de testadores do Firebase App Distribution](https://appdistribution.firebase.dev/)**  

> 💡 No Android, será necessário ativar a **instalação de fontes desconhecidas**.  
> No iOS, o testador precisa instalar o **perfil de distribuição** (provisioning profile). 

---

## 📚 Recursos úteis

- [React Native Docs](https://reactnative.dev)
- [GitHub Actions Docs](https://docs.github.com/actions)
- [Firebase App Distribution Docs](https://firebase.google.com/docs/app-distribution)
- [wzieba/Firebase Distribution Action](https://github.com/wzieba/Firebase-Distribution-Github-Action)

---

## 🎉 Conclusão

Você agora tem uma pipeline completa de **CI/CD gratuita** para React Native:
- PRs passam por testes automatizados ✅  
- Builds de preview são geradas e enviadas direto ao Firebase 📲  
- Equipes podem testar novas versões rapidamente 🚀  

---


Feito com ☕ e ❤️ por [Renan](https://github.com/RenanCardoso).
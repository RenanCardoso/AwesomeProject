# 🤖 CI/CD Android com GitHub Actions + Firebase App Distribution

Este guia descreve o fluxo completo de automação para **builds Android** usando **GitHub Actions** e **Firebase App Distribution**.

---

## ⚙️ Pré-requisitos

- **React Native 0.73+**
- Scripts no `package.json`:
```json
{
  "scripts": {
    "test": "jest --passWithNoTests",
    "lint": "eslint src --ext .js,.jsx,.ts,.tsx",
    "type-check": "tsc --noEmit",
    "build:android": "cd android && ./gradlew assembleRelease"
  }
}
```
- Firebase configurado
- Repositório hospedado no GitHub

---

## 🔧 Configuração do Firebase

1. Vá até o [Console do Firebase](https://console.firebase.google.com)
2. Crie um novo projeto
3. Adicione um **app Android**
4. Baixe o arquivo `google-services.json`
5. Crie o secret no GitHub:  
   `GOOGLE_SERVICES_JSON_ANDROID` → cole o conteúdo inteiro do arquivo JSON

No arquivo **`android/build.gradle`**:
```gradle
buildscript {
  dependencies {
    classpath("com.google.gms:google-services:4.4.3")
  }
}
```

No arquivo **`android/app/build.gradle`**:
```gradle
apply plugin: 'com.google.gms.google-services'

dependencies {
  implementation platform('com.google.firebase:firebase-bom:34.2.0')
}
```

---

## 🔐 Secrets necessários no GitHub

| Nome | Descrição |
|------|------------|
| `FIREBASE_APP_ID_ANDROID` | ID do app Android (do google-services.json) |
| `FIREBASE_SERVICE_ACCOUNT_JSON` | JSON completo da conta de serviço |
| `GOOGLE_SERVICES_JSON_ANDROID` | Conteúdo do google-services.json |
| `ANDROID_KEYSTORE_BASE64` | Keystore codificado em Base64 |

---

#### 3. Gerando os secrets do Firebase

- **FIREBASE_APP_ID_ANDROID** → obtido do `google-services.json` (`mobilesdk_app_id`)
- **FIREBASE_SERVICE_ACCOUNT_JSON** → gere uma chave privada em  
  *Configurações do projeto → Contas de serviço → Gerar nova chave privada* -> Salve o conteúdo do JSON gerado no secret. 

  (Para gerar essa chave, a role mínima necessária é **Firebase App Distribution Admin**).

---

## 🔑 Gerar Keystore

```bash
keytool -genkey -v -keystore my-upload.keystore -alias upload -keyalg RSA -keysize 2048 -validity 10000
base64 -w 0 my-upload.keystore
```

Crie o secret `ANDROID_KEYSTORE_BASE64` com o conteúdo base64.

---

## 📦 Workflow de build e distribuição

Arquivo: `.github/workflows/preview-android.yml`

```yaml
name: Preview Android

on:
  workflow_run:
    workflows: ["PR Check"]
    types: [completed]

jobs:
  build-preview:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' && github.event.workflow_run.head_branch == 'main' }}
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
## 📱 Acesso dos testadores  

📌 Certifique-se de criar pelo menos um grupo de testador e que os usuários de teste que receberam o aplicativo pelo Firebase estejam devidamente criados e vinculados a este grupo antes de iniciar o primeiro build da pipeline. [Clique aqui para ver o guia de como criar grupo de testadores e usuários de teste no Firebase App Distribution.](./docs/CRIAR_USUARIOS_DE_TESTE_FIREBASE_APP_DISTRIBUTION.md)

1. O testador recebe um e-mail com o convite do Firebase.  
2. Ele aceita o convite e faz login com uma conta Google.  
3. Pode instalar o app via:  
   - **Link do e-mail recebido**  
   - **[Página de testadores do Firebase App Distribution](https://appdistribution.firebase.dev/)**  

> 💡 No Android, será necessário ativar a **instalação de fontes desconhecidas**.  

---
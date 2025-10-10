# 🍏 CI/CD iOS com GitHub Actions + Firebase App Distribution

Este guia cobre o processo de **build automatizado para iOS** via **GitHub Actions** e distribuição com **Firebase App Distribution**.

---

## ⚙️ Pré-requisitos

- Conta Apple Developer ativa (A Apple exige a conta paga para permitir assinaturas de distribuição, que é o que o Firebase App Distribution precisa)
- Projeto iOS configurado no Firebase
- Runner **macOS** disponível (GitHub hospedado ou self-hosted)
- Scripts no `package.json`:
```json
{
  "scripts": {
    "build:ios": "cd ios && xcodebuild -workspace AwesomeProject.xcworkspace -scheme AwesomeProject -configuration Release"
  }
}
```

---

## 🔐 Secrets necessários no GitHub

| Nome | Descrição |
|------|------------|
| `FIREBASE_APP_ID_IOS` | ID do app iOS no Firebase |
| `FIREBASE_SERVICE_ACCOUNT_JSON` | JSON completo da conta de serviço |
| `IOS_GOOGLE_SERVICE_INFO_BASE64` | JSON completo do arquivo de configuração do Firebase |
| `APPLE_P12_BASE64` | Certificado `.p12` em Base64 |
| `APPLE_P12_PASSWORD` | Senha do certificado |
| `APPLE_PROVISION_PROFILE_BASE64` | Provisioning Profile em Base64 |

---

### 1️⃣ Gerando os secrets do Firebase 

- **FIREBASE_APP_ID_IOS** → é criado automaticamente quando você cadastra o app iOS no Firebase.
  
  Passo a passo para obter:

  ### ⚠️ Importante:
  Na segunda etapa do assistente "Adicionar o Firebase ao seu aplicativo da Apple", não salve o arquivo `GoogleService-Info.plist` diretamente no repositório, mesmo que o Firebase oriente a colocá-lo na raiz do projeto Xcode.
  Esse arquivo contém informações sensíveis de configuração do seu app e não deve ser exposto publicamente.
  Em vez disso, vamos criar um secret no GitHub para armazenar seu conteúdo com segurança e utilizá-lo no processo de build automatizado.

  1. Vá no Firebase Console
  2. Selecione o seu projeto.
  3. Clique em Configurações do projeto (⚙️ → "Configurações do projeto").
  4. Vá até a aba Seus apps.
  5. Lá você verá todos os apps registrados no projeto: iOS, Android, Web.
  6. Clique no app iOS.
  7. O App ID aparece no campo ID do Aplicativo (geralmente no formato):
  ```bash
  1:1234567890:ios:abcdef1234567890
  ```

  👉 Esse valor é o que você deve usar como `FIREBASE_APP_ID_IOS` no GitHub Actions.

  📌 Observação:
  Se você ainda não cadastrou um app iOS, vai precisar criar um:
  - Clicar em Adicionar app → iOS
  - Inserir o Bundle ID do seu projeto Xcode (ex.: com.seuprojeto.app)
  - O Firebase vai gerar o App ID iOS na hora.
  - Na quarta etapa do assistente "Adicionar código de inicialização", pode ser que você esteja está usando a estrutura mais moderna do React Native (com herança de RCTAppDelegate), nesse caso, a inicialização do Firebase deve ser feita dentro do método didFinishLaunchingWithOptions, antes de chamar o super.

    ✅ Aqui está como você deve modificar seu AppDelegate.mm:

    1. Importe o Firebase no topo do arquivo:
    ```bash
    #import <Firebase.h>
    ```

    2. Adicione a linha [FIRApp configure]; dentro do método didFinishLaunchingWithOptions, antes do super:  
    ```bash
    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
    {
      [FIRApp configure]; // <- Inicializa o Firebase

      self.moduleName = @"AwesomeProject";
      self.initialProps = @{};

      return [super application:application didFinishLaunchingWithOptions:launchOptions];
    }
    ```  

- **IOS_GOOGLE_SERVICE_INFO_BASE64** → 
  1. Após baixar o arquivo `GoogleService-Info.plist` da quarta etapa e concluir todas as etapas do assistente de adição do Firebase ao aplicativo, execute o comando no terminal:
  ```bash
  base64 -i ios/AwesomeProject/GoogleService-Info.plist
  ```

  E copie o conteúdo gerado.

  📌 Observação: Ajuste o comando com o caminho relativo onde você salvou o arquivo. 
  
  (Exemplo: o caminho usado aqui é ios/AwesomeProject/GoogleService-Info.plist)

  Lembre-se, `NUNCA` suba este arquivo ao controle de versão.

  2. Criar o secret no GitHub

  - Vá até o repositório no GitHub.
  - Acesse Settings > Secrets and variables > Actions.
  - Clique em "New repository secret".
  - Nomeie como IOS_GOOGLE_SERVICE_INFO_BASE64.
  - Cole o conteúdo base64.

- **FIREBASE_SERVICE_ACCOUNT_JSON** → gere uma chave privada em  
  *Configurações do projeto → Contas de serviço → Gerar nova chave privada* -> Salve o conteúdo do JSON gerado no secret. 

  (Para gerar essa chave, a role mínima necessária é **Firebase App Distribution Admin**).
---

## 🔑 Certificados e Perfis

### 2️⃣ Gerar `.p12` (certificado)
1. Abra o **Keychain Access**
2. Encontre o certificado "Apple Distribution"
3. Clique com o botão direito → **Export**
4. Salve como `cert.p12` e defina uma senha (`APPLE_P12_PASSWORD`)
5. Converta em base64:
```bash
base64 -i cert.p12 | pbcopy
```
Crie o secret `APPLE_P12_BASE64`.

### 3️⃣ Provisioning Profile (.mobileprovision)
1. Vá ao [Apple Developer](https://developer.apple.com/account/resources/profiles/list)
2. Crie um novo perfil **App Store** ou **Ad Hoc**
3. Selecione App ID e certificado
4. Baixe o arquivo `.mobileprovision`
5. Converta para base64:
```bash
base64 -i MyProfile.mobileprovision | pbcopy
```
Crie o secret `APPLE_PROVISION_PROFILE_BASE64`.

---

## ⚙️ Workflow de build iOS

Arquivo: `.github/workflows/preview-ios.yml`

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
          cache: "yarn"

      - run: yarn install --frozen-lockfile

      - run: gem install bundler

      - uses: apple-actions/import-codesign-certs@v2
        with:
          p12-file-base64: ${{ secrets.APPLE_P12_BASE64 }}
          p12-password: ${{ secrets.APPLE_P12_PASSWORD }}
          mobileprovision-base64: ${{ secrets.APPLE_PROVISION_PROFILE_BASE64 }}

      - name: Decode GoogleService-Info.plist
        run: |
          echo "${{ secrets.IOS_GOOGLE_SERVICE_INFO_BASE64 }}" | base64 --decode > ios/GoogleService-Info.plist

      - run: |
          cd ios
          pod install --repo-update

      - run: |
          cd ios
          xcodebuild \
            -workspace MyApp.xcworkspace \
            -scheme MyApp \
            -configuration Release \
            -sdk iphoneos \
            -archivePath build/MyApp.xcarchive archive

          xcodebuild \
            -exportArchive \
            -archivePath build/MyApp.xcarchive \
            -exportOptionsPlist ExportOptions.plist \
            -exportPath build/

      - uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID_IOS }}
          serviceCredentialsFileContent: ${{ secrets.FIREBASE_SERVICE_ACCOUNT_JSON }}
          groups: testers
          file: ios/build/MyApp.ipa
```
💡 **Como funciona:**
- Dispara **após** o PR Check com sucesso.
- Gera o `.ipa` e envia automaticamente ao Firebase App Distribution.
- Testers recebem o app direto no celular.

---
## 📱 Acesso dos testadores  

📌 Certifique-se de criar pelo menos um grupo de testador e que os usuários de teste que receberam o aplicativo pelo Firebase estejam devidamente criados e vinculados a este grupo antes de iniciar o primeiro build da pipeline. [Clique aqui para ver o guia de como criar grupo de testadores e usuários de teste no Firebase App Distribution.](./docs/CRIAR_USUARIOS_DE_TESTE_FIREBASE_APP_DISTRIBUTION.md)

1. O testador recebe um e-mail com o convite do Firebase.  
2. Ele aceita o convite e faz login com uma conta Google.  
3. Pode instalar o app via:  
   - **Link do e-mail recebido**  
   - **[Página de testadores do Firebase App Distribution](https://appdistribution.firebase.dev/)**  

> 💡 No iOS, o testador precisa instalar o **perfil de distribuição** (provisioning profile). 

---
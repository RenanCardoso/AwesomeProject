# 📲 Como criar grupo de testadores e usuários de teste no Firebase App Distribution  

Este guia explica **como adicionar testadores** no **Firebase App Distribution** — o serviço do Firebase que permite distribuir **builds de preview** do seu app (Android e iOS) para testadores com apenas um link.  

---

## ⚙️ 1. Pré-requisitos  

Antes de criar os testadores, garanta que você:  
- Possui um projeto configurado no [Firebase Console](https://console.firebase.google.com/)  
- Já fez o **upload de uma build** (APK, AAB ou IPA) no App Distribution  
- Tem o acesso ao painel **App Distribution** dentro do Firebase  

---

## 👥 2. Acessando o App Distribution  

1. Vá até o [Firebase Console](https://console.firebase.google.com/).  
2. Selecione o projeto do seu app.  
3. No menu lateral, clique em **“Distribuição de aplicativos (App Distribution)”**.  
4. Escolha o app (Android ou iOS) que deseja distribuir.  

---

## ➕ 3. Criar usuários de teste (testadores)  

Você pode adicionar testadores de **duas formas**: manualmente pela interface ou via **Firebase CLI**.  

### 🖱️ Método 1 — Via interface (manual)

1. Dentro do painel **App Distribution**, clique em **“Testadores e grupos”**.  
2. Dentro da seção  **“Testadores e grupos”**, clique em **“Adicionar grupo”**.  
3. Informe o nome do grupo (ex: QA, Devs, Stakeholders) e clique em **“Salvar”**.  
4. Clique em **“Adicionar um testador”**.  
5. Informe o e-mail do testador e clique em **“Adicionar um testador”**.  

Os testadores receberão um **convite por e-mail** para acessar o app.  
Eles precisarão aceitar o convite e **entrar com uma conta Google**.  

---

### 💻 Método 2 — Via Firebase CLI

Você também pode adicionar testadores via terminal, útil em pipelines de CI/CD.  

1. Instale o Firebase CLI:
   ```bash
   npm install -g firebase-tools
   ```

2. Faça login:
   ```bash
   firebase login
   ```

3. Adicione testadores:
   ```bash
   firebase appdistribution:testers:add tester1@example.com tester2@example.com --app <APP_ID>
   ```

   > 🔍 O **APP_ID** pode ser encontrado no Firebase Console, em *Configurações do Projeto → Seus Apps*.

4. Para adicionar um **grupo de testadores**:
   ```bash
   firebase appdistribution:groups:create qa-team --app <APP_ID>
   firebase appdistribution:testers:add tester1@example.com tester2@example.com --group qa-team --app <APP_ID>
   ```

---

## 📱 4. Acesso dos testadores  

1. O testador recebe um e-mail com o convite do Firebase.  
2. Ele aceita o convite e faz login com uma conta Google.  
3. Pode instalar o app via:  
   - **Link do e-mail recebido**  
   - **[Página de testadores do Firebase App Distribution](https://appdistribution.firebase.dev/)**  

> 💡 No Android, será necessário ativar a **instalação de fontes desconhecidas**.  
> No iOS, o testador precisa instalar o **perfil de distribuição** (provisioning profile).  

---

## ✅ Conclusão  

Agora você sabe como:  
- Criar testadores no App Distribution (manual ou via CLI)  
- Organizar grupos (QA, Devs, etc.)  

Assim, seus **testadores recebem builds automaticamente**, mantendo o processo de testes ágil e integrado ao CI/CD.  

---

📚 **Documentação oficial:**  
🔗 [Firebase App Distribution - Adicionar testadores](https://firebase.google.com/docs/app-distribution/manage-testers?hl=pt-br)
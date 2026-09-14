# Avaliação de Desempenho UGM — colocar no ar

Este pacote tem 1 arquivo só: `index.html`. Ele é a ferramenta inteira. Siga os passos abaixo, nesta ordem. Nenhum deles exige linha de comando — tudo é feito pelo navegador (github.com, console.firebase.google.com, portal.azure.com).

Repositório de destino: **https://github.com/angelicabernardes/Avaliacao-de-Desempenho-ugm**

---

## 1. Subir o arquivo no GitHub

1. Abra o repositório no navegador.
2. Clique em **Add file → Upload files**.
3. Arraste o `index.html` deste pacote para lá.
4. Escreva uma mensagem de commit (ex: "primeira versão") e clique em **Commit changes**.

> O repositório precisa ser **público** para o GitHub Pages funcionar no plano gratuito. Isso é seguro aqui: o arquivo só contém o código da ferramenta (HTML/CSS/JS), nenhum dado de avaliação — as avaliações em si ficam guardadas no Firebase (passo 2), não no GitHub.

## 2. Ativar o GitHub Pages

1. No repositório, vá em **Settings → Pages**.
2. Em "Build and deployment" → "Source", escolha **Deploy from a branch**.
3. Em "Branch", escolha **main** (ou a branch onde subiu o arquivo) e a pasta **/ (root)**. Salve.
4. Espere 1–2 minutos. O endereço final vai aparecer no topo dessa mesma página — deve ser:

   **`https://angelicabernardes.github.io/Avaliacao-de-Desempenho-ugm/`**

   Guarde essa URL — ela é usada nos passos 3 e 4.

Neste ponto a ferramenta já abre nesse endereço, mas ainda em "modo de demonstração" (sem login exigido, sem salvar de verdade) — é assim que ela avisa que os passos 3 e 4 ainda faltam.

## 3. Criar o banco de dados (Firebase)

1. Acesse **console.firebase.google.com** e entre com uma conta Google (pode ser a da UGM, se tiver Google Workspace, ou uma pessoal — só quem administra o Firebase depois importa).
2. **Add project** → dê um nome (ex: "avaliacao-ugm") → pode desligar o Google Analytics (não precisa) → Create project.
3. No menu lateral, **Build → Firestore Database → Create database**. Escolha uma região perto do Brasil (ex: `southamerica-east1`) → **Start in production mode**.
4. Depois que o banco for criado, vá na aba **Rules** (dentro de Firestore Database) e substitua o conteúdo por:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read, write: if request.auth != null
           && request.auth.token.email is string
           && request.auth.token.email.matches('.*@umgrauemeio[.]com$');
       }
     }
   }
   ```

   Isso é o que garante, no próprio banco de dados (não só na tela), que só quem loga com e-mail `@umgrauemeio.com` consegue ler ou gravar qualquer avaliação. Clique em **Publish**.

5. Menu lateral → **Build → Authentication → Get started**. Na aba **Sign-in method**, clique em **Add new provider → Microsoft**.
6. Ative o provedor. O Firebase vai pedir dois dados do Azure AD: **Application (client) ID** e **Application (client) secret**.
   - O Client ID já temos: `c4767279-af52-4824-8614-112154e25327`
   - O **Client secret** ainda não existe — peça ao administrador do Azure para criar um: no portal.azure.com, abrir o app "Avaliação de Desempenho (GC)" → **Certificados e segredos** → **Novo segredo do cliente** → copiar o **Valor** (não o ID do segredo) assim que for gerado, porque ele só aparece uma vez.
7. Cole o Client ID e o Client secret nos campos do Firebase e salve.
8. Depois de salvar, o Firebase mostra um **redirect URI** (algo como `https://SEU-PROJETO.firebaseapp.com/__/auth/handler`). Copie esse endereço.
9. Volte ao **portal.azure.com** → o app "Avaliação de Desempenho (GC)" → **Authentication** → **Add a URI** → cole esse endereço do Firebase ali (pode manter ou apagar o antigo, que apontava para o Claude — não é mais usado). Salve.

   Esse redirect URI do Firebase é fixo — diferente do que tínhamos antes, ele nunca muda, então esse cadastro só precisa ser feito uma vez.

10. Ainda no Firebase, vá em **Authentication → Settings → Authorized domains** e clique em **Add domain**. Adicione:

    **`angelicabernardes.github.io`**

    (sem `https://` e sem a barra final). Sem isso, o login trava com o erro "domínio não autorizado".

## 4. Colar as chaves do Firebase no arquivo

1. No Firebase, vá em **Configurações do projeto** (ícone de engrenagem, ao lado de "Project Overview") → aba **General** → role até "Your apps" → clique no ícone **</>** (Web) para criar um app da Web → dê um nome (ex: "avaliacao-web") → **Register app**. Não precisa marcar a opção de Firebase Hosting.
2. Ele mostra um bloco `firebaseConfig = {...}` com 6 valores (`apiKey`, `authDomain`, `projectId`, `storageBucket`, `messagingSenderId`, `appId`).
3. Volte no `index.html` (pode editar direto pelo GitHub: abra o arquivo no repositório → ícone de lápis "Edit") e procure por `FIREBASE_CONFIG` perto do topo do arquivo. Substitua os seis `"COLE_AQUI_..."` pelos valores correspondentes do Firebase.
4. Commit direto na branch main. O GitHub Pages atualiza sozinho em 1–2 minutos.

## Pronto

A partir daqui, `https://angelicabernardes.github.io/Avaliacao-de-Desempenho-ugm/` é o link definitivo para mandar aos gestores. Só quem entrar com uma conta `@umgrauemeio.com` (validada pela própria Microsoft, no tenant da empresa) consegue abrir a ferramenta e ler ou gravar qualquer avaliação — reforçado tanto na tela de login quanto nas regras do banco de dados.

### Se algo der errado
- **Login trava dizendo "domínio não autorizado"** → falta o passo 3.10 (Authorized domains no Firebase).
- **Login funciona mas nada salva / fica girando** → confira se coleu certinho as 6 chaves (passo 4) e se as Regras do Firestore (passo 3.4) foram publicadas.
- **"Erro ao entrar" genérico** → confira se o Client secret (passo 3.6) foi colado certo no Firebase — é comum copiar o ID do segredo em vez do Valor por engano.

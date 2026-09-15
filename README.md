# Reposição de Lojas — Uniso

Ferramenta interna para gestão de pedidos de reposição e novidades entre lojas: importação de planilhas, painel por prazo, atribuição de colaboradores em 2 fases (Separação e Organização), impressão de ordens (Separação, Versão para Entrega) e painel de tempos/desempenho.

Este pacote contém tudo o que você precisa para publicar o app na internet de forma permanente, com um banco de dados compartilhado entre todos que acessarem o link (Firebase) e o código-fonte versionado no GitHub.

**Tempo estimado para o primeiro deploy: 20 a 30 minutos.**

---

## 1. Visão geral do que vamos montar

| Peça | Serviço | Para quê |
|---|---|---|
| Site (HTML/CSS/JS) | Firebase Hosting | Onde o app fica acessível por uma URL (ex.: `reposicao-uniso.web.app`) |
| Banco de dados | Firebase Firestore | Onde ficam salvos os pedidos e colaboradores, compartilhados entre todos |
| Código-fonte | GitHub (repositório privado) | Histórico de versões, backup, facilita futuras alterações |

Você **não precisa saber programar** para seguir este guia — são comandos para copiar e colar, um de cada vez.

---

## 2. Pré-requisitos (instalar uma vez só)

1. **Conta Google** (para criar o projeto Firebase) — provavelmente você já tem.
2. **Conta GitHub** — crie gratuitamente em [github.com](https://github.com) se ainda não tiver.
3. **Node.js** instalado no seu computador — baixe em [nodejs.org](https://nodejs.org) (escolha a versão "LTS"). Isso é necessário para instalar a ferramenta de linha de comando do Firebase.
4. **Git** instalado — no Windows, baixe em [git-scm.com](https://git-scm.com); no Mac, já vem instalado (ou instale via `xcode-select --install` no Terminal).

Para confirmar que Node e Git estão instalados, abra o Terminal (Mac) ou o Prompt de Comando/PowerShell (Windows) e digite:

```
node -v
git --version
```

Se aparecer um número de versão em cada um, está tudo certo.

---

## 3. Criar o projeto no Firebase (do zero)

1. Acesse [console.firebase.google.com](https://console.firebase.google.com) e faça login com sua conta Google.
2. Clique em **"Criar um projeto"** (ou "Add project").
3. Dê um nome, por exemplo: `reposicao-uniso`. O Firebase vai gerar um ID único do projeto (ex.: `reposicao-uniso-a1b2c`) — anote esse ID, você vai precisar dele.
4. Na etapa seguinte, pode **desativar o Google Analytics** (não é necessário para este app) e clicar em "Criar projeto".
5. Aguarde a criação (leva cerca de 1 minuto) e clique em "Continuar".

### 3.1. Ativar o Firestore (banco de dados)

1. No menu à esquerda do console, clique em **"Firestore Database"** (em "Compilação"/"Build").
2. Clique em **"Criar banco de dados"**.
3. Escolha o modo **"Produção"** (não use "Teste" — as regras de segurança deste pacote já cuidam disso).
4. Escolha a localização do servidor — recomendado: `southamerica-east1 (São Paulo)`, para menor latência no Brasil. **Atenção: essa escolha não pode ser alterada depois.**
5. Clique em "Ativar". O banco começa vazio — os dados serão criados automaticamente conforme o app for usado.

### 3.2. Registrar o app Web e pegar as credenciais

1. Na página inicial do projeto (ícone de casa no menu), clique no ícone **`</>`** ("Adicionar app" > Web).
2. Dê um apelido, ex.: `reposicao-uniso-web`. **Não** marque a opção de configurar Firebase Hosting nesta tela (faremos isso pela linha de comando).
3. Clique em "Registrar app".
4. Você verá um bloco de código parecido com este — **copie esses valores**, você vai colá-los no arquivo `public/index.html`:

```js
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "reposicao-uniso-a1b2c.firebaseapp.com",
  projectId: "reposicao-uniso-a1b2c",
  storageBucket: "reposicao-uniso-a1b2c.appspot.com",
  messagingSenderId: "123456789012",
  appId: "1:123456789012:web:abcdef123456"
};
```

5. Clique em "Continuar no console" (pode pular as etapas de instalar SDK via npm, já está tudo pronto neste pacote).

> **Sobre segurança:** essas chaves (`apiKey` etc.) **não são segredas** — elas identificam o projeto, não dão acesso irrestrito. Quem realmente controla o que pode ser lido/gravado são as **regras do Firestore** (arquivo `firestore.rules`, já incluído neste pacote, configurado para permitir acesso por link, conforme combinamos).

---

## 4. Configurar o app com suas credenciais

1. Abra o arquivo `public/index.html` num editor de texto (o Bloco de Notas serve, mas recomendo o [VS Code](https://code.visualstudio.com), gratuito).
2. Procure por este trecho, perto do topo do arquivo (use Ctrl+F / Cmd+F para achar `firebaseConfig`):

```js
var firebaseConfig = {
  apiKey: "COLE_AQUI_SUA_API_KEY",
  authDomain: "SEU-PROJETO.firebaseapp.com",
  projectId: "SEU-PROJETO",
  storageBucket: "SEU-PROJETO.appspot.com",
  messagingSenderId: "SEU_SENDER_ID",
  appId: "SEU_APP_ID"
};
```

3. Substitua cada valor pelos que você copiou no passo 3.2. Salve o arquivo.
4. Abra também o arquivo `.firebaserc` (na raiz do pacote) e troque `SEU-PROJECT-ID-AQUI` pelo ID do seu projeto (o mesmo `projectId` de cima).

---

## 5. Instalar a ferramenta de linha de comando do Firebase

No Terminal / Prompt de Comando, digite:

```
npm install -g firebase-tools
```

Depois, faça login (vai abrir uma janela do navegador para autorizar):

```
firebase login
```

---

## 6. Publicar no Firebase Hosting (primeira vez)

1. Pelo Terminal, entre na pasta deste pacote (ajuste o caminho para onde você salvou):

```
cd caminho/para/reposicao-uniso
```

2. Publique as regras do Firestore:

```
firebase deploy --only firestore:rules
```

3. Publique o site:

```
firebase deploy --only hosting
```

4. Ao final, o Firebase mostra a URL pública do seu app, algo como:

```
Hosting URL: https://reposicao-uniso-a1b2c.web.app
```

Essa é a URL que você vai compartilhar com as lojas. Pronto — o app está no ar, com banco de dados compartilhado real.

### Para publicar atualizações futuras

Sempre que eu (ou você) alterar o `public/index.html`, para colocar a nova versão no ar:

```
cd caminho/para/reposicao-uniso
firebase deploy --only hosting
```

---

## 7. Subir o código para o GitHub (repositório privado)

1. Acesse [github.com/new](https://github.com/new) para criar um repositório.
2. Nome sugerido: `reposicao-uniso`.
3. Marque a opção **"Private"** (conforme combinamos).
4. **Não** marque "Add a README" (já temos um neste pacote).
5. Clique em "Create repository". O GitHub vai mostrar comandos — use estes, pelo Terminal, dentro da pasta do pacote:

```
cd caminho/para/reposicao-uniso
git init
git add .
git commit -m "Versão inicial do app de Reposição de Lojas Uniso"
git branch -M main
git remote add origin https://github.com/SEU-USUARIO/reposicao-uniso.git
git push -u origin main
```

(Troque `SEU-USUARIO` pelo seu nome de usuário do GitHub — o próprio GitHub mostra o comando exato com seu usuário já preenchido, na tela após criar o repositório.)

### Para enviar atualizações futuras ao GitHub

```
git add .
git commit -m "descreva o que mudou"
git push
```

---

## 8. Testando se ficou tudo certo

1. Abra a URL do Hosting (passo 6.4) em um navegador.
2. Na aba "1. Painel", deve aparecer uma faixa **verde/azul** dizendo algo como "🔗 Pedidos salvos e visíveis para todos que abrirem este link". Se aparecer uma faixa **amarela de aviso** dizendo que não foi possível conectar ao Firebase, revise o passo 4 (credenciais) e o passo 6.2 (regras publicadas).
3. Importe uma planilha de teste, gere um pedido, e abra o mesmo link em outro navegador (ou peça para outra pessoa abrir) — o pedido deve aparecer lá também, confirmando que o banco está compartilhado de verdade.

---

## 9. Estrutura deste pacote

```
reposicao-uniso/
├── public/
│   ├── index.html        ← o app completo (HTML+CSS+JS em um arquivo só)
│   └── assets/
│       └── uniso-logo.png
├── firebase.json          ← configuração do Firebase Hosting
├── firestore.rules        ← regras de segurança do banco de dados
├── .firebaserc             ← identifica qual projeto Firebase usar
├── .gitignore
└── README.md               ← este arquivo
```

---

## 10. Perguntas frequentes

**O Firebase vai me cobrar alguma coisa?**
Não, para este volume de uso. O plano gratuito (Spark) do Firebase inclui 10 GB de hospedagem, 360 MB/dia de tráfego e 1 GiB de armazenamento no Firestore com 50 mil leituras e 20 mil gravações por dia — muito acima do que uma operação interna de algumas lojas gera. Se um dia isso mudar, o Firebase avisa antes de qualquer cobrança (é preciso ativar manualmente um plano pago).

**Posso trocar o domínio (ex.: `reposicao.uniso.com.br`) no lugar de `.web.app`?**
Sim — no console do Firebase, em Hosting, há a opção "Adicionar domínio personalizado". Exige acesso ao DNS do domínio da empresa.

**Quero restringir o acesso só para quem estiver logado com e-mail da empresa. Dá para fazer depois?**
Sim, mas é uma etapa adicional (Firebase Authentication + tela de login + ajuste das regras do Firestore). Se quiser seguir esse caminho no futuro, me avise que preparo essa evolução.

**Perdi minhas credenciais do Firebase, e agora?**
Elas continuam visíveis a qualquer momento em: Console do Firebase > Configurações do projeto > Seus aplicativos > app Web.

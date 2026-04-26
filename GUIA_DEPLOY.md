# 🚀 Guia de Deploy no GitHub Pages

Como colocar seu dashboard online gratuitamente.

## 📋 O que você vai precisar

- Conta no GitHub (gratuita)
- Os arquivos do produto

## 🎯 Passo a passo

### 1. Criar repositório no GitHub

1. Acesse [github.com](https://github.com) e faça login
2. Clique em **"New repository"** (botão verde)
3. Preencha:
   - **Repository name:** `controle-financeiro` (ou outro nome)
   - **Public** (deve ser público)
   - ✅ Marque "Add a README file"
4. Clique em **"Create repository"**

### 2. Fazer upload dos arquivos

**Opção A - Via interface web (mais fácil):**

1. No repositório criado, clique em **"Add file" → "Upload files"**
2. Arraste os arquivos:
   - `index.html`
   - `planilha_financeira_familiar.xlsx`
   - `README.md`
3. Escreva uma mensagem: `Primeiro commit - Dashboard financeiro`
4. Clique em **"Commit changes"**

**Opção B - Via Git (se você já usa):**

```bash
git init
git add .
git commit -m "Primeiro commit - Dashboard financeiro"
git branch -M main
git remote add origin https://github.com/SEU-USUARIO/controle-financeiro.git
git push -u origin main
```

### 3. Ativar GitHub Pages

1. No repositório, clique em **"Settings"** (⚙️)
2. No menu lateral, clique em **"Pages"**
3. Em "Source":
   - Branch: **main**
   - Folder: **/ (root)**
4. Clique em **"Save"**

### 4. Aguardar deploy

- Aguarde 1-2 minutos
- Volte na página "Pages"
- Vai aparecer o link: `https://seu-usuario.github.io/controle-financeiro`

### 5. Testar

1. Acesse o link gerado
2. Cole o ID da planilha de teste
3. Verifique se carrega os dados

## 🎁 Link para entregar aos clientes

Seu link final será:
```
https://SEU-USUARIO.github.io/controle-financeiro
```

Você pode entregar:
1. **Link direto** — cliente acessa e configura
2. **Link com ID** — `?id=ABC123` (mais fácil pro cliente)

## 🔄 Como atualizar o dashboard

Se você fizer melhorias:

1. Edite os arquivos localmente
2. Volte no GitHub → seu repositório
3. Clique no arquivo → **Edit** (✏️)
4. Cole o novo código
5. **Commit changes**

Ou pelo Git:
```bash
git add .
git commit -m "Atualização do dashboard"
git push
```

Aguarde 1-2 minutos e o site atualiza automaticamente!

## 💡 Dicas extras

### Domínio personalizado (opcional)

Se quiser um link mais profissional:

1. Compre um domínio (ex: Registro.br — R$ 40/ano)
2. No GitHub Pages, adicione o domínio em "Custom domain"
3. Configure DNS do domínio para apontar pro GitHub

### Limites

- **Largura de banda:** 100GB/mês (mais que suficiente)
- **Armazenamento:** 1GB (index.html tem ~100KB, tranquilo)
- **Builds:** 10 por hora

Para um produto vendido a R$ 10-20, o GitHub Pages gratuito atende perfeitamente!

## ❓ Problemas comuns

**"404 Not Found"**
→ Aguarde 2-3 minutos após ativar Pages

**"Site não carrega"**
→ Verifique se o repositório é Public

**"Arquivos não aparecem"**
→ Certifique-se que o index.html está na raiz (não em pasta)

---

🎉 Pronto! Seu dashboard está no ar e você pode vender pra quantas pessoas quiser sem custo adicional.

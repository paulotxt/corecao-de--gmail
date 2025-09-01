# 📧 Email Normalizer

Um pequeno script em **JavaScript** que corrige e padroniza endereços de e-mail digitados de forma incorreta ou informal.  

Ele tenta “consertar” variações comuns como:  

- `joao (at) gmail ponto com` → `joao@gmail.com`  
- `@gmail.com` → `usuario@gmail.com`  
- `paulo-gmail.com` → `paulo@gmail.com`  
- `ana@outlook.com` → `ana@gmail.com` *(sempre força o domínio para Gmail)*  

---

## ✨ Funcionalidades
- Remove espaços desnecessários e caracteres inválidos.  
- Substitui termos como **“arroba”**, **“at”**, **“ponto”** por `@` e `.`.  
- Garante que todo e-mail tenha o formato `usuario@gmail.com`.  
- Adiciona cada novo e-mail corrigido em uma tabela HTML.  
- Numera automaticamente os e-mails cadastrados.  

---

## 📂 Estrutura do Código
- `contador`: controla a numeração da tabela.  
- `corrigirEmail(email)`: aplica as correções no texto digitado.  
- `cadastrarEmail()`: pega o valor do input, corrige, insere na tabela e limpa o campo.  

---

## 🛠 Como usar
1. Copie o script para dentro de uma página HTML.  
2. Crie um campo de entrada e uma tabela, por exemplo:  

```html
<input type="text" id="emailInput" placeholder="Digite seu e-mail">
<button onclick="cadastrarEmail()">Cadastrar</button>

<table id="tabelaEmails" border="1">
  <thead>
    <tr>
      <th>#</th>
      <th>Email Corrigido</th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

# Visão geral

* Mantém um contador (`contador`) para numerar cada e-mail cadastrado.
* Tem duas funções:

  * `corrigirEmail(email)`: “conserta” o texto digitado para virar um e-mail no formato `usuario@gmail.com`.
  * `cadastrarEmail()`: pega o que está no `<input id="emailInput">`, corrige, adiciona uma linha na tabela `<table id="tabelaEmails">` (no `<tbody>`) e limpa o campo.

# Quando e como o código é executado

1. Ao carregar a página, o navegador lê a tag `<script>`, cria a variável global `contador = 0` e **declara** (mas não executa) as funções `corrigirEmail` e `cadastrarEmail`.
2. Nada acontece até alguém chamar `cadastrarEmail()` (geralmente por um botão com `onclick="cadastrarEmail()"` ou um listener).
3. Quando `cadastrarEmail()` é chamada, ela:

   * Lê o valor do input `#emailInput`.
   * Se estiver vazio, mostra `alert("Digite um e-mail!")` e para por aí.
   * Caso contrário, chama `corrigirEmail(valor)` para normalizar o texto.
   * Incrementa `contador` e insere uma nova `<tr>` no `<tbody>` de `#tabelaEmails` com o número e o e-mail corrigido.
   * Limpa o campo de texto.

# `corrigirEmail(email)` passo a passo

Suponha que `email` seja o texto cru digitado pelo usuário.

1. **Normalização básica**

   ```js
   email = email.trim().toLowerCase();
   ```

   * `trim()` remove espaços do começo/fim.
   * `toLowerCase()` deixa tudo minúsculo.

2. **Substituições comuns**

   ```js
   email = email
     .replace(/\s+/g, "")                // remove todos os espaços internos
     .replace(/\(at\)|\[at\]| at /g, "@")// troca "(at)", "[at]" e " at " por "@"
     .replace(/arroba/gi, "@")           // troca "arroba" por "@"
     .replace(/ponto/gi, ".");           // troca "ponto" por "."
   ```

   Objetivo: transformar jeitos “humanos” de escrever e-mail (ex.: “joao (at) gmail ponto com”) num formato de e-mail real.

3. **Caso o texto comece com “@”**

   ```js
   if (email.startsWith("@")) {
     email = "usuario" + email;
   }
   ```

   * Se a pessoa digitou só “@gmail.com”, vira “[usuario@gmail.com](mailto:usuario@gmail.com)”.

4. **Ajustes dependendo de ter ou não “@”**

   * **Sem “@” mas contendo “gmail”** (ex.: `paulo-gmail.com`):

     ```js
     let usuario = email.split("gmail")[0].replace(/[-_.]$/, "");
     email = usuario + "@gmail.com";
     ```

     * Pega tudo antes de “gmail”, remove um traço/ponto/sublinhado se estiver **no final** e monta `usuario@gmail.com`.

   * **Sem “@” e sem “gmail”** (caso genérico):

     ```js
     email = email + "@gmail.com";
     ```

     * Apenas acrescenta `@gmail.com`.

   * **Se já tem “@”**:

     ```js
     let usuario = email.split("@")[0];
     email = usuario + "@gmail.com";
     ```

     * Mantém **só a parte antes do @** e **força o domínio para `gmail.com`** (troca qualquer domínio por Gmail).

5. **Retorno**

   ```js
   return email;
   ```

# `cadastrarEmail()` passo a passo

1. Lê o valor do input:

   ```js
   let email = document.getElementById("emailInput").value;
   ```
2. Se estiver vazio, alerta e sai:

   ```js
   if (!email) return alert("Digite um e-mail!");
   ```
3. Corrige/normaliza:

   ```js
   let emailCorrigido = corrigirEmail(email);
   ```
4. Incrementa o contador e monta a linha da tabela:

   ```js
   contador++;
   let tabela = document.querySelector("#tabelaEmails tbody");
   let linha = `<tr><td>${contador}</td><td>${emailCorrigido}</td></tr>`;
   tabela.innerHTML += linha;
   ```
5. Limpa o input:

   ```js
   document.getElementById("emailInput").value = "";
   ```

# Exemplos rápidos

* Entrada: `Paulo (at) Gmail ponto com` → `paulo@gmail.com`
* Entrada: `@gmail.com` → `usuario@gmail.com`
* Entrada: `paulo-gmail.com` → `paulo@gmail.com`
* Entrada: `ana.silva@outlook.com` → **vira** `ana.silva@gmail.com` (o código **sempre** troca o domínio para Gmail)

# Observações importantes (comportamentos/pegadinhas)

* **Força o domínio**: qualquer e-mail com `@` vira `...@gmail.com`. Se você quiser preservar o domínio real (outlook, yahoo, empresa etc.), esse comportamento precisa ser alterado.
* **“ponto”/“arroba” em nomes**: a substituição acontece no texto todo; se alguém digitar um usuário que realmente contenha “ponto” como palavra, ele vira “.”.
* **Sensibilidade a maiúsculas**: o padrão `/(at)/g` não tem o modificador `i` (case-insensitive) — pega “ at ”, mas não “ AT ”.
* **`innerHTML +=`**: reinterpreta HTML a cada inclusão e pode abrir margem para XSS se um usuário conseguir injetar tags. Mais seguro é criar elementos com `document.createElement` e usar `textContent`.
* **Validação limitada**: não há checagem se o resultado é um e-mail válido segundo um padrão. É um “conserto heurístico”.
* **Contador é volátil**: volta a 0 quando recarrega a página.

Se quiser, eu adapto o código para: preservar o domínio, endurecer a validação, aceitar variações (“AT”, “arroba ” com acento, etc.) e inserir as linhas de forma segura sem `innerHTML +=`.

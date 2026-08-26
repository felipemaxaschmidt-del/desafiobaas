# 📋 Relatório Técnico de Análise e Resolução de Bugs — Nexus dos Heróis

##Desenvolvimento de Sistemas 301
##Felipe Maximiliano Alves Schmidt Nathália Luiza Eble
## 📖 Resumo Executivo

Este relatório apresenta a documentação detalhada da análise, causa-raiz e resolução técnica dos **8 bugs** mapeados no projeto **Nexus dos Heróis**. O projeto consiste em uma aplicação web desenvolvida em **Next.js** integrada ao **Firebase (Authentication e Firestore)**.

O objetivo deste relatório é estruturar individualmente cada falha encontrada (do BUG 01 ao BUG 08), explicitando a causa do problema, a lógica adotada para a correção e os conceitos técnicos associados a cada etapa.

---

## 🐞 Análise e Resolução dos Bugs

---

### BUG 01 — Login Silencia Erros

* **Arquivo:** `src/app/(auth)/login/page.tsx`
* **Linha aproximada:** ~46
* **Conceito ensinado:** Tratamento de erros com `try/catch`, tratamento de exceções do Firebase Auth e feedback de interface (UX).


<img width="447" height="129" alt="image" src="https://github.com/user-attachments/assets/82dd072f-a54c-4e35-a09f-0d8fdff3ca0b" />







#### 🔴 Causa do Problema
O código continha um bloco `catch` genérico sem captura da variável de erro e completamente vazio (`// catch vazio — erro engolido`). Dessa forma, qualquer falha durante a autenticação (senha incorreta, usuário inexistente, erros de rede) era ignorada pelo sistema sem exibir mensagens ou alertas para o usuário final.

#### 🟢 Lógica de Resolução
O bloco `catch` foi atualizado para receber o parâmetro de erro (`err`), verificar a instância da exceção e filtrar as mensagens de falha retornadas pelo Firebase Auth (como `invalid-credential`, `wrong-password` e `user-not-found`). Com base no tipo de erro, o estado local `setErro` é atualizado com uma mensagem clara e inteligível.

#### 💻 Código Bugado
```ts
} catch {
  // catch vazio — erro engolido
}
```

#### ✅ Correção Aplicada
```ts
} catch (err) {
  const msg = err instanceof Error ? err.message : "Erro desconhecido";
  if (msg.includes("invalid-credential") || msg.includes("wrong-password")) {
    setErro("E-mail ou senha incorretos.");
  } else if (msg.includes("user-not-found")) {
    setErro("Nenhuma conta encontrada com este e-mail.");
  } else {
    setErro("Erro ao entrar. Tente novamente.");
  }
}
```
<img width="722" height="232" alt="image" src="https://github.com/user-attachments/assets/e356354d-74ce-4c3c-86de-19fcb6322f50" />
---

### BUG 02 — Middleware com Condição Invertida

<img width="558" height="132" alt="image" src="https://github.com/user-attachments/assets/ea02eb21-db30-4f43-9d84-b8c5a2f9d72a" />


* **Arquivo:** `middleware.ts`
* **Linha aproximada:** ~28
* **Conceito ensinado:** Next.js Middleware, proteção de rotas e operador lógico de negação (`!`).

#### 🔴 Causa do Problema
A estrutura condicional do middleware utilizava `if (token)`, redirecionando o usuário para a página `/login` caso um token de sessão **existisse**. Essa negação ausente provocava a inversão da lógica de proteção: usuários autenticados eram bloqueados e redirecionados para a tela de login, enquanto usuários não autenticados tinham acesso livre.

#### 🟢 Lógica de Resolução
Adicionou-se o operador lógico de negação (`!token`). Dessa forma, a validação passou a verificar a ausência do token de sessão para realizar o redirecionamento correto à página de login.

#### 💻 Código Bugado
```ts
if (token) {  // ← deveria ser !token
  return NextResponse.redirect(new URL("/login", request.url));
}
```

#### ✅ Correção Aplicada
```ts
if (!token) {
  return NextResponse.redirect(new URL("/login", request.url));
}
```
<img width="647" height="205" alt="image" src="https://github.com/user-attachments/assets/b0b07011-860f-4476-97f8-6efa3827f86f" />

---

### BUG 03 — Confirmação de Senha Compara com Nome

<img width="463" height="106" alt="image" src="https://github.com/user-attachments/assets/f4b2e19f-c7c9-4f02-83e6-bbf9d82812fa" />


* **Arquivo:** `src/app/(auth)/cadastro/page.tsx`
* **Linha aproximada:** ~30
* **Conceito ensinado:** Validação de formulários no cliente e atenção à nomeação e comparação de variáveis.

#### 🔴 Causa do Problema
Durante a validação do formulário de cadastro, a verificação de igualdade da senha comparava a variável `senha` com a variável `nome` (`senha !== nome`), em vez de comparar com a variável correspondente à confirmação de senha (`confirmarSenha`).

#### 🟢 Lógica de Resolução
Substituiu-se o uso da variável `nome` pela variável de estado `confirmarSenha`, garantindo que o formulário certifique a correspondência correta entre a senha e a sua confirmação.

#### 💻 Código Bugado
```ts
if (senha !== nome) {  // ← variável errada!
```

#### ✅ Correção Aplicada
```ts
if (senha !== confirmarSenha) {
```
<img width="631" height="135" alt="image" src="https://github.com/user-attachments/assets/bee70b66-6f43-447a-af15-1a2b54b552b3" />

---

### BUG 04 — Query Sem Filtro de userId

<img width="485" height="110" alt="image" src="https://github.com/user-attachments/assets/91645393-13c8-4e7f-b587-697354a49b3c" />


* **Arquivo:** `src/services/personagens.ts`
* **Linha aproximada:** ~29
* **Conceito ensinado:** Queries no Firestore com `where()`, isolamento e separação de dados por usuário.

#### 🔴 Causa do Problema
A consulta de busca no Firestore trazia a coleção inteira de personagens (`query(collection(db, "personagens"))`) sem aplicar restrições de propriedade. Isso resultava no vazamento de dados, permitindo que qualquer usuário visualizasse os personagens cadastrados por outros usuários.

#### 🟢 Lógica de Resolução
Importou-se o método `where` do pacote `firebase/firestore` e adicionou-se o filtro `where("userId", "==", uid)` na construção da query, restringindo a busca apenas aos documentos associados ao ID do usuário autenticado.

#### 💻 Código Bugado
```ts
const q = query(collection(db, "personagens"));
```

#### ✅ Correção Aplicada
```ts
import { where } from "firebase/firestore";
// ...
const q = query(
  collection(db, "personagens"),
  where("userId", "==", uid)
);
```
<img width="701" height="166" alt="image" src="https://github.com/user-attachments/assets/adeeb691-9c16-451d-8e34-01083c78104e" />

---

### BUG 05 — Nome de Coleção Errado no Create

<img width="599" height="127" alt="image" src="https://github.com/user-attachments/assets/c96d407f-cf0e-4598-9826-f0878f10f042" />


* **Arquivo:** `src/services/personagens.ts`
* **Linha aproximada:** ~52
* **Conceito ensinado:** Nomes de coleções no Firestore e padronização/consistência de nomenclatura.

#### 🔴 Causa do Problema
Na função de inserção de novos registros, a coleção foi especificada no singular (`"personagem"`), enquanto a consulta de leitura e listagem utilizava o nome no plural (`"personagens"`). Essa divergência fazia com que os novos registros fossem gravados em uma coleção paralela e nunca fossem exibidos na interface.

#### 🟢 Lógica de Resolução
Corrigiu-se a string passada para a função `collection()` de `"personagem"` para `"personagens"`, uniformizando o nome da coleção no Firestore.

#### 💻 Código Bugado
```ts
const ref = await addDoc(collection(db, "personagem"), { ... });
//                                       ↑ singular — errado!
```

#### ✅ Correção Aplicada
```ts
const ref = await addDoc(collection(db, "personagens"), { ... });
//                                       ↑ plural — correto
```
<img width="698" height="237" alt="image" src="https://github.com/user-attachments/assets/86098dc2-ee36-4f98-a59e-36c6e2511cc3" />

---

### BUG 06 — setDoc Apaga o Documento Inteiro

<img width="676" height="113" alt="image" src="https://github.com/user-attachments/assets/b7a58509-cac9-45ba-81e3-08f827285f7b" />



* **Arquivo:** `src/services/personagens.ts`
* **Linha aproximada:** ~82
* **Conceito ensinado:** Operações de escrita no Firestore: diferença entre `setDoc` (substituição total) e `updateDoc` (atualização parcial).

#### 🔴 Causa do Problema
O método `setDoc` foi utilizado sem o parâmetro de fusão (`{ merge: true }`). Ao tentar atualizar um único campo/slot do item, o Firestore substituía o documento inteiro pelo novo objeto `{ [slot]: itemId }`, apagando todas as outras propriedades previamente salvas no personagem.

#### 🟢 Lógica de Resolução
Substituiu-se a chamada do método `setDoc` pela função `updateDoc`. O `updateDoc` atualiza estritamente os campos especificados no objeto sem sobrescrever os demais dados existentes no documento.

#### 💻 Código Bugado
```ts
await setDoc(doc(db, "personagens", personagemId), { [slot]: itemId });
```

#### ✅ Correção Aplicada
```ts
await updateDoc(doc(db, "personagens", personagemId), { [slot]: itemId });
```
<img width="717" height="87" alt="image" src="https://github.com/user-attachments/assets/6f79149a-084c-433a-a7f5-7bda1b7ee15b" />

---

### BUG 07 — Deletar Usa Índice Como ID

<img width="604" height="126" alt="image" src="https://github.com/user-attachments/assets/c2447ce6-21e0-40f4-b6c8-8cb43e42e90b" />


* **Arquivo:** `src/services/personagens.ts`
* **Linha aproximada:** ~100
* **Conceito ensinado:** Identificadores únicos (IDs) no Firestore vs. índice posicional em listas de frontend.

#### 🔴 Causa do Problema
A função de exclusão passava o índice numérico da lista local (`0, 1, 2...`) convertido para string como o identificador do documento Firestore. Como os IDs gerados pelo Firestore são identificadores únicos alfanuméricos, a tentativa de deleção falhava por apontar para um ID inexistente no banco.

#### 🟢 Lógica de Resolução
Ajustou-se a referência do documento passado à função `doc()` para utilizar a propriedade `.id` do objeto `personagem` (`personagem.id`), referenciando o ID real gravado no Firestore.

#### 💻 Código Bugado
```ts
await deleteDoc(doc(db, "personagens", String(indice)));
//                                      ↑ índice 0, 1, 2... não é o ID!
```

#### ✅ Correção Aplicada
```ts
await deleteDoc(doc(db, "personagens", personagem.id));
```
<img width="613" height="84" alt="image" src="https://github.com/user-attachments/assets/6b118bd3-9e3d-4a0d-9cee-ad25660c27be" />

---

### BUG 08 — Security Rules Abertas

<img width="293" height="145" alt="image" src="https://github.com/user-attachments/assets/776f5ab1-5d6f-4059-836d-9f75be7c94bc" />


* **Arquivo:** `firestore.rules`
* **Conceito ensinado:** Firebase Security Rules, autenticação vs. autorização e validação de contexto via `resource.data` e `request.resource.data`.

#### 🔴 Causa do Problema
As regras de segurança estavam configuradas para permitir leitura e escrita públicas e irrestritas em qualquer documento (`allow read, write: if true;`), o que representava uma vulnerabilidade grave de segurança da informação.

#### 🟢 Lógica de Resolução
As regras de segurança foram reestruturadas para a coleção `personagens` aplicando controle de acesso baseado em autenticação e propriedade de dados:
* **Leitura (`read`), Atualização (`update`) e Deleção (`delete`):** Válidas somente se o usuário estiver autenticado (`request.auth != null`) e seu UID for idêntico ao `userId` gravado no documento existente (`resource.data.userId`).
* **Criação (`create`):** Válida somente se o usuário estiver autenticado e o `userId` contido no documento a ser inserido corresponder ao UID da requisição (`request.resource.data.userId`).

#### 💻 Código Bugado
```
match /{document=**} {
  allow read, write: if true;
}
```

#### ✅ Correção Aplicada
```
match /personagens/{personagemId} {
  allow read: if request.auth != null &&
              request.auth.uid == resource.data.userId;
  allow create: if request.auth != null &&
                request.auth.uid == request.resource.data.userId;
  allow update, delete: if request.auth != null &&
                        request.auth.uid == resource.data.userId;
}
```
<img width="739" height="237" alt="image" src="https://github.com/user-attachments/assets/baf77ed8-bb82-44f0-9093-1f55250b2003" />

---

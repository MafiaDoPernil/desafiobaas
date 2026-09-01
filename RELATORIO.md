## BUG #01 — bug do login

## Localização
Arquivo e Localização: src/app/(auth)/login/page.tsx (linha ~46)

### O que estava acontecendo
o inserir credenciais incorretas (e-mail ou senha), a interface permanecia travada no estado de carregamento ("Entrando...") sem apresentar nenhum alerta ao usuário.

### Por que acontecia
O bloco try/catch responsável pela requisição de login capturava a exceção, mas possuía o corpo do catch vazio, silenciando o erro retornado pelo Firebase Auth e impedindo a atualização do estado da interface.

### Como corrigi
 catch {}

corrigido:
catch {
       setErro('E-mail ou senha inválidos.')
    } 

## BUG #02 — rota desprotegida

## Localização
Arquivo e Localização: middleware.ts (linha ~28)

### O que estava acontecendo
Usuários não autenticados conseguiam acessar áreas restritas da aplicação, enquanto usuários com login ativo eram redirecionados para a tela de autenticação.

### Por que acontecia
O Middleware do Next.js continha uma condição lógica invertida: redirecionava a requisição para /login justamente quando o token de sessão estava presente.

### Como corrigi

if (token) {
  return NextResponse.redirect(new URL("/login", request.url));
}

corrigido:
if (!token) {
      return NextResponse.redirect(new URL("/login", request.url));
    }

## BUG #03 — bug da senha

## Localização
Arquivo e Localização: src/app/(auth)/cadastro/page.tsx (linha ~30)

### O que estava acontecendo
O formulário de cadastro apresentava comportamento errático ao validar as senhas, bloqueando ou permitindo registros incorretamente.

### Por que acontecia
Durante a checagem dos campos do formulário, a variável senha estava sendo comparada incorretamente com o campo nome em vez da variável confirmarSenha.

### Como corrigi
if (token) {
  return NextResponse.redirect(new URL("/login", request.url));
}

corrigido:
if (!token) {
      return NextResponse.redirect(new URL("/login", request.url));
    }

## BUG #04 — bug dos personagem

## Localização
Arquivo e Localização: src/services/personagens.ts (linha ~29)

### O que estava acontecendo
Ao carregar a lista de heróis, a tela exibia personagens criados por todos os usuários da base de dados, em vez de isolar os do usuário logado.

### Por que acontecia
Importação e aplicação do método where na query do Firestore, filtrando os documentos onde o campo userId é estritamente igual ao uid da sessão ativa.

### Como corrigi

const q = query(collection(db, "personagens"));

corrigido:
import { where } from "firebase/firestore";
....
....
const q = query(
  collection(db, "personagens"),
  where("userId", "==", uid)
);

## BUG #05 — personagem some

## Localização
Arquivo e Localização: src/services/personagens.ts (linha ~52)

### O que estava acontecendo
O personagem era criado com sucesso, mas desaparecia imediatamente após a atualização do dashboard.

### Por que acontecia
A função addDoc gravava os dados na coleção "personagem" (no singular), enquanto o restante da aplicação realizava a leitura na coleção "personagens" (no plural).

### Como corrigi

const q = query(collection(db, "personagens"));

corrigido:
import { where } from "firebase/firestore";
....
....
const q = query(
  collection(db, "personagens"),
  where("userId", "==", uid)
);



## BUG #06 — bug do documento

## Localização
Arquivo e Localização: src/services/personagens.ts (linha ~82)

### O que estava acontecendo
Ao equipar um novo item no herói, todos os demais atributos e equipamentos salvos anteriormente eram apagados.

### Por que acontecia
Uso do método setDoc sem a flag de mesclagem (merge: true), resultando na substituição integral do documento pelo novo objeto enviado contendo apenas o slot atualizado.

### Como corrigi

 await setDoc(doc(db, "personagens", personagemId), { [slot]: itemId });

 corrigido:
 await updateDoc(doc(db, "personagens", personagemId), { [slot]: itemId });


## BUG #07 — bug deletar

## Localização
Arquivo e Localização: src/services/personagens.ts (linha ~100)

### O que estava acontecendo
A ação de deletar excluía um registro incorreto ou falhava silenciosamente sem remover o item selecionado.

### Por que acontecia
O método deleteDoc estava recebendo a posição do array na interface (indice) convertida em string, em vez do identificador único (id) gerado pelo Firestore para o documento.

### Como corrigi

 await deleteDoc(doc(db, "personagens", String(indice)));

 corrigido:
await deleteDoc(doc(db, "personagens", personagem.id));

## BUG #08 — regras de segurança abertas

## Localização
Arquivo e Localização: firestore.rules

### O que estava acontecendo
O banco de dados estava completamente aberto, permitindo leitura, alteração e exclusão irrestrita de dados por qualquer requisição não autenticada.

### Por que acontecia
As regras de segurança utilizavam a instrução genérica allow read, write: if true;, concedendo privilégios totais a qualquer cliente.

### Como corrigi

 match /{document=**} {
  allow read, write: if true;
}

 corrigido:
match /personagens/{personagemId} {
  allow read: if request.auth != null &&
              request.auth.uid == resource.data.userId;
  allow create: if request.auth != null &&
                request.auth.uid == request.resource.data.userId;
  allow update, delete: if request.auth != null &&
                        request.auth.uid == resource.data.userId;
}


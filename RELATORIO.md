## BUG #01 — bug do login

### O que estava acontecendo
Quando você digita a senha ou o email errado, o botão fica em entrando e nenhuma mensagem de erro aparece

### Por que acontecia
o catch (mensagem de erro) estava vazio.
linha aproximadamente 48

### Como corrigi

 catch {}

corrigido:
catch {
       setErro('E-mail ou senha inválidos.')
    } 



## BUG #02 — rota desprotegida

### O que estava acontecendo
middleware com condição invertida

### Por que acontecia
faltava o ! antes do token para inverter a função booleana
linha aproximadamente 29


### Como corrigi

if (token) {
  return NextResponse.redirect(new URL("/login", request.url));
}

corrigido:
if (!token) {
      return NextResponse.redirect(new URL("/login", request.url));
    }

## BUG #03 — bug da senha

### O que estava acontecendo
o campo confirmar senha não funciona como deveria

### Por que acontecia
faltava o ! antes do token para inverter a função booleana
linha aproximadamente 29


### Como corrigi

if (token) {
  return NextResponse.redirect(new URL("/login", request.url));
}

corrigido:
if (!token) {
      return NextResponse.redirect(new URL("/login", request.url));
    }


## BUG #03 — bug da senha

### O que estava acontecendo
o campo confirmar senha não funciona como deveria

### Por que acontecia
faltava o ! antes do token para inverter a função booleana
linha aproximadamente 29


### Como corrigi

if (token) {
  return NextResponse.redirect(new URL("/login", request.url));
}

corrigido:
if (!token) {
      return NextResponse.redirect(new URL("/login", request.url));
    }

## BUG #04 — bug dos personagem

### O que estava acontecendo
personagens criados dos outros usuários aparecem

### Por que acontecia
linha aproximadamente 28
import no lugar errado e faltava a segunda condição


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

### O que estava acontecendo
personagem some do dashboard

### Por que acontecia
nome era pra ser em plural mas estava no singular 

const ref = await addDoc(collection(db, "personagem"), { ... });



corrigido:

const ref = await addDoc(collection(db, "personagens"), {
    nome,
    classe,
    nivel: 1,
    xp: 0,
    userId: uid,
    criadoEm: serverTimestamp(),
  });
  return ref.id;
}



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

### O que estava acontecendo
Cada novo item equipado apaga todos os outros

### Por que acontecia
setDoc apaga o documento inteiro ao invés de atualizar só o campo
linha aproximadamente 82

### Como corrigi

 await setDoc(doc(db, "personagens", personagemId), { [slot]: itemId });

 corrigido:
 await updateDoc(doc(db, "personagens", personagemId), { [slot]: itemId });


## BUG #07 — bug deletar

### O que estava acontecendo
Deletar Personagem Deleta o Errado

### Por que acontecia
setDoc apaga o documento inteiro ao invés de atualizar só o campo
linha aproximadamente 100

### Como corrigi

 await deleteDoc(doc(db, "personagens", String(indice)));

 corrigido:
await deleteDoc(doc(db, "personagens", personagem.id));

## BUG #08 — regras de segurança abertas

### O que estava acontecendo
voce podia ver todos os personagems de todos os usuários

### Por que acontecia
regras do firestore erradas
firebase.firestore.rules

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


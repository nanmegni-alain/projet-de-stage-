# index.ts:
import Sqlite3 from "better-sqlite3"; 
//Importe la fonction Fastify depuis le paquet fastify 
import Fastify from "fastify"; 
 
//Crée une instance de Fastify 
const fastify = Fastify(); 
 
//Crée une route HTTP GET vers l’URL / 
const db = Sqlite3("../database/db.sqlite");
fastify.get("/user", (request, reply) => { 

    //Envoie une string en réponse au client 
    const query = db.prepare("SELECT * FROM user;"); 
    const results = query.all(); 
    reply.send(results); 
}); 
 
//Démarre le serveur Web qui écoutera sur le port 8080 
fastify.listen({port : 8080});


# Explication du TD sur la creation d'un API avec fastify

Je peux t'expliquer ce TD comme si nous étions en cours, en détaillant chaque notion, chaque commande et chaque étape. L'objectif n'est pas seulement de le terminer, mais de comprendre pourquoi on fait chaque manipulation.

1. Quel est l'objectif du TD ?

Le but est de créer une API Web avec NodeJS, TypeScript, Fastify et une base de données SQLite.

L'application est composée de deux parties :

                Utilisateur
                      │
                      │
               Navigateur Web
                      │
                      │ Requête HTTP
                      ▼
              API NodeJS + Fastify
                      │
                      │ Requête SQL
                      ▼
              Base de données SQLite

Le frontend (interface graphique) existe déjà.

Toi, tu dois développer le backend (API).

2. Architecture du projet

Le TD demande de créer cette structure :

Projet
│
├── front/
│      index.html
│      css/
│      js/
│
└── api/
       │
       ├── src/
       │      index.ts
       │
       ├── database/
       │      db.sqlite
       │
       ├── dist/
       │
       ├── package.json
       └── tsconfig.json

Le dossier front contient l'interface utilisateur.

Le dossier api contient le serveur que tu vas programmer.

3. Pourquoi utiliser TypeScript ?

Tu écris :

index.ts

et TypeScript le transforme en

index.js

Cette transformation s'appelle la transpilation.

index.ts
      │
      │ npx tsc
      ▼
index.js

NodeJS ne comprend que JavaScript.

C'est pourquoi on doit toujours transpiler avant d'exécuter le programme.

4. tsconfig.json

Le TD demande d'exécuter :

npx tsc --init

Cette commande crée un fichier :

tsconfig.json

qui indique à TypeScript comment travailler.

rootDir
rootDir = ./src

signifie :

"Tous les fichiers TypeScript du projet sont dans src."

api/
│
├── src/
│      index.ts
│
└── dist/
outDir
outDir = ./dist

signifie :

"Tous les fichiers JavaScript générés seront placés dans dist."

index.ts
      │
      ▼
transpilation
      │
      ▼
dist/index.js

Ainsi on ne mélange jamais les fichiers TS et JS.

5. Premier programme

Le TD demande :

console.log("Hello World !");

Puis :

npx tsc

puis

node index.js

Pourquoi ?

Parce que cela vérifie que :

NodeJS est installé
TypeScript est installé
la transpilation fonctionne
Node peut exécuter le JavaScript généré.
6. Fastify

Le TD demande ensuite :

npm i fastify

Fastify est un framework qui permet de créer facilement un serveur HTTP.

Sans Fastify :

Client
   │
   ▼
NodeJS

Avec Fastify :

Client
   │
   ▼
Fastify
   │
   ▼
NodeJS

Fastify simplifie énormément le développement.

7. Comprendre le premier serveur

Le TD fournit ce code :

import Fastify from "fastify";

const fastify = Fastify();

fastify.get("/", (request, reply) => {
    reply.send("Hello World !");
});

fastify.listen({port : 8080});

Décomposons-le.

Étape 1
import Fastify from "fastify";

On importe la bibliothèque Fastify.

Étape 2
const fastify = Fastify();

On crée un serveur Web.

Fastify()
      │
      ▼
Serveur HTTP
Étape 3
fastify.get("/")

GET signifie :

"Lorsqu'un navigateur demande cette URL..."

GET /
Étape 4
reply.send("Hello World!")

Le serveur répond :

Hello World!
Étape 5
fastify.listen({port:8080})

Le serveur écoute sur le port 8080.

Le navigateur envoie donc :

http://localhost:8080

et reçoit

Hello World !
8. La base de données SQLite

Le TD ajoute ensuite une base SQLite.

API
 │
 │ SELECT
 ▼
SQLite

Le paquet utilisé est

better-sqlite3

qui permet d'exécuter des requêtes SQL directement depuis TypeScript.

9. Première API : GET /users

Le TD demande :

GET /users

qui retourne tous les utilisateurs.

Le fonctionnement est :

Navigateur

GET /users
      │
      ▼

Fastify

      │
SELECT * FROM users

      ▼

SQLite

      │

Liste des utilisateurs

      ▼

Fastify

      │

JSON

      ▼

Navigateur

La documentation indique que la réponse est un tableau JSON contenant :

[
   {
      id,
      name,
      picture
   }
]

10. Le problème CORS

Le frontend est chargé depuis :

http://127.0.0.1:5500

alors que l'API est sur

http://localhost:8080

Pour le navigateur, ce sont deux origines différentes.

Il bloque donc la requête :

Frontend
      │
      │ GET /users
      ▼

API

❌ Refusée (CORS)

Le TD demande donc d'installer :

@fastify/cors

puis :

import cors from "@fastify/cors";

fastify.register(cors);

pour autoriser les requêtes venant du frontend.

11. POST /users

Cette route sert à créer un utilisateur.

Le frontend envoie :

{
    "name":"Alice",
    "mail":"alice@test.com",
    "password":"1234"
}

Le serveur :

récupère les données ;
vérifie qu'elles sont valides ;
vérifie que l'email n'existe pas déjà ;
insère l'utilisateur dans SQLite.
Client

POST /users
        │

{name,mail,password}

        ▼

API

        │

INSERT INTO users

        ▼

SQLite

        │

204 ou 409

        ▼

Client

Le TD précise :

204 : compte créé ;
409 : email déjà utilisé ;
500 : erreur serveur.
12. POST /auth

Cette route permet la connexion.

Le client envoie :

{
    "mail":"alice@test.com",
    "password":"1234"
}

Le serveur :

POST /auth

      │

Recherche utilisateur

      │

Mot de passe correct ?

      │

Oui

      ▼

Création JWT

      ▼

Retour du token

Si les identifiants sont incorrects :

404

Sinon :

200 + token

13. JWT (Json Web Token)

Après connexion :

Utilisateur
      │

mail + password

      ▼

Serveur

      │

JWT.sign(...)

      ▼

eyJhbGc...

Le client conserve ce token.

À chaque nouvelle requête :

Authorization:
Bearer eyJhbGc...

Le serveur vérifie le token avec :

JWT.verify(...)

Si le token est invalide :

401 Unauthorized

14. POST /messages

Cette route permet d'envoyer un message.

Utilisateur connecté

      │

POST /messages

{
   to_user,
   message
}

      │

Token valide ?

      │

Oui

      ▼

Insertion en base

      ▼

Retour du message enregistré

Le serveur renvoie :

200 : succès ;
401 : token invalide ;
404 : destinataire inconnu ;
500 : erreur serveur.
15. GET /messages/:userId

Cette route récupère toute une conversation.

GET /messages/5

        │

Token

        ▼

API

        │

SELECT messages

        ▼

SQLite

        │

Liste JSON

        ▼

Frontend

16. Sécurité

Le TD explique qu'il ne faut jamais stocker un mot de passe en clair.

Au lieu de stocker :

123456

on calcule un hash SHA-256 :

123456

↓

SHA-256

↓

8d969eef6ecad3c29...

Le serveur stocke uniquement cette valeur.

À la connexion :

mot de passe saisi

↓

SHA-256

↓

comparaison avec la base

Le mot de passe réel n'est jamais enregistré.




 #Oui. Pour un projet moderne, je te recommande même de faire ce sujet en TypeScript + Fastify plutôt qu'en JavaScript pur.

L'idée est la suivante :

Frontend
├── HTML
├── CSS
└── TypeScript

Backend
└── Fastify (TypeScript)
1. Création du projet
npm init -y

Installation :

npm install fastify @fastify/static

Dépendances TypeScript :

npm install -D typescript ts-node-dev @types/node

Initialisation :

npx tsc --init
2. Structure du projet
atm-project/

├── src/
│   ├── server.ts
│   │
│   └── public/
│       ├── index.html
│       ├── style.css
│       └── app.ts
│
├── package.json
└── tsconfig.json
3. Configuration TypeScript
tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "CommonJS",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "dist"
  }
}
Explication
{
  "strict": true
}

Active les vérifications fortes.

Exemple :

let age: number = 25;

age = "bonjour";

Erreur immédiatement.

4. Serveur Fastify TypeScript
server.ts
import Fastify from "fastify";
import fastifyStatic from "@fastify/static";
import path from "path";

const app = Fastify({
    logger: true
});

app.register(fastifyStatic, {
    root: path.join(__dirname, "public")
});

app.listen({
    port: 3000
});
5. Modélisation des données

Le sujet utilise deux réponses API :

Balance :

{
  "balance": 500
}

Retrait :

{
  "newBalance": 450
}

Créons des interfaces.

interface BalanceResponse {
    balance: number;
}

interface WithdrawResponse {
    newBalance: number;
}
6. Variables globales
app.ts
let currentScreen: number = 1;

let currentStudent: string = "";

let inactivityTimer: number;
7. Récupération des éléments HTML

TypeScript impose le typage.

const studentInput =
document.getElementById(
    "studentNumber"
) as HTMLInputElement;

const withdrawInput =
document.getElementById(
    "withdrawAmount"
) as HTMLInputElement;

const balanceDisplay =
document.getElementById(
    "balance"
) as HTMLParagraphElement;

const errorDisplay =
document.getElementById(
    "error"
) as HTMLParagraphElement;
Pourquoi utiliser "as" ?

JavaScript :

const input =
document.getElementById("studentNumber");

TypeScript :

const input =
document.getElementById("studentNumber")
as HTMLInputElement;

Sinon TypeScript ne sait pas que l'objet possède :

input.value
8. Gestion du clavier
const digits =
document.querySelectorAll(".digit");

digits.forEach(button => {

    button.addEventListener("click", () => {

        resetTimer();

        const value =
        button.textContent ?? "";

        if(currentScreen === 1){

            studentInput.value += value;

        }else{

            withdrawInput.value += value;
        }
    });

});
Le "??"
const value =
button.textContent ?? "";

signifie :

si textContent est null
alors utilise ""
9. Bouton Supprimer
const deleteButton =
document.getElementById("delete")!;

deleteButton.addEventListener(
    "click",
    () => {

        resetTimer();

        const field =
            currentScreen === 1
                ? studentInput
                : withdrawInput;

        field.value =
        field.value.slice(0,-1);
    }
);
10. Appel API Balance

Le sujet indique :

GET

/accounts/student_number/balance

TypeScript :

async function getBalance(
    studentNumber: string
): Promise<BalanceResponse> {

    const response =
    await fetch(
        `https://bank.lesmoulinsdudev.com/accounts/${studentNumber}/balance`
    );

    return await response.json();
}
Explication Promise

La fonction :

Promise<BalanceResponse>

signifie :

je vais renvoyer
un objet BalanceResponse
mais plus tard

après la réponse du serveur.

11. Validation étudiant
async function validateStudent()
: Promise<void> {

    currentStudent =
    studentInput.value;

    const data =
    await getBalance(currentStudent);

    balanceDisplay.textContent =
    `${data.balance} €`;

    showWithdrawScreen();
}
12. Affichage écran retrait
function showWithdrawScreen()
: void {

    const screen1 =
    document.getElementById("screen1")!;

    const screen2 =
    document.getElementById("screen2")!;

    screen1.classList.add("hidden");

    screen2.classList.remove("hidden");

    currentScreen = 2;
}
13. Appel API Retrait

Le sujet indique :

POST

/accounts/student_number/withdraw

avec :

{
amount:number
}

async function withdraw(
    studentNumber:string,
    amount:number
): Promise<Response> {

    return fetch(
        `https://bank.lesmoulinsdudev.com/accounts/${studentNumber}/withdraw`,
        {
            method:"POST",

            headers:{
                "Content-Type":
                "application/json"
            },

            body:JSON.stringify({
                amount
            })
        }
    );
}
14. Validation retrait
async function validateWithdraw()
: Promise<void> {

    const amount =
    Number(withdrawInput.value);

    const response =
    await withdraw(
        currentStudent,
        amount
    );

    if(response.status === 200){

        backToHome();

        return;
    }

    if(response.status === 412){

        errorDisplay.textContent =
        "Fonds insuffisants";
    }
}

Le sujet précise :

412 = Fonds insuffisants

15. Bouton Valider
const validateButton =
document.getElementById("validate")!;

validateButton.addEventListener(
    "click",
    async () => {

        resetTimer();

        if(currentScreen === 1){

            await validateStudent();

        }else{

            await validateWithdraw();
        }
    }
);
16. Bouton Annuler

Le sujet impose :

vider le champ
ou revenir écran accueil si vide
const cancelButton =
document.getElementById("cancel")!;

cancelButton.addEventListener(
    "click",
    () => {

        resetTimer();

        if(currentScreen === 1){

            studentInput.value = "";

            return;
        }

        if(withdrawInput.value !== ""){

            withdrawInput.value = "";

        }else{

            backToHome();
        }
    }
);
17. Retour accueil
function backToHome()
: void {

    studentInput.value = "";

    withdrawInput.value = "";

    errorDisplay.textContent = "";

    document
        .getElementById("screen1")!
        .classList.remove("hidden");

    document
        .getElementById("screen2")!
        .classList.add("hidden");

    currentScreen = 1;
}
18. Gestion du timeout

Le sujet demande :

10 secondes d'inactivité
→ retour écran accueil

function resetTimer()
: void {

    clearTimeout(inactivityTimer);

    inactivityTimer =
    window.setTimeout(
        () => {

            backToHome();

        },
        10000
    );
}

Initialisation :

resetTimer();
Ce que TypeScript apporte ici
Vérification des types
let amount:number = 50;

Impossible de faire :

amount = "abc";
Autocomplétion

Quand tu écris :

studentInput.

VS Code propose :

value
focus()
select()
Détection d'erreurs avant exécution
const balance:number =
response.balance;

Si l'API renvoie autre chose, TypeScript te le signalera pendant le développement.

Pour un examen ou un projet professionnel, je structurerais encore davantage le code en modules :

src/

├── types/
│   ├── BalanceResponse.ts
│   └── WithdrawResponse.ts

├── services/
│   └── bankApi.ts

├── views/
│   └── atm.ts

└── app.ts

C'est l'architecture TypeScript/Fastify que l'on retrouve le plus souvent dans les projets réels.

# Code TypeScript de création de la base
// db.ts
import Database from "better-sqlite3";

// 1. Connexion / création du fichier de base
const db = new Database("app.db");

// 2. Création de la table users
db.exec(`
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL,
    created_at TEXT NOT NULL
);
`);

console.log("Base de données et table créées avec succès.");

# 4. Ajout d’un utilisateur (INSERT)
import Database from "better-sqlite3";

const db = new Database("app.db");

const insert = db.prepare(`
INSERT INTO users (name, email, password, created_at)
VALUES (?, ?, ?, ?)
`);

insert.run(
    "Alain",
    "alain@mail.com",
    "1234",
    new Date().toISOString()
);

console.log("Utilisateur ajouté.");

# Lecture des utilisateurs (SELECT)
import Database from "better-sqlite3";

const db = new Database("app.db");

const users = db.prepare("SELECT * FROM users").all();

console.log(users);
🧪 6. Simulation du fonctionnement interne

Quand tu exécutes ton fichier :

Étape 1 : ouverture
SQLite → ouverture app.db
(si fichier absent → création automatique)


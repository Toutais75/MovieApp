# Module Member – Documentation complète

## 📌 Objectif

Le module Member gère l’ensemble du cycle de vie des utilisateurs :
création, connexion, permissions, mot de passe et activation.

Aucune inscription publique n’est autorisée.

---

## 🔐 Connexion

Conditions pour se connecter :

- Le compte existe
- Le compte est activé (`activated = true`)
- Le mot de passe est valide

Si `mustChangePassword = true`, l’utilisateur est redirigé
vers la création ou la modification de son mot de passe.

---

## 👥 Rôles et permissions

### ROLE_ADMINISTRATEUR
- Accès total
- Création, modification, suppression de tous les membres
- Attribution de tous les rôles
- Activation / désactivation des comptes

### ROLE_METTEUR_EN_SCENE
- Création de membres
- Accès aux membres de son théâtre
- Activation / désactivation
- Ne peut pas modifier les profils

### ROLE_MEMBER
- Accès à son espace personnel
- Modification de son profil
- Modification de son mot de passe

---

## ➕ Création d’un membre

La création est possible uniquement par :
- Administrateur
- Metteur en scène

État initial :
- activated = false
- mustChangePassword = true
- mot de passe provisoire

---

## 🔑 Mot de passe

### Premier mot de passe
- Controller : `MemberPasswordController`
- FormType : `FirstPasswordType`

Champs :
- Mot de passe provisoire
- Nouveau mot de passe
- Confirmation

Effets :
- Mot de passe enregistré
- activated = true
- mustChangePassword = false

### Modification du mot de passe
- Même controller
- Même formulaire
- Twig conditionnel

---

## 🛡️ Sécurité

### Voter : MemberVoter
- CREATE → admin, metteur en scène
- EDIT → uniquement son propre profil

### Subscriber
- `ForcePasswordChangeSubscriber`
- Redirection forcée si mustChangePassword = true

---

## 📊 Diagramme de flux – Connexion

```mermaid
flowchart TD
    A[Login] --> B{Compte actif ?}
    B -- Non --> C[Refus]
    B -- Oui --> D{Mot de passe valide ?}
    D -- Non --> E[Erreur]
    D -- Oui --> F{mustChangePassword ?}
    F -- Oui --> G[Formulaire mot de passe]
    F -- Non --> H[Espace membre]

classDiagram
    class Member {
        id
        email
        password
        roles
        activated
        mustChangePassword
    }

    class MemberVoter
    class MemberCrudController
    class MemberPasswordController
    class FirstPasswordType
    class ForcePasswordChangeSubscriber

    Member --> MemberVoter
    Member --> MemberCrudController
    Member --> MemberPasswordController
    MemberPasswordController --> FirstPasswordType
    ForcePasswordChangeSubscriber --> Member

sequenceDiagram
    participant User
    participant Security
    participant Subscriber
    participant PasswordController
    participant DB

    User->>Security: login
    Security->>DB: check credentials
    DB-->>Security: ok

    Security->>Subscriber: kernel.request
    Subscriber->>DB: check mustChangePassword

    alt mustChangePassword
        Subscriber-->>User: redirect password
        User->>PasswordController: submit password
        PasswordController->>DB: update password
        PasswordController-->>User: redirect space
    else
        User-->>User: access space
    end

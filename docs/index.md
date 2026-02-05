# Accueil Playerbots

Bienvenue sur la documentation technique du module Playerbots pour AzerothCore.

## Fonctionnement de l'IA
Voici un schéma de décision d'un bot :

```mermaid
graph TD;
    A[Bot détecte ennemi] --> B{Ennemi proche ?};
    B -- Oui --> C[Attaque corps à corps];
    B -- Non --> D[Lancer Sort];
    C --> E[Boucle de combat];
    D --> E;

---
title: "Django et le frontend"
---

Le samedi 21 mai 2016 j'ai présenté une session sur Django et le frontend où comment utiliser React/Redux (ou autres projets front) dans un projet Django de façon simple.

Vous pouvez voir les slides sur [Speakerdeck](https://speakerdeck.com/arnaudlimbourg/django-and-the-frontend) et le code du projet est sur [Github](https://github.com/arnaudlimbourg/rencontres-django-2016). Revoyons les grandes étapes de la mise en place.

## Django
Installation de django-webpack-loader
Ajout dans les settings des morceaux suivants

```python
'webpack_loader' # INSTALLED_APPS

# Ajout du répertoire avec l'application
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'assets'),
)
```

Toutes les autres valeurs sont celles par défaut de django-webpack-loader.

## Javascript

Sans répéter le code qui se trouve sur Github les étapes sont les suivantes:

- npm install
- création server.js
- création .babelrc
- création assets/webpack.config.js
- création du point d'entrée indiqué dans webpack config soit assets/js/rencontres/index.jsx
- le reste des composants React avec Redux, immutable, etc.

N'hésitez pas à envoyer vos questions sur [Twitter](https://twitter.com/arnaudlimbourg).

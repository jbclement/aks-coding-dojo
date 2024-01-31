# Utilisez une image Nginx de base
FROM nginx:latest

# Copiez le fichier index.html depuis le répertoire actuel du build vers /usr/share/nginx/html/ dans le conteneur
COPY index.html /usr/share/nginx/html/

# Exposez le port 80 pour permettre l'accès au serveur web Nginx
EXPOSE 80

# La commande CMD spécifie la commande à exécuter lorsque le conteneur démarre
CMD ["nginx", "-g", "daemon off;"]

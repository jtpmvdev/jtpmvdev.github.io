# www

### Comment installer le widget dans Grist :

1. Enregistrez le code ci-dessus dans un fichier (ou hébergez-le sur Github Pages / un serveur web).
2. Dans votre document **Grist** :
   - Ajoutez une nouvelle vue de type **Custom Widget**.
   - Dans le panneau de droite (*Widget Options*), réglez l'URL du widget vers votre fichier HTML (ou sélectionnez *Custom URL*).
   - Assurez-vous d'accorder les permissions **Full Access** au widget pour qu'il puisse soumettre des fichiers et mettre à jour vos lignes.
   - Assurez-vous d'avoir une colonne de type **Attachment (Pièce Jointe)** nommée `Fichier` dans votre table.

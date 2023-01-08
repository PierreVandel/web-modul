Pour ce site, j'ai essayé de faire un memory utilisant des cartes NFC. Le but du jeu est de retrouver les paires de cartes
en utilisant un téléphone. Lorsque l'on va scanner une carte, une image lié à cette carte va s'afficher sur le téléphone.
Je lie à chaque paire de carte une image (j'aurai aussi voulu associer un son à chaque paire). En scannant les cartes une
à une, nous pouvons retrouver les paires. Pour tester le site, vous avez besoin de 6 cartes NFC vierges.

J'utilise en tout 3 technologies web : NFC, la capture de video et la sauvegarde de fichier

Pour commencer, on initialise le jeu en prennant une photo d'une personne ou d'un paysage : il faut cliquer sur les
boutons "Grab video", "Take photo" et si la photo vous plaît "save image". Ensuite, nous avons besoin d'écrire sur les cartes NFC avec
le bouton "Write tag to link the photo" (2 cartes par photo). 
Pour des raisons d'espace mémoire qu'offre les NFC, je n'ai pas réussi à écrire l'image directement dans la carte, j'ai alors
du utiliser une référence ("tag"). 

Je n'ai malheureusement pas assez de connaissance nécessaires sur les langages, l'architecture et le workflow qu'il faut utiliser en web. 
Je suis concient des problèmes d'architecture que possède mon code mais j'ai essayé de faire de mon mieux.

J'aurai aimer avoir plus de cours sur les langages web en eux même et des cours sur les bonnes pratiques à utiliser en web.

Pierre VANDEL
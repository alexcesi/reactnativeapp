Lien github
https://github.com/alexcesi/reactnativeapp

## Créer le projet 
npx create-react-app <nom du projet>
## Github actions générer le main.yml de base
L'intégration continue a été mise en place avec github action.
Dans votre repo, allez dans l'onglet Actions et cliquez sur "set up a workflow yourself".
Un fichier yml est généré. Vous pouvez soit le modifier maintenant soit cliquer sur "start commit" et le modifier par la suite dans votre éditeur de code
Ne pas oublier de faire un git pull pour récuperer le fichier.

## Modifier le .yml pour votre CI CD
### Nommer votre ci cd 
name: <NOM>
### Configurer la CI pour qu'elle se déclenche lors de chaque push sur une branche (ici master)
on:
  push:
    branches: [ <Nom de votre branche par défaut: master> ]

### Build votre projet react sur un ubuntu et installe les dépendances
jobs:
  build-react:
    runs-on: ubuntu-latest
    steps:
      - name: Clone repository
        uses: actions/checkout@v2
      - name: Use Node.js 16.x
        uses: actions/setup-node@v1
        with:
          node-version: 16.x
      - name: Install dependencies
        run: npm install
      - name: Build Artefact 
        run: npm run build

### Executer les tests par défaut de votre projet
  test-react:
    runs-on: ubuntu-latest
    steps:
      - name: Clone repository
        uses: actions/checkout@v2
      - name: Use Node.js 16.x
        uses: actions/setup-node@v1
        with:
          node-version: 16.x
      - name: Install dependencies
        run: npm install
      - name: Run test 
        run: npm run test

### variables secret
Dans votre repo github, allez dans settings -> Secrets -> Actions
Cliquez sur New repository secret et ajouter vos variables d'environnements 
### Ajout de la variable ssh privée
      - name: Adding Known Hosts
        run: ssh-keyscan -H ${{ secrets.SSH_HOST }} >> ~/.ssh/known_hosts
        
### On crée un dossier apk
      - name: Add folder apk
        run: ssh ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} mkdir -p ./mobile/apk

### On copie dans dossier apk avec rsync en ssh avec notre login du serveur distant et l'ip de ce serveur dans le dossier /mobile/apk
      - name: Deploy with rsync
        run: rsync -avz ./android/app/build/outputs/apk/release/app-release.apk ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }}:~/mobile/apk

### Versionning
Pour ajouter une version à vos différents builds :
      - name: Rename folder
        run: ssh ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} 'mv ~/mobile/apk ~/mobile/apk-$(date +%Y%m%d_%H%M%S)'

name: CICD for Dev-React
on:
  push:
    branches:
      - dev-react

jobs:
  run-frontend-on-ec2:
    name: Deploy Frontend
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x]

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2
        with:
          ref: dev-react

      - name: Set up Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v2
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install sshpass
        run: |
          sudo apt-get update
          sudo apt-get install -y sshpass
      - name: SSH to instance
        env:
          HOSTNAME: ${{ secrets.FICTIVEBOX_VM_SSH_HOST }}
          PASSWORD: ${{ secrets.FICTIVEBOX_VM_PASSWORD }}
          USER_NAME: ${{ secrets.FICTIVEBOX_SSH_USER_NAME }}
          GITHUB_TOKEN: ${{ secrets.FICTIVEBOX_TOKEN }}
        run: |
          sshpass -p "${PASSWORD}" ssh -o StrictHostKeyChecking=no ${USER_NAME}@${HOSTNAME} '
            cd /home/fictivebox-devunnati/htdocs/dev-unnati.fictivebox.tech &&
            echo "Deploymentstarted" >> test.txt
            if [ ! -d "Unnati" ]; then
              echo "Cloning the repository..."
              git clone git@github.com:Fictivebox-Digital/Unnati.git
            else
              cd Unnati
              echo "Updating the repository..."
              git pull git@github.com:Fictivebox-Digital/Unnati.git
                cd ..
            fi
            git stash && git checkout dev-react &&
            git stash && git pull origin dev-react &&
            echo "Installing Dependencies"
            npm install &&
            npm install -g serve &&
            npm install -g pm2 &&
            npm run build &&
            pm2 stop dev-react-start || true &&
            pm2 serve build 3000 --name "dev-react-start" &&
            pm2 restart dev-react-start  &&
            pm2 save
          '
        

sudo apt update
sudo apt install git

git --version

git config --global user.name "Twoje Imię"
git config --global user.email "email (I used generated private git address)"
git config --list

ssh-keygen -t ed25519 -C "twoj@email.pl"

eval "$(ssh-agent -s)"

ssh-add ~/.ssh/id_ed25519

Kopia wygenerowanego klucza publicznego i jego publikacja na github

ssh -T git@github.com

POŁĄCZONE

git clone <adress SSH z repozytorium>

git pull
git add . lub git add -A
git status
git commit -m "Opis zmian"
git push

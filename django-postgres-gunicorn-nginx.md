Guide that covers the install of django with postgres, gunicorn and niginx. 
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip python3-dev python3-venv libpq-dev -y
mkdir ~/mydjangoapp && cd ~/mydjangoapp
# I prefer to put Django in "/var/www/dir/" as it's easier to share with the nginx user.
python3 -m venv myenv

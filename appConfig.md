# Config the VM for the web app

install nvm packege manager
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```

This load the nvm
```
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```
Now type `exit` to exit the VM and reconnect
connect with the ssh
```
ssh -i id_rsa_file username@IP_address
```

check the version
```
nvm -v
```

check for the version you need
```
nvm ls-remote
```
and install it, i choose v14.17.6
```
nvm install v14.17.6
 ```
To see wich verstion is default
```
nvm list
```
the output is 
```
->     v14.17.6
default -> v14.17.6
iojs -> N/A (default)
unstable -> N/A (default)
node -> stable (-> v14.17.6) (default)
stable -> 14.17 (-> v14.17.6) (default)
lts/* -> lts/fermium (-> N/A)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.17.1 (-> N/A)
lts/carbon -> v8.17.0 (-> N/A)
lts/dubnium -> v10.24.1 (-> N/A)
lts/erbium -> v12.22.6 (-> N/A)
lts/fermium -> v14.18.0 (-> N/A)
```

And we can see the olso by `node --version`

init gin in your project
```
git init
```
config global vairiable 
```
git config --global user.name "your name"
```
and email address
```
git config --global user.email username@domain.com
```
You can see the value 
```
git config user.name
git config user.email
```

# cloe the repo
```
git clone https://github.com/MatanTal2/bootcamp-app.git
```
type `ls` to see the new folder, `cd` to it and type `ls -la` to see all the repository file

Next, use `npm` to initialize the project’s `package.json` file.
```
npm init -y
```

In this tutorial, you will use hapi, an excellent application framework that supports all the latest features of Node.js and the JavaScript language. 
Here is an overview of the modules you will use in this project.

Install the project dependencies using the following npm commands
```
npm install @hapi/hapi@19 @hapi/bell@12 @hapi/boom@9 @hapi/cookie@11 @hapi/inert@6 @hapi/joi@17 @hapi/vision@6 dotenv@8 ejs@3 postgres@1

npm install --save-dev nodemon@2
```

Create a new file named `.env` in the root of the project and add the following configuration.
```
touch .env
```
write inside
```
# Host configuration
PORT=8080
HOST=WEB_VM_IP_ADDRESS
NODE_ENV=development
HOST_URL=http://WEB_VM_IP_ADDRESS:8080
COOKIE_ENCRYPT_PWD=superAwesomePasswordStringThatIsAtLeast32CharactersLong!

# Okta configuration
OKTA_ORG_URL=https://{yourOrgUrl}
OKTA_CLIENT_ID={yourClientId}
OKTA_CLIENT_SECRET={yourClientSecret}

# Postgres configuration
PGHOST=SQL_VM_IP_ADDRESS
PGUSERNAME=postgres
PGDATABASE=postgres
PGPASSWORD=p@ssw0rd42
PGPORT=5432
```
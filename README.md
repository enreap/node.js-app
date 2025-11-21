# node.js-app

To start this application first you can get this repo code using below url

git clone https://github.com/enreap/node.js-app.git

cd node.js-app

npm install

node app.js 

(OR) 

npm start

To Execute the SonarQube Repor, execute the below command.

npm run sonar

(OR) 

node sonar-project.js

to upload into nexus repo, user below command
npm publish

To generate the Nexus token with base 64 format, we will use below command

echo -n "username:password" | openssl base64



FROM 012346589737.dkr.ecr.us-east-1.amazonaws.com/base-image 
 
ENV PORT=80 
 
EXPOSE $PORT 
 
WORKDIR /app 
 
COPY package.json /app 
 
RUN npm install 
 
COPY . /app 
 
CMD node app.js

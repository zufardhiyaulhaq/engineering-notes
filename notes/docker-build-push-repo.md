# Building & Push Docker Image into Repository
- Login to repository
```
docker login DOCKER_URL
```
for default public registry (hub.docker.com), simply 
```
docker login
```
- Build your image
```
docker build -t USERNAME/IMAGE_NAME:TAGS .
```
for example
```
docker build -t zufardhiyaulhaq/rubyweekly_dispatcher:latest .
```
- Push to repository
```
docker push USERNAME/IMAGE_NAME:TAGS
```
for example
```
docker push zufardhiyaulhaq/rubyweekly_dispatcher:latest
```



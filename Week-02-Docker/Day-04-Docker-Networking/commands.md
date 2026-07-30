# List networks
docker network ls

# Inspect a network
docker network inspect bridge

# Create a network
docker network create devops-net

# Remove a network
docker network rm devops-net

# Connect a container
docker network connect devops-net nginx1

# Disconnect a container
docker network disconnect devops-net nginx1

# Run on a network
docker run -d --network devops-net nginx

# Publish a port
docker run -d -p 8080:80 nginx

# Publish all exposed ports
docker run -P nginx

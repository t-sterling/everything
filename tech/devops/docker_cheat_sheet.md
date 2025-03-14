# 🐳 Docker Cheat Sheet

## **1. Installation & Version**
```sh
docker --version        # Check Docker version
docker info             # Get system-wide information
```

## **2. Working with Containers**
### Run & Manage Containers
```sh
docker run -d -p 8080:80 --name mycontainer nginx  # Run container in detached mode
docker ps                                          # List running containers
docker ps -a                                       # List all containers
docker stop mycontainer                            # Stop a container
docker start mycontainer                           # Start a stopped container
docker restart mycontainer                         # Restart a container
docker rm mycontainer                              # Remove a container
docker logs -f mycontainer                         # View logs of a running container
docker exec -it mycontainer bash                   # Enter a running container
```

## **3. Working with Images**
```sh
docker pull nginx                # Download an image from Docker Hub
docker images                    # List all downloaded images
docker rmi nginx                  # Remove an image
docker build -t myimage .         # Build an image from a Dockerfile
docker tag myimage myrepo/myimage:latest  # Tag an image
docker push myrepo/myimage:latest # Push an image to Docker Hub
```

## **4. Docker Volumes**
```sh
docker volume create myvolume     # Create a volume
docker volume ls                  # List volumes
docker volume inspect myvolume    # Inspect a volume
docker volume rm myvolume         # Remove a volume
```

## **5. Docker Networks**
```sh
docker network ls                 # List networks
docker network create mynetwork   # Create a network
docker network inspect mynetwork  # Inspect a network
docker network connect mynetwork mycontainer  # Connect a container to a network
docker network disconnect mynetwork mycontainer  # Disconnect a container from a network
docker network rm mynetwork       # Remove a network
```

## **6. Docker Compose**
```sh
docker-compose up -d              # Start services in detached mode
docker-compose down               # Stop and remove containers, networks, and volumes
docker-compose ps                 # List services
docker-compose logs -f            # View logs
```

## **7. Clean Up Unused Resources**
```sh
docker system prune -a            # Remove all stopped containers, networks, and images
docker container prune            # Remove all stopped containers
docker image prune -a             # Remove unused images
docker volume prune               # Remove unused volumes
docker network prune              # Remove unused networks
```

## **8. Inspect & Debug**
```sh
docker inspect mycontainer        # Get detailed container information
docker stats                      # Display real-time container resource usage
docker top mycontainer            # Show running processes inside a container
docker events                     # View real-time events from the Docker daemon
```

## **9. Dockerfile Basics**
```dockerfile
# Use a base image
FROM python:3.9

# Set working directory
WORKDIR /app

# Copy files to the container
COPY . .

# Install dependencies
RUN pip install -r requirements.txt

# Expose port
EXPOSE 5000

# Command to run the application
CMD ["python", "app.py"]
```

## **10. Save & Load Images**
```sh
docker save -o myimage.tar myimage      # Save an image as a tar file
docker load -i myimage.tar              # Load an image from a tar file
```

## **11. Export & Import Containers**
```sh
docker export mycontainer -o mycontainer.tar   # Export container filesystem
docker import mycontainer.tar mynewimage       # Import container as an image
```

## **12. Docker Swarm (Orchestration)**
```sh
docker swarm init                    # Initialize a swarm
docker swarm join --token <token> <manager-ip>  # Join a swarm
docker node ls                        # List nodes in the swarm
docker service create --name myservice -p 80:80 nginx  # Deploy a service
docker service ls                     # List running services
docker service rm myservice            # Remove a service
```

---
**Happy Dockering!** 🚢

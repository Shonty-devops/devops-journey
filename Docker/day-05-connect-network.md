# Run both containers
docker run -d --name app-1 \
  -v /root/app-1:/usr/share/nginx/html \
  nginx:alpine

docker run -d --name app-2 \
  -v /root/app-2:/usr/share/nginx/html \
  nginx:alpine

# Confirm both running
docker ps

# Get app-2 IP address
docker inspect app-2 | grep IPAddress

# Send GET request by IP (replace x with actual IP)
docker exec app-1 wget -qO- 172.17.0.x

# Send GET request by hostname
docker exec app-1 wget -qO- app-2

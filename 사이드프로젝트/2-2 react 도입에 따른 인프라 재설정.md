
### docker-compose.yml 수정
```yml
  # React 프론트엔드 서비스 추가
  react:
    image: ghcr.io/anonichat/app/frontend:latest
    container_name: anonichat-frontend
    expose:
      - "3000"
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_API_URL=https://anonichat.world/api
    depends_on:
      - spring
    networks:
      - elk
    restart: unless-stopped
```
docker-compose.yml에 react 추가

### nginx 설정 수정
```

```

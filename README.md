# Docker na Ubuntu Serverju

Praktičen vodič za namestitev Dockerja, ustvarjanje imageov in containerjev, upravljanje storitev ter uporabo Docker Compose.

## Kazalo
1. [Namestitev Dockerja](#1-namestitev-dockerja)
2. [Preverjanje namestitve](#2-preverjanje-namestitve)
3. [Image in container](#3-image-in-container)
4. [Ustvarjanje in zagon containerja](#4-ustvarjanje-in-zagon-containerja)
5. [Upravljanje containerjev](#5-upravljanje-containerjev)
6. [Dostop do containerja](#6-dostop-do-containerja)
7. [Trajno shranjevanje podatkov](#7-trajno-shranjevanje-podatkov)
8. [Izdelava lastnega imagea](#8-izdelava-lastnega-imagea)
9. [Docker Compose](#9-docker-compose)
10. [Čiščenje in vzdrževanje](#10-čiščenje-in-vzdrževanje)
11. [Najpogostejši ukazi](#11-najpogostejši-ukazi)

---

## 1. Namestitev Dockerja

### Posodobi seznam paketov

```bash
sudo apt update
```

### Namesti potrebne pakete

```bash
sudo apt install -y ca-certificates curl
```

### Dodaj uradni Dockerjev ključ

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Dodaj uradno Dockerjevo skladišče

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

### Namesti Docker Engine in Docker Compose

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 2. Preverjanje namestitve

Zaženi Docker in omogoči samodejni zagon ob zagonu strežnika:

```bash
sudo systemctl enable --now docker
```

Preveri stanje storitve:

```bash
sudo systemctl status docker
```

Za izhod iz prikaza pritisni `q`.

Preveri različico Dockerja:

```bash
docker --version
docker compose version
```

Zaženi testni container:

```bash
sudo docker run hello-world
```

Če se izpiše `Hello from Docker!`, namestitev deluje.

## 3. Image in container

- **Image:** predloga, iz katere ustvarimo container.
- **Container:** zagnana instanca imagea.
- **Volume:** prostor za trajne podatke.
- **Port mapping:** povezava med vrati strežnika in containerja.
- **Dockerfile:** navodila za izdelavo lastnega imagea.
- **Docker Compose:** upravljanje storitev s konfiguracijsko datoteko YAML.

Prikaži lokalno shranjene image:

```bash
sudo docker images
```

Prenesi image Nginx:

```bash
sudo docker pull nginx:alpine
```

## 4. Ustvarjanje in zagon containerja

Ustvari in zaženi spletni strežnik Nginx:

```bash
sudo docker run -d \\
  --name moj-nginx \\
  --restart unless-stopped \\
  -p 8080:80 \\
  nginx:alpine
```

Pomen parametrov:

- `-d`: delovanje v ozadju.
- `--name`: ime containerja.
- `--restart unless-stopped`: samodejni ponovni zagon, razen če je bil container ročno ustavljen.
- `-p 8080:80`: vrata 8080 na strežniku poveže z vrati 80 v containerju.
- `nginx:alpine`: image, ki se uporabi.

Preveri delovanje:

```bash
sudo docker ps
```

Na računalniku odpri brskalnik:

```text
http://IP_STREZNIKA:8080
```

Primer:

```text
http://192.168.1.71:8080
```

Zamenjaj IP z naslovom svojega Ubuntu Serverja.

## 5. Upravljanje containerjev

### Prikaži delujoče containerje

```bash
sudo docker ps
```

### Prikaži vse containerje, tudi ustavljene

```bash
sudo docker ps -a
```

### Ustavi container

```bash
sudo docker stop moj-nginx
```

### Zaženi obstoječi container

```bash
sudo docker start moj-nginx
```

### Ponovno zaženi container

```bash
sudo docker restart moj-nginx
```

### Prikaži dnevnike

```bash
sudo docker logs moj-nginx
```

Zadnjih 100 vrstic:

```bash
sudo docker logs --tail 100 moj-nginx
```

Sprotno spremljanje dnevnikov:

```bash
sudo docker logs -f moj-nginx
```

Za izhod pritisni `Ctrl+C`. To praviloma ustavi samo spremljanje dnevnikov, ne containerja.

### Prikaži podrobnosti containerja

```bash
sudo docker inspect moj-nginx
```

### Spremljaj porabo virov

```bash
sudo docker stats
```

### Odstrani container

```bash
sudo docker rm moj-nginx
```

Če container še deluje, ga najprej ustavi.

Za prisilno odstranitev:

```bash
sudo docker rm -f moj-nginx
```

**Opozorilo:** pred odstranjevanjem preveri, ali container vsebuje pomembne podatke, ki niso shranjeni v volumeu ali vezani mapi.

## 6. Dostop do containerja

Odpri ukazno vrstico znotraj delujočega containerja:

```bash
sudo docker exec -it moj-nginx sh
```

Uporabni ukazi znotraj containerja:

```sh
pwd
ls -la
cat /etc/os-release
```

Zapusti container:

```sh
exit
```

Če image vsebuje Bash, lahko uporabiš:

```bash
sudo docker exec -it IME_CONTAINERJA bash
```

Če Bash ni na voljo, poskusi `sh`.

## 7. Trajno shranjevanje podatkov

Podatke, ki jih želiš ohraniti po odstranitvi containerja, shranjuj v volume ali mapo na strežniku.

### Primer z bind mountom

Ustvari mapo:

```bash
mkdir -p ~/docker/nginx/html
```

Če že obstaja testni container z istim imenom, ga odstrani:

```bash
sudo docker rm -f moj-nginx
```

Zaženi Nginx z vezano mapo:

```bash
sudo docker run -d \\
  --name moj-nginx \\
  --restart unless-stopped \\
  -p 8080:80 \\
  -v "$HOME/docker/nginx/html:/usr/share/nginx/html:ro" \\
  nginx:alpine
```

Pomen:

- `-v`: poveže mapo strežnika z mapo containerja.
- `~/docker/nginx/html`: mapa na strežniku.
- `/usr/share/nginx/html`: mapa spletne vsebine v containerju.
- `:ro`: samo branje znotraj containerja.

Če mapa na strežniku ne vsebuje `index.html`, privzeta stran Nginx ne bo prikazana, ker vezana mapa prekrije vsebino te poti v imageu.

Ustvari testno stran:

```bash
cat > ~/docker/nginx/html/index.html <<'EOF'
<!DOCTYPE html>
<html lang="sl">
<head><meta charset="UTF-8"><title>Moj Docker</title></head>
<body><h1>Moj prvi Docker strežnik!</h1></body>
</html>
EOF
```

Odpri `http://IP_STREZNIKA:8080`.

## 8. Izdelava lastnega imagea

Ustvari mapo projekta:

```bash
mkdir -p ~/docker/moj-projekt
cd ~/docker/moj-projekt
```

Ustvari datoteko `index.html`:

```bash
cat > index.html <<'EOF'
<!DOCTYPE html>
<html lang="sl">
<head><meta charset="UTF-8"><title>Moj projekt</title></head>
<body><h1>Moj prvi Docker projekt!</h1></body>
</html>
EOF
```

Ustvari datoteko `Dockerfile`:

```bash
cat > Dockerfile <<'EOF'
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
EOF
```

Zgradi image:

```bash
sudo docker build -t moj-splet:1.0 .
```

Zaženi container:

```bash
sudo docker run -d \\
  --name moj-splet \\
  --restart unless-stopped \\
  -p 8081:80 \\
  moj-splet:1.0
```

Odpri:

```text
http://IP_STREZNIKA:8081
```

Pika na koncu ukaza `docker build` pomeni, da je trenutna mapa kontekst izdelave imagea.

## 9. Docker Compose

Compose omogoča, da nastavitve containerja shraniš v datoteko in jih lažje ponovno uporabiš.

Ustvari mapo:

```bash
mkdir -p ~/docker/nginx-compose
cd ~/docker/nginx-compose
```

Ustvari `compose.yaml`:

```bash
cat > compose.yaml <<'EOF'
services:
  nginx:
    image: nginx:alpine
    container_name: moj-nginx-compose
    restart: unless-stopped
    ports:
      - "8082:80"
EOF
```

Zaženi storitev:

```bash
sudo docker compose up -d
```

Prikaži stanje:

```bash
sudo docker compose ps
```

Prikaži dnevnike:

```bash
sudo docker compose logs -f
```

Ustavi in odstrani container ter omrežja, ki jih je ustvaril Compose:

```bash
sudo docker compose down
```

Ta ukaz ne odstrani imagea in običajno ne izbriše named volumes. Bind-mountane datoteke ostanejo na strežniku.

**Pozor:** `docker compose down -v` lahko izbriše named volumes in s tem pomembne podatke.

## 10. Čiščenje in vzdrževanje

Prikaži porabo prostora:

```bash
sudo docker system df
```

Odstrani neuporabljene, ustavljene containerje:

```bash
sudo docker container prune
```

Odstrani neuporabljene image:

```bash
sudo docker image prune
```

Odstrani vse neuporabljene image, ne samo visečih:

```bash
sudo docker image prune -a
```

Pred čiščenjem preveri, kaj uporabljaš. Ukazov za čiščenje ne izvajaj na slepo na strežniku s pomembnimi storitvami.

## 11. Najpogostejši ukazi

| Opravilo | Ukaz |
|---|---|
| Delujoči containerji | `sudo docker ps` |
| Vsi containerji | `sudo docker ps -a` |
| Lokalni imagei | `sudo docker images` |
| Prenos imagea | `sudo docker pull IMAGE` |
| Ustvari in zaženi container | `sudo docker run IMAGE` |
| Ustavi container | `sudo docker stop IME` |
| Zaženi container | `sudo docker start IME` |
| Ponovni zagon | `sudo docker restart IME` |
| Dnevniki | `sudo docker logs IME` |
| Vstop v container | `sudo docker exec -it IME sh` |
| Podrobnosti | `sudo docker inspect IME` |
| Poraba CPU/RAM | `sudo docker stats` |
| Poraba prostora | `sudo docker system df` |
| Izdelava imagea | `sudo docker build -t IME:VERZIJA .` |
| Zagon Compose | `sudo docker compose up -d` |
| Ustavitev Compose | `sudo docker compose down` |

Zamenjaj `IME`, `IMAGE` in `VERZIJA` z ustreznimi imeni.

## 12. Dovoljenja in varnost

Docker lahko uporabljaš z `sudo`. Če želiš ukaze izvajati brez `sudo`, lahko svojega uporabnika dodaš v skupino `docker`:

```bash
sudo usermod -aG docker "$USER"
```

Nato se odjavi iz SSH in ponovno prijavi.

**Varnostno opozorilo:** skupina `docker` omogoča zelo visoke privilegije, praktično primerljive s skrbniškimi. Dodaj samo zaupanja vredne uporabnike.

Ne objavljaj vrat containerjev neposredno v internet brez ustrezne zaščite. Preveri tudi pravila požarnega zidu, usmerjevalnika in dostop do upravljalnih vmesnikov.

## Uradna dokumentacija

- [Namestitev Dockerja na Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Docker run](https://docs.docker.com/engine/containers/run/)
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Upravljanje podatkov](https://docs.docker.com/engine/storage/)

---

*Vodič je namenjen učenju in uporabi Dockerja na Ubuntu Serverju. Pred brisanjem containerjev, volumeov ali imageov vedno preveri, ali ne vsebujejo pomembnih podatkov.*

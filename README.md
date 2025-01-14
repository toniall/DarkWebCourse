# DarkWeb Docker Container

## Getting Started with Digital Ocean

If you don’t have a Digital Ocean account yet, consider using this Referral Invite to get **$200** free credit. 

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%201.svg)](https://www.digitalocean.com/?refcode=66c72edc0110&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

https://m.do.co/c/66c72edc0110



Follow these steps to create a Droplet (virtual server) where you’ll run this container:

1. **Sign up** using the referral link to receive your free credits.  
2. **Create a new Droplet** in your Digital Ocean dashboard:
   - Choose an **Ubuntu** image (version 20.04 or later recommended).
   - Select your **Droplet size** (e.g., 1 CPU, 1GB RAM – or larger if needed).
   - Choose a **Data Center Region** close to you.
   - Add your **SSH keys** or use a **root password** (recommended to use SSH keys).
   - Click **Create Droplet** and wait for the Droplet to be ready.
3. **Obtain the Droplet’s IP address** from the Digital Ocean dashboard.

## Cloning This Repository

1. Once your Droplet is running, **SSH** into your server:

```bash
ssh root@<DROPLET_IP_ADDRESS>
```

3. Then clone the repository:

```
git clone https://github.com/toniall/DarkWebCourse.git
```

4. Go into the cloned directory:
```
cd DarkWebCourse
```

## Building and Running the Docker Container ##

1. Build the Docker image (using the included dockerfile):
```
docker build -t darkweb -f dockerfile .
```

2. Run the Docker container using one of the following commands:

 - Option A: docker run
```
docker run -d \
  --name darkweb \
  -e VNC_PW=password \
  -e VNC_USER=user \
  -p 5901:6080 \
  darkweb
```

- Option B: docker-compose
```
docker-compose up -d
```

3. Accessing the Container
Open your browser and go to:
```
https://<SERVER_IP>:5901
```
Note: The port 5901 can be changed to any other port if needed (e.g., -p 5902:6080, -p 5903:6080, etc.).

**Note: Sometimes, you must reload the page to get all the content correctly.**

4. When prompted for a username and password, use the same VNC_USER and VNC_PW you specified when running the container (e.g., user / password).

4. When prompted for a username and password, use the same VNC_USER and VNC_PW you specified when running the container (e.g., user / password).

## Multiple Containers ##
To run multiple containers on the same server, use different ports (e.g., -p 5902:6080, -p 5903:6080, etc.). Note that multiple users can connect, but only one user can control the session at a time.

## Additional Docker Commands ##
Check container logs:
```
docker logs darkweb
```

Stop the container:
```
docker stop darkweb
```

Remove the container:
```
docker rm darkweb
```

